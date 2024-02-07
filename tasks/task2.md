### Overview:
The mission is to enhance some functionality to the updated database to support some of the business rules around their integrated timesheets, pays, and also a new aspect of paid training programs for staff. This task entails the implementation of additional functionality via stored procedures.

**1 - Function- Annual Leave Calculation - Handling Numbers and Rates (e.g., Full-Time and Part-Time Rates)**

This function is designed to calculate the amount of annual leave an individual should receive based on the hours they have worked. A person working a full year is entitled to 4 weeks of annual leave. It's important to note that this function operates only with positive numbers. If a negative number of hours worked is input, the output will be 0 annual leave hours.

````sql
DELIMITER //
DROP FUNCTION IF EXISTS calculate_annual_leave;
//

CREATE FUNCTION calculate_annual_leave( numberOfHoursWorked DOUBLE)
	RETURNS DOUBLE -- RETURNS THE NUMBER OF ANNUAL LEAVE HOURS
    DETERMINISTIC
BEGIN
    DECLARE numberOfAnnualLeaveHours DECIMAL( 15,2);
	IF numberOfHoursWorked > 0 THEN 
		SET numberOfAnnualLeaveHours := numberOfHoursWorked/ 13.035714;
	ELSEIF numberOfHoursWorked <= 0 THEN 
		SET numberOfAnnualLeaveHours= 0;
	END IF;
    RETURN numberOfAnnualLeaveHours;
END
DELIMITER ;
SELECT calculate_annual_leave(38.5);

````
**Result of test** 
| Function Name           | Input        | Output |
|-------------------------|--------------|--------|
| calculate_annual_leave  | 38.5         | 2.95   |

This function is named calculate_annual_leave and takes numberOfHoursWorked as input. Inside the function, It declare the variable numberOfAnnualLeaveHours as a DECIMAL(15,2), ensuring precision to two decimal places. Then use an IF statement to check if numberOfHoursWorked is greater than zero. If it is, It calculate numberOfAnnualLeaveHours by dividing numberOfHoursWorked by 13.035714, which represents the number of hours in a standard work week divided by the number of weeks in a year (52 weeks).If numberOfHoursWorked is less than or equal to zero, It set numberOfAnnualLeaveHours to 0.

**2 - Function - tax calculator for a week**
````sql
DELIMITER //
DROP FUNCTION IF EXISTS calculate_basic_tax_withheld_amount;
//

CREATE FUNCTION calculate_basic_tax_withheld_amount(
		total_pay_before_tax DOUBLE,
        number_of_weeks INT,
        is_claiming_tax_free_threshhold BOOLEAN
	)
    RETURNS DOUBLE -- tax_withheld_amount
    DETERMINISTIC
BEGIN
 DECLARE total_pay_before_tax_per_week DECIMAL(15,2);
 DECLARE tax_withheld_amount DECIMAL(15,2);
	
	-- Calculate total pay before tax per week
    SET total_pay_before_tax_per_week := total_pay_before_tax / number_of_weeks;
    
    -- Calculate tax withheld amount based on income brackets
    IF total_pay_before_tax_per_week BETWEEN 0 AND 300 THEN 
	    SET tax_withheld_amount := 0;
    ELSEIF total_pay_before_tax_per_week BETWEEN 300 AND 800 THEN 
	    SET tax_withheld_amount := ((total_pay_before_tax_per_week - 300) * 0.15);
    ELSEIF total_pay_before_tax_per_week BETWEEN 800 AND 2300 THEN 
	    SET tax_withheld_amount := 152 + ((total_pay_before_tax_per_week - 800) * 0.3);
    ELSEIF total_pay_before_tax_per_week BETWEEN 2300 AND 3500 THEN 
	    SET tax_withheld_amount := 602 + ((total_pay_before_tax_per_week - 2300) * 0.375);
    ELSEIF total_pay_before_tax_per_week > 3500 THEN 
	    SET tax_withheld_amount := (total_pay_before_tax_per_week - 1142) * 0.45;
    END IF;
    
    -- Adjust tax withheld amount based on tax-free threshold claim
    IF is_claiming_tax_free_threshhold THEN 
        RETURN tax_withheld_amount * number_of_weeks;
    ELSE 
        RETURN tax_withheld_amount * 1.2 * number_of_weeks;
    END IF;
END
//
DELIMITER ;

SELECT calculate_basic_tax_withheld_amount(1000, 2, TRUE);
````

| Function Name                   | Input               | Output |
|---------------------------------|---------------------|--------|
| calculate_basic_tax_withheld_amount | (1000, 2, TRUE)     | 60     |

In this example, calculate_basic_tax_withheld_amount(1000, 2, TRUE) with a total pay of $1000 over 2 weeks and claiming the tax-free threshold as true, the result would be $60. The total pay before tax per week is $1000 / 2 = $500, which falls within the $300 to $800 income bracket, resulting in a tax withheld amount of $(500 - 300) * 0.15 = $30.
Since the claim for the tax-free threshold is true, the final result is $30 * 2 = $60.

**3 - Function - Sick leave**

````sql
DELIMITER //
DROP FUNCTION IF EXISTS calculate_sick_leave;
//

CREATE FUNCTION calculate_sick_leave(numberOfHoursWorked DOUBLE)
    RETURNS DOUBLE -- sick_leave_hours
    DETERMINISTIC
BEGIN
    DECLARE annual_hours_leave DECIMAL(15,2);
    DECLARE sick_leave_hours DECIMAL(15,2);
    
    -- Calculate annual leave hours using the calculate_annual_leave function
    SELECT calculate_annual_leave(numberOfHoursWorked) INTO annual_hours_leave;
    
    -- Calculate sick leave hours
    SET sick_leave_hours := 0.775 * annual_hours_leave;
    
    RETURN sick_leave_hours;
END
//

DELIMITER ;

SELECT calculate_sick_leave(346.5);

````
| Function Name        | Input    | Output |
|----------------------|----------|--------|
| calculate_sick_leave | 346.5    | 20.07  |


Explanation:
The calculate_sick_leave function is designed to calculate sick leave hours based on the number of hours worked.It first calculates the annual leave hours using the calculate_annual_leave function. Then, it multiplies the annual leave hours by 0.775 to determine the sick leave hours. For the input of 346.5 hours worked, the output is approximately 20.07 hours of sick leave.

**4 - Stored Procedure - staff who have worked across the most stores for a time period (timesheet + contracts)**

````sql
DELIMITER //

DROP PROCEDURE IF EXISTS staff_report_5;
//

CREATE PROCEDURE staff_report_5(
		IN staff_id INT,
		IN date_from_inclusive DATE,
        IN date_to_inclusive DATE
	)
BEGIN

    -- Declare variables
    DECLARE v_staff_id INT(11);
    DECLARE v_max INT(11) DEFAULT 0;
    
    -- Select the person ID and count the number of stores worked across for the given time period
    SELECT acq_person_id, COUNT(acq_store_id) AS number_of_stores INTO v_staff_id, v_max 
    FROM (
        -- Subset combining contract and store information
        SELECT acq_contract_id, acq_person_id, CONCAT(given_names, ' ', family_name) AS staff_name, acq_store_id, start_date, end_date 
        FROM acq_staff
        JOIN acq_contract ON acq_person_id = acq_staff_acq_person_id 
        JOIN casual_for_stores ON acq_contract_id = casual_contract_id
        
        UNION ALL 
        
        -- Another subset combining contract and store information
        SELECT acq_contract_id, acq_person_id, CONCAT(given_names, ' ', family_name) AS staff_name, for_store_id_as_home_store AS acq_store_id, start_date, end_date 
        FROM acq_staff
        JOIN acq_contract ON acq_person_id = acq_staff_acq_person_id 
        JOIN (
            SELECT for_store_id_as_home_store, acq_contract_acq_contract_id
            FROM acq_ft_contract
            
            UNION ALL 
            
            SELECT for_store_id_as_home_store, acq_contract_acq_contract_id
            FROM acq_pt_contract
        ) AS ft_pt ON acq_contract.acq_contract_id = ft_pt.acq_contract_acq_contract_id
    ) AS ft_pt_casual
    
    -- Filter the subset based on the given time period and staff ID
    WHERE ft_pt_casual.start_date < date_from_inclusive 
    AND (ft_pt_casual.end_date < date_from_inclusive OR ft_pt_casual.end_date IS NULL) 
    AND ft_pt_casual.acq_person_id = staff_id
    
    -- Group the results by person ID
    GROUP BY ft_pt_casual.acq_person_id;
    
    -- Return the results
    SELECT v_staff_id, v_max;

