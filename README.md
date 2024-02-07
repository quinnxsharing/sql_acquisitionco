# sql_acquisitionco
## Improvement of database function and stored procedures  for Acquisition Co
### Introduction 
#### Problem Statement
An investment company (Acquisition Co) has been buying a few fast-food brands to build up a portfolio of companies, staff, sales, locations, and stock. They are looking to build a system that can be used to add / remove / update details about the brands they have acquired.

Food Chain Company: A food chain company is a company that Acquisition Co owns and wants to be able to keep a track of. The details they care about are the date they bought the company, the amount they bought the company for, the company name, and a unique company identifier assigned by Acquisition Co. Each company may have many physical stores. Each store would have a unique id assigned by Acquisition Co but might also have another unique identifier assigned by the company itself. The companies that are acquired may have used the same identifier scheme or different schemes to other acquired companies.

Shopping Centre locations and other locations: Each store may exist at a location, but a location may or may not be part of a larger shopping centre. So, for instance shop 5 and shop 7 in the Macquarie Shopping Centre are two different locations, but both part of the same shopping centre. The location details should include the address of the location, the size of the location (in square metres), and whether it has seating or is a takeaway only location. Acquisition Co would use their own location identifiers. For shopping centres, the details should include the name of the shopping centre, the phone number of the security office and the phone number and name of the centre manager. A location must be associated with at least one store, but a store must be associated with only one location.

Staff and Contracts: Acquisition Co would like to keep a track of current staff across all stores and chains. These staff may be hired on various contracts with different stores, or a contract might cover a number of stores. A unique person ID will be assigned by Acquisition Co, but there may be other store-specific identifiers used as part of a contract for a staff member. A staff member might also be working with multiple chains, so this needs to be able to be captured in the system, but only 1 staff ID should exist per person. A contract might be a full time, part time, or casual contract. A full-time contract would have a start date, a salary, and the amount to be paid into their superannuation each year. The part time contract would include the start date, and the agreed number of hours per week, and the hourly pay rate and superannuation amount. The casual rate includes the start date, a base pay rate, and a loading % to be paid on top of the base pay rate.

A staff member may also be a manager for store; however this is to be captured separately from any contract details stored in the system. The manager details include start date, contact number, email, and the date of their next scheduled performance review.



## Overview
This project consists 2 parts : 

- part 1 : Acquisition Co requires reports on the information from the database such as: list of non-shopping centre locations, upcoming manager's performance review overview, etc.
* [Acquisition Co Data Analysis Question & Answers](./tasks/task1.md)

- part 2 :Acquisition Co also want to also integrate timesheets and payments for all their companies into a new payment tracking system . The task is add some functionality to the updated database to support some of the business rules around their integrated timesheets, pays, and also a new aspect of paid training programs for staff.
* [Acquisition Co Functions and Stored Procedures](./tasks/task2.md)

## Entity Relationship Diagram
![alt text](./images/ERD.PNG)