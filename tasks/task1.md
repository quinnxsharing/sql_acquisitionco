
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
loc_street_address|the area size|the potential seating capacity|
----------------------------------------------------------------|
6 Springfield Blvd	|138.00 sqm	|82|
15 Springfield Blvd	|116.00 sqm	|67|
23 3rd St	|203.00 sqm	|125|





Upcoming manager's performance review overview containing the manager's full name and email address, also the number of staff they managed. The report should only capture managers with the next review date within the next 30 days.
Casual staff contact directory containing their full contact details (full name, phone number, complete address, and date of birth) sorted alphabetically based on the full name. 
List of shopping centre stores with sit-down service containing the company's name, store's name, shopping centre's name and full address, all the stores located within a shopping centre that offer sit-down service in a specific state (choose one). The list should be sorted alphabetically based on the company's name, store's state, suburb, and name.
List all staff of one of the companies (choose one) containing their full name, phone number and start date, who has never worked for Krusty Burger and is not a manager. 
Yearly total part-time and full-time staff wage overview containing the company's name and the total yearly wage of all staff. Part-time staff yearly wage can be calculated based on agreed minimum hours per week, the hourly rate plus the superannuation. There are 52 weeks in a year. Full-time staff yearly wage is the salary amount.
List of long-time staff with multiple contracts containing their full name, the number of contracts, and the earliest start date. Only show staff with more than one contract and have worked for any company in Acquisition Co longer than ten years.
Large acquisition overview containing the company's name, the number of stores, and the number of staff. A company is considered a large acquisition if it has more than 25 stores or more than 100 staff.
