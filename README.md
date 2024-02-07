# sql_acquisitionco
## Improvement of database function and stored procedures  for Acquisition Co
### Introduction 
#### Problem Statement
An investment company (Acquisition Co) has been buying a few fast-food brands to build up a portfolio of companies, staff, sales, locations, and stock. They are looking to build a system that can be used to add / remove / update details about the brands they have acquired.

Food Chain Company: A food chain company is a company that Acquisition Co owns and wants to be able to keep a track of. The details they care about are the date they bought the company, the amount they bought the company for, the company name, and a unique company identifier assigned by Acquisition Co. Each company may have many physical stores. Each store would have a unique id assigned by Acquisition Co but might also have another unique identifier assigned by the company itself. The companies that are acquired may have used the same identifier scheme or different schemes to other acquired companies.

Shopping Centre locations and other locations: Each store may exist at a location, but a location may or may not be part of a larger shopping centre. So, for instance shop 5 and shop 7 in the Macquarie Shopping Centre are two different locations, but both part of the same shopping centre. The location details should include the address of the location, the size of the location (in square metres), and whether it has seating or is a takeaway only location. Acquisition Co would use their own location identifiers. For shopping centres, the details should include the name of the shopping centre, the phone number of the security office and the phone number and name of the centre manager. A location must be associated with at least one store, but a store must be associated with only one location.

Staff and Contracts: Acquisition Co would like to keep a track of current staff across all stores and chains. These staff may be hired on various contracts with different stores, or a contract might cover a number of stores. A unique person ID will be assigned by Acquisition Co, but there may be other store-specific identifiers used as part of a contract for a staff member. A staff member might also be working with multiple chains, so this needs to be able to be captured in the system, but only 1 staff ID should exist per person. A contract might be a full time, part time, or casual contract. A full-time contract would have a start date, a salary, and the amount to be paid into their superannuation each year. The part time contract would include the start date, and the agreed number of hours per week, and the hourly pay rate and superannuation amount. The casual rate includes the start date, a base pay rate, and a loading % to be paid on top of the base pay rate.

A staff member may also be a manager for store; however this is to be captured separately from any contract details stored in the system. The manager details include start date, contact number, email, and the date of their next scheduled performance review.



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
Acquisition Co requires reports on the following information from the database:

List of non-shopping centre locations containing the location's full address, the area size, and the potential seating capacity (assuming that min. service area is 15 sqm and 1.5 sqm per seating), e.g. for 125 sqm area, the area available for seating is 110 sqm, therefore can hold 73 seats.
Upcoming manager's performance review overview containing the manager's full name and email address, also the number of staff they managed. The report should only capture managers with the next review date within the next 30 days.
Casual staff contact directory containing their full contact details (full name, phone number, complete address, and date of birth) sorted alphabetically based on the full name. 
List of shopping centre stores with sit-down service containing the company's name, store's name, shopping centre's name and full address, all the stores located within a shopping centre that offer sit-down service in a specific state (choose one). The list should be sorted alphabetically based on the company's name, store's state, suburb, and name.
List all staff of one of the companies (choose one) containing their full name, phone number and start date, who has never worked for Krusty Burger and is not a manager. 
Yearly total part-time and full-time staff wage overview containing the company's name and the total yearly wage of all staff. Part-time staff yearly wage can be calculated based on agreed minimum hours per week, the hourly rate plus the superannuation. There are 52 weeks in a year. Full-time staff yearly wage is the salary amount.
List of long-time staff with multiple contracts containing their full name, the number of contracts, and the earliest start date. Only show staff with more than one contract and have worked for any company in Acquisition Co longer than ten years.
Large acquisition overview containing the company's name, the number of stores, and the number of staff. A company is considered a large acquisition if it has more than 25 stores or more than 100 staff.

#### Acquisition Co reports requirements:

Full names should be displayed as "<FAMILY NAME>, <Other Name>", e.g. "KRABAPPEL, Edna".
Full addresses should be displayed as "<Street Address>, <Suburb>, <STATE> <Postcode>", e.g. "63 Beecroft Road, Epping, NSW 2121".
Dates should be display as "<date> <MonthName> <4-digit year>", e.g. "29 January 2001".
Financial data should be display as "$ <amount>", e.g. "$ 12.34"
Columns containing values of a particular measurement must have one of the following applied, but not both:
the heading should include the metric, e.g. "Area Size in sqm"; OR
the values should be included as part of the values, e.g. "125 sqm"
## Overview
This project consists 2 parts : 

- part 1 : Acquisition Co requires reports on the information from the database such as: list of non-shopping centre locations, upcoming manager's performance review overview, etc.
* [Acquisition Co Data Analysis Question & Answers](./tasks/task1.md)

- part 2 :Acquisition Co also want to also integrate timesheets and payments for all their companies into a new payment tracking system . The task is add some functionality to the updated database to support some of the business rules around their integrated timesheets, pays, and also a new aspect of paid training programs for staff.
* [Acquisition Co Functions and Stored Procedures](./tasks/task2.md)

## Entity Relationship Diagram
![alt text](./images/ERD.PNG)