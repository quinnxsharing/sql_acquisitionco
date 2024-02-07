
### Addtional Database specifications : 
Timesheet ID is a combination of the year of timesheet submission and an identifier.
Store ID is a combination of the state of the store's location and an identifier.
Role Code is made up of a single letter code of the role and a two digits level.
Role Name consists of the role description (e.g. Manager) and the level (e.g. L2).
Database specifications:

A person's family name should be separated from any other parts of the name (given name, middle name, etc.)
All names should be stored in normal language conventions, such as "Krabappel" and "Anne Marie".
Addresses should be separated into street address, suburb, state, and postcode.
All financial data (price, cost, pay rate, etc.) should be stored to two decimal places.

### Acquisition Co reports requirements:

Full names should be displayed as "<FAMILY NAME>, <Other Name>", e.g. "KRABAPPEL, Edna".
Full addresses should be displayed as "<Street Address>, <Suburb>, <STATE> <Postcode>", e.g. "63 Beecroft Road, Epping, NSW 2121".
Dates should be display as "<date> <MonthName> <4-digit year>", e.g. "29 January 2001".
Financial data should be display as "$ <amount>", e.g. "$ 12.34"
Columns containing values of a particular measurement must have one of the following applied, but not both:
the heading should include the metric, e.g. "Area Size in sqm"; OR
the values should be included as part of the values, e.g. "125 sqm"

### Acquisition Co requires reports on the following information from the database:

**1.** List of non-shopping centre locations containing the location's full address, the area size, and the potential seating capacity ?(assuming that min. service area is 15 sqm and 1.5 sqm per seating), e.g. for 125 sqm area, the area available for seating is 110 sqm, therefore can hold 73 seats.
````sql
SELECT loc_street_address, CONCAT(loc_size,' sqm') as'the area size' , round(( loc_size-15)/ 1.5,0) as 'the potential seating capacity'
FROM location
WHERE centre_id IS NULL;
````
**Results:**

| loc_street_address | area size (sqm) | potential seating capacity |
|--------------------|-----------------|----------------------------|
| 6 Springfield Blvd | 138.00          | 82                         |
| 15 Springfield Blvd| 116.00          | 67                         |
| 23 3rd St          | 203.00          | 125                        |


**2.** Casual staff contact directory containing their full contact details (full name, phone number, complete address, and date of birth) sorted alphabetically based on the full name. 
````sql
SELECT CONCAT(UPPER(staff_family_name) ,', ', staff_other_names) as staff_fullname,staff_phone, CONCAT(staff_street_address,', ', staff_suburb,', ', staff_state,', ', staff_postcode)
as ' address', CONCAT(DAY(dob),' ',MONTHNAME(dob),' ',YEAR(dob)) as'day of birth'
FROM staff s  RIGHT JOIN contract c ON s.acq_staff_id = c.staff_id JOIN  pt_contract pt ON c.contract_id=pt.contract_id
ORDER BY staff_fullname;
````
**Results:**
| Staff Fullname       | Staff Phone | Address                               | Date of Birth |
|----------------------|-------------|---------------------------------------|---------------|
| BELLOWS, Rory B      | 0412836227  | 62 Kearney St, Springfield, TAS, 7123 | 21 July 1986  |
| FREEDMAN, Jeremy     | 0477122226  | 11 Quimby St, Springfield, TAS, 7112  | 12 March 1982 |
| KURTOFSKY, Herschel S| 0411888222  | 33 Kearney St, Springfield, TAS, 7123 | 15 March 1981 |

**3.**List of shopping centre stores with sit-down service containing the company's name, store's name, shopping centre's name and full address, all the stores located within a shopping centre that offer sit-down service in a specific state (choose one). The list should be sorted alphabetically based on the company's name, store's state, suburb, and name.

````sql
SELECT co.company_name, s.store_name, ce.centre_name, CONCAT(l.loc_street_address,', ', l.loc_suburb,', ', l.loc_state,', ', l.loc_postcode) as 'shopping centre addtress'
FROM centre ce RIGHT JOIN location l ON ce.centre_id = l.centre_id RIGHT JOIN store s ON l.loc_id = s.store_loc_id LEFT JOIN company co ON s.company_id= co.company_id
WHERE l.hasSeating=1 AND l.loc_state ='NSW'
ORDER BY co.company_name, l.loc_state, l.loc_suburb, s.store_name;
```` 
**Results:**
| Company Name | Store Name      | Centre Name             | Shopping Centre Address                   |
|--------------|-----------------|-------------------------|-------------------------------------------|
| KCF          | Macquarie Park  | Chatstone Shopping Centre | Shop DF2, Macquarie Park, NSW, 2341     |
| Soul Origin  | Parramatta      | Westfield Parramatta    | Shop A03, Parramatta, NSW, 2150          |
| Woolworth    | Parramatta      | Westfield Homebush      | Shop H07, Homebush, NSW, 2233            |
| Woolworth    | Macquarie Park  | Macquarie Shopping Centre | Shop 132, Macquarie Park, NSW, 2113    |



**4.**Yearly total part-time and full-time staff wage overview containing the company's name and the total yearly wage of all staff. Part-time staff yearly wage can be calculated based on agreed minimum hours per week, the hourly rate plus the superannuation. There are 52 weeks in a year. Full-time staff yearly wage is the salary amount.

````sql
SELECT co.company_name,SUM((SELECT pt_hours_per_wk*pt_hourly_rate*52 + pt_super_amt as 'total_pt_wage'
FROM pt_contract pt WHERE ct.contract_id=pt.contract_id))as 'total part-time wage', SUM((SELECT ft_salary + ft_super_amt
FROM ft_contract ft WHERE ct.contract_id=ft.contract_id)) as 'total_ft_wage'
FROM company co, contract ct 
WHERE co.company_id=ct.company_id
GROUP BY company_name;
````
**Results:**

| Company Name | Total Part-Time Wage | Total Full-Time Wage |
|--------------|----------------------|----------------------|
| BigW         | 80471.29             | 55250.00             |
| Soul Origin  |                      | 56375.00             |
| Woolworth    |                      | 53500.00             |

**5.**Large acquisition overview containing the company's name, the number of stores, and the number of staff. A company is considered a large acquisition if it has more than 2 stores or more than 3 staff.
````sql
SELECT company_name ,c.company_id, num_staff, num_store
FROM company c , (SELECT cp.company_id, COUNT( distinct ct.staff_id) as num_staff
FROM company cp RIGHT JOIN contract ct ON cp.company_id= ct.company_id RIGHT JOIN staff s ON ct.staff_id = s.acq_staff_id
GROUP BY cp.company_id
HAVING num_staff >2) nsf ,(SELECT cp.company_id, COUNT( acq_store_id) as num_store
FROM company cp RIGHT JOIN store s ON cp.company_id= s.company_id
GROUP BY cp.company_id
HAVING  num_store> 1) nst
WHERE c.company_id = nsf.company_id AND c.company_id = nst.company_id;
````
**Results:**

| Company Name | Company ID | Number of Staff | Number of Stores |
|--------------|------------|-----------------|------------------|
| BigW         | 1          | 3               | 2                |