END
//  

DELIMITER ;


````
This stored procedure retrieves information about staff members who have worked across the most stores during a specified time period. It combines data from various tables to gather contract and store information. The procedure then filters the results based on the provided time period and staff ID, finally returning the person ID along with the count of stores worked in as v_staff_id and v_max, respectively.

**Testing**

````sql
call staff_report_5(15,"2022-12-12","2022-12-13");
```` 
**Result**

| v_staff_id | v_max |
|------------|-------|
|     15     |   7   |

**5 - Stored Procedure- reading all casual timesheet entries for a selected staff member for a period of time. - cursors and query results based on the query results**

````sql
DELIMITER //

DROP PROCEDURE IF EXISTS total_casual_pay_for_staff_member;
//

CREATE PROCEDURE total_casual_pay_for_staff_member(
		IN date_from_inclusive DATE,
        IN date_to_inclusive DATE,
        IN staff_id INT,
        OUT total_casual_pay DOUBLE
	)
BEGIN
    -- Declare variables
    DECLARE v_staff_id INT(11);
    DECLARE v_total_casual_pay_per_week DECIMAL(15,2);
    DECLARE v_total_casual_pay_per_period DECIMAL(15,2);
    DECLARE v_number_of_weeks INT(11);

    -- Select total casual pay per week for the specified staff member
    SELECT acq_staff_id, SUM(total_casual_pay_per_week) AS total_pay_multi_contract_per_week INTO v_staff_id, v_total_casual_pay_per_week
    FROM (
        -- Subquery to calculate total casual pay per week
        SELECT acq_staff_acq_person_id AS acq_staff_id, wage_per_week.*, start_date, end_date
        FROM acq_contract
        JOIN (
            -- Subquery to calculate total casual pay per week for each contract
            SELECT casual_contract_id, SUM(total_casual_pay_per_week) AS total_casual_pay_per_week
            FROM (
                SELECT casual_contract_id, number_of_hours * pay_rate AS total_casual_pay_per_week
                FROM acq_timesheet
            ) AS count_wage
            GROUP BY count_wage.casual_contract_id
        ) AS wage_per_week ON acq_contract.acq_contract_id = wage_per_week.casual_contract_id  
        WHERE start_date < date_from_inclusive 
        AND (end_date > date_to_inclusive OR end_date IS NULL)
    ) AS multi_contract 
    WHERE acq_staff_id = staff_id
    GROUP BY multi_contract.acq_staff_id;

    -- Calculate number of weeks between date_from_inclusive and date_to_inclusive
    SET v_number_of_weeks := WEEK(date_to_inclusive) - WEEK(date_from_inclusive);

    -- Calculate total casual pay for the specified period
    SET v_total_casual_pay_per_period := v_number_of_weeks * v_total_casual_pay_per_week;
    
    -- Store the result in the output parameter
    SELECT v_total_casual_pay_per_period INTO total_casual_pay;

    -- Return the results
    SELECT v_staff_id, v_total_casual_pay_per_period;

END
//  
DELIMITER ;


````
This stored procedure calculates the total pay amount that a staff member should receive (pre-tax) from all their time-sheet entries falling within a specified period. It retrieves the total casual pay per week for the specified staff member and then calculates the total casual pay for the specified period based on the number of weeks. The result is stored in the output parameter total_casual_pay.


**Testing**

````sql
call total_casual_pay_for_staff_member("2021-06-01","2021-12-20",15,@b);
select @b
````
**Result**

| v_staff_id | v_total_casual_pay_per_period |
|------------|-------------------------------|
|     15     |             85521.00          |
