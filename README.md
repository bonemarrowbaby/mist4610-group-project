# Wilderness Exploration Society — Database Project

**MIST 4610 | Group Project | Case #1**  
University of Georgia — Terry College of Business | Spring 2026

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Team Members](#team-members)
3. [Data Model](#data-model)
4. [Data Model Description](#data-model-description)
5. [Data Dictionary](#data-dictionary)
6. [Queries](#queries)
7. [Database Information](#database-information)

---

## Project Overview

The Wilderness Exploration Society (WES) at Peachtree State University is an outdoor recreation organization that manages equipment rentals and trip registrations for university members (students, faculty/staff, and alumni) as well as sponsored guests. This project designs and implements a relational database to support WES operations, including:

- Customer and member/guest management with sponsor tracking
- Trip type definitions, prerequisite chains, and scheduling
- Trip registration with waiver and status tracking
- Equipment inventory management by type and individual item
- Rental agreement processing with per-item return date tracking
- Equipment maintenance requests, maintenance logs, and parts usage
- Supplier and parts inventory management
- Staff assignment to trips and rental agreements
- Staff assignment to maintenance work – Extension
- Maintenance requests, logs, and parts tracking – Extension
- Suppliers and parts inventory details – Extension

---

## Team Members

| Name               |
|--------------------|
| Haylesh Fernandez  |
| Italia Roman       |
| Zain Naseer        |
| Alden Majors       |

---

## Data Model 

![WES ER Diagram](er_diagram.png)

---

## Data Model Description

### Customers, Members & Guests
All people who interact with WES are stored in the `Customer` table (ID, first/last name, phone, date of birth, and customer type). University members have an associated `Member` record capturing their university role. Guests are tracked in a separate `Guest` table with a reference to their sponsoring `Customer`. A member can sponsor many guests, but each guest has exactly one sponsor.

### Trip Types & Prerequisites
WES offers named trip types stored in `Trip_Type`, each with a difficulty level, fee, and length in days. A self-referencing many-to-many bridge table — `Trip_Prerequisite` — captures which trip types must be completed before others can be attempted.

### Scheduled Trips & Registrations
Each trip type may be offered on multiple dates via `Scheduled_Trip`, which also designates a lead and assistant staff member. A unique constraint prevents the same trip type from being scheduled twice on the same date. `Registration` records each customer's enrollment, including waiver status (TINYINT: 1 = signed, 0 = not signed) and registration status (Confirmed, Waitlisted, or Cancelled). A unique constraint ensures at most one registration per customer per scheduled trip.

### Equipment Types & Items
Equipment is tracked at two levels: `Equipment_Type` defines the category with three tiered daily rates (student, faculty/alumni, guest), while `Equipment_Item` tracks each individual physical piece of gear with a unique auto-incremented ID and condition rating.

### Rental Agreements & Rental Items
When a customer rents equipment, a `Rental_Agreement` is created with a rental date and managing staff member. The bridge table `Rental_Agreement_has_Equipment_Item` links each agreement to one or more individual equipment items, recording expected and actual return dates per item.

### Maintenance Requests, Logs & Parts - Extension
Equipment issues are filed as `Maintenance_Request` records linked to a specific item and assigned staff member, with priority and status tracking. Work performed is logged in `Maintenance_Log` (date, description, outcome, cost, performing staff, and item). Parts consumed are recorded in `Maintenance_Part`, bridging a log entry to a `Part` and quantity used.

### Suppliers & Parts - Extension 
`Supplier` stores vendor contact information. `Part` tracks individual parts with unit cost, stock quantity, and a link to the supplying vendor.

---

## Data Dictionary

![Customer Table](table_customer.png)
![Member Table](table_member.png)
![Guest Table](table_guest.png)
![Staff Table](table_staff.png)
![Trip Type Table](table_triptype.png)
![Trip Prerequisite Table](table_tripprerequisite.png)
![Scheduled Trip Table](table_scheduledtrip.png)
![Registration Table](table_registration.png)
![Equipment Type Table](table_equipmenttype.png)
![Equipment Item Table](table_equipmentitem.png)
![Rental Agreement Table](table_rentalagreement.png)
![Rental Agreement Item Table](table_rentalagreementitem.png)
![Maintenance Request Table](table_maintenancerequest.png)
![Maintenance Log Table](table_maintenancelog.png)
![Maintenance Part Table](table_maintenancepart.png)
![Part Table](table_part.png)
![Supplier Table](table_supplier.png)

---

## Queries
| Feature                     | Q1 | Q2 | Q3 | Q4 | Q5 | Q6 | Q7 | Q8 | Q9 | Q10 |
|----------------------------|----|----|----|----|----|----|----|----|----|-----|
| Multiple Table Join        |    |    |    | X  |    | X  |    |    | X  | X   |
| Subquery                   |    |    |    |    | X  | X  |    |    |    |     |
| GROUP BY                   |    |    |    |    |    |    | X  | X  |    | X   |
| GROUP BY with HAVING       |    |    |    |    |    |    |    | X  |    |     |
| Multi-condition WHERE      | X  |    |    | X  |    | X  |    |    |    |     |
| Built-in Functions         |    |    |    |    | X  |    | X  | X  | X  | X   |
| REGEXP                     |    | X  |    |    | X  |    |    |    |    |     |
| NOT EXISTS / NOT IN        |    |    |    |    |    | X  |    |    |    |     |
| ORDER BY                   | X  | X  | X  |    |    |    | X  |    | X  |     |


### Query 1 - 
Query 1 lists the trip type ID, trip name, and trip fee for all trips that cost more than $75 and than 3 days. The results are also ordered by trip fee in ascending order.

![Query1](query1.png)

Query 1 allows WES managers to identify which trip types are short in duration but generate higher fees, making them attractive premium offerings. These trips are likely appealing to busy students, faculty, or guests with limited time, so WES can prioritize promoting and scheduling these trips more frequently to maximize revenue.

---

### Query 2 
Query 2 lists the request ID, item ID, and status for all maintenance requests whose status contains the word “Progress.” The results are also ordered by request ID.

![Query2](query2.png)

Query 2 allows WES managers to see which equipment items are currently undergoing maintenance and are unavailable for rental. This helps ensure that enough equipment remains available for upcoming rentals and trips, while also allowing managers to monitor repair progress and avoid operational delays.

---

### Query 3 
 Query 3 lists the equipment type ID, equipment name, and guest rental rate for all equipment types with a guest rate greater than or equal to $5. The results are also ordered by guest rate in ascending order.

<img width="821" height="402" alt="Screenshot 2026-04-09 at 12 31 28 PM" src="https://github.com/user-attachments/assets/d7a594cd-710e-4300-b233-72d268e16551" />

Query 3 allows WES managers to identify which equipment types have higher rental rates for guests, who are typically charged the most. These items likely generate greater revenue, so WES can prioritize maintaining and stocking these high-value items to increase profitability.

---

### Query 4 
Query 4 lists the item ID, condition rating, and equipment name for all equipment items that are in poor condition. This is done by joining the equipment item and equipment type tables.

![Query4](query4.png)

Query 4 allows WES managers to quickly identify equipment items that are in poor condition and may need repair or replacement. Since unsafe or damaged equipment can negatively impact trip experiences and safety, this query helps managers prioritize maintenance and ensure all gear meets quality standards.

---

### Query 5 
Query 5 lists the percentage of customers who are students, faculty/staff, alumni, and guests out of the total number of customers. These percentages are calculated using subqueries.

![Query5](query5.png)

Query 5 allows WES managers to understand the distribution of customers across different groups such as students, faculty/staff, alumni, and guests. This helps WES tailor pricing, marketing strategies, and trip offerings toward their largest customer segments to better meet demand.

---

### Query 6 
Query 6 lists the item ID and equipment name for all equipment items that do not appear in any maintenance request. This is done using a subquery to exclude items with maintenance records.

![Query6](query6.png)

Query 6 allows WES managers to identify equipment that has never required maintenance, which may indicate either high reliability or a lack of inspection. This helps managers ensure that all equipment is being properly monitored and maintained to prevent unexpected failures during rentals or trips.

---

### Query 7 
Query 7 lists each customer ID along with their total number of rentals and assigns a rental level of High, Medium, or Low based on that total. The results are grouped by customer ID and ordered in ascending order.

![Query7](query7.png)

Query 7 allows WES managers to categorize customers based on how frequently they rent equipment, identifying high, medium, and low activity users. This helps managers target frequent renters with loyalty incentives while also encouraging less active customers to increase participation.

---

### Query 8 
Query 8 lists the staff ID of employees along with the number of maintenance tasks they have performed, but only includes those who have completed exactly three tasks. The results are grouped by staff ID.

![query8](query8.png)

Query 8 allows WES managers to identify staff members who have completed exactly three maintenance tasks, providing insight into workload distribution. This helps ensure maintenance responsibilities are balanced and allows managers to adjust staffing if certain employees are under- or over-utilized.

---

### Query 9 
Query 9 lists each scheduled trip ID along with the full names of the lead and assistant staff members assigned to the trip. The results are also ordered by scheduled trip ID.

![Query9](query9.png)

Query 9 allows WES managers to clearly see which staff members are assigned as lead and assistant leaders for each scheduled trip. This ensures that every trip is properly staffed and helps managers coordinate scheduling, accountability, and communication among staff.


---

### Query 10 
Query 10 lists each equipment type along with the average cost of maintenance and categorizes each as High Cost,  Medium Cost, or Low Cost. The results are grouped by equipment name.

![Query10](query10.png)

Query 10 allows WES managers to evaluate which types of equipment have higher average maintenance costs and categorize them accordingly. This helps managers make informed decisions about whether to continue maintaining certain equipment, adjust rental pricing, or invest in replacements to reduce long-term costs.


---

## Database Information

**Database Name:** `mb_B2`

All 10 queries are bookmarked as stored procedures following the `GP_Qx` format.

| Query | Procedure Name | Call Format |
|-------|---------------|-------------|
| Query 1  | `GP_Q1`  | `CALL GP_Q1();`  |
| Query 2  | `GP_Q2`  | `CALL GP_Q2();`  |
| Query 3  | `GP_Q3`  | `CALL GP_Q3();`  |
| Query 4  | `GP_Q4`  | `CALL GP_Q4();`  |
| Query 5  | `GP_Q5`  | `CALL GP_Q5();`  |
| Query 6  | `GP_Q6`  | `CALL GP_Q6();`  |
| Query 7  | `GP_Q7`  | `CALL GP_Q7();`  |
| Query 8  | `GP_Q8`  | `CALL GP_Q8();`  |
| Query 9  | `GP_Q9`  | `CALL GP_Q9();`  |
| Query 10 | `GP_Q10` | `CALL GP_Q10();` |
