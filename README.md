# Wilderness Exploration Society — Database Project

**MIST 4610 | Group Project | Case #1**  
University of Georgia — Terry College of Business | Spring 2026

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Team Members](#team-members)
3. [Entity-Relationship (ER) Diagram](#entity-relationship-er-diagram)
4. [Data Model Description](#data-model-description)
5. [Data Dictionary](#data-dictionary)
6. [Sample Queries](#sample-queries)
7. [Database Setup Instructions](#database-setup-instructions)

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
- Staff assignment to trips, rental agreements, and maintenance work

---

## Team Members

| Name | 
| [Haylesh Fernandez]
| [Italia Roman]
| [Zain Naseer]
| [Alden Majors]
| [Darryl McNeil]

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

### Maintenance Requests, Logs & Parts
Equipment issues are filed as `Maintenance_Request` records linked to a specific item and assigned staff member, with priority and status tracking. Work performed is logged in `Maintenance_Log` (date, description, outcome, cost, performing staff, and item). Parts consumed are recorded in `Maintenance_Part`, bridging a log entry to a `Part` and quantity used.

### Suppliers & Parts
`Supplier` stores vendor contact information. `Part` tracks individual parts with unit cost, stock quantity, and a link to the supplying vendor.

---

## Data Dictionary

### Table: `Customer`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Customer_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for each customer |
| `Customer_F_Name` | VARCHAR(15) | NOT NULL | Customer's first name |
| `Customer_L_Name` | VARCHAR(15) | NOT NULL | Customer's last name |
| `Customer_Phone` | VARCHAR(12) | | Customer's phone number |
| `Customer_DOB` | DATE | | Customer's date of birth |
| `Customer_Type` | VARCHAR(45) | NOT NULL | Student, Faculty/Staff, Alumni, or Guest |

---

### Table: `Member`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Member_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for the member record |
| `University_Role` | VARCHAR(90) | | Role at the university (e.g., Professor, Graduate Student) |
| `Customer_ID` | INT | FK → Customer(Customer_ID) | Links this member to a customer |

---

### Table: `Guest`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Guest_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for the guest record |
| `Customer_ID` | INT | FK → Customer(Customer_ID) | Links this guest to a customer |
| `Sponsor_ID` | INT | FK → Customer(Customer_ID) | The university member sponsoring this guest |

---

### Table: `Staff`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Staff_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for each staff member |
| `Staff_F_Name` | VARCHAR(15) | NOT NULL | Staff member's first name |
| `Staff_L_Name` | VARCHAR(15) | NOT NULL | Staff member's last name |
| `Staff_Phone` | VARCHAR(12) | | Staff member's phone number |

---

### Table: `Trip_Type`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Trip_Type_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for the trip type |
| `Trip_Type_Name` | VARCHAR(45) | NOT NULL | Descriptive name (e.g., "Beginner Kayaking") |
| `Trip_Type_Diff_Lvl` | VARCHAR(15) | NOT NULL | Difficulty level |
| `Trip_Type_Fee` | DECIMAL(5,2) | NOT NULL | Participation fee in dollars |
| `Trip_Type_Length_days` | INT | NOT NULL | Duration of the trip in days |

---

### Table: `Trip_Prerequisite`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Trip_Type_ID` | INT | PK, FK → Trip_Type(Trip_Type_ID) | The trip that has a requirement |
| `Prerequisite_Trip_Type_ID` | INT | PK, FK → Trip_Type(Trip_Type_ID) | The trip that must be completed first |

> Composite primary key. Self-referencing many-to-many bridge on `Trip_Type`.

---

### Table: `Scheduled_Trip`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Scheduled_Trip_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for each scheduled occurrence |
| `Trip_Date` | DATE | NOT NULL | Date the trip is scheduled |
| `Lead_Staff_ID` | INT | FK → Staff(Staff_ID) | Staff member serving as lead |
| `Assistant_Staff_ID` | INT | FK → Staff(Staff_ID) | Staff member serving as assistant leader |
| `Trip_Type_ID` | INT | FK → Trip_Type(Trip_Type_ID) | The trip type being offered |

> Unique constraint on (`Trip_Type_ID`, `Trip_Date`) — a trip type cannot be scheduled more than once per date.

---

### Table: `Registration`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Registration_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for each registration |
| `Customer_ID` | INT | FK → Customer(Customer_ID) | The registering customer |
| `Scheduled_Trip_ID` | INT | FK → Scheduled_Trip(Scheduled_Trip_ID) | The scheduled trip being registered for |
| `Registration_Waiver` | TINYINT(1) | NOT NULL, DEFAULT 0 | 1 = waiver signed, 0 = not signed |
| `Registration_Status` | VARCHAR(10) | NOT NULL | Confirmed, Waitlisted, or Cancelled |

> Unique constraint on (`Customer_ID`, `Scheduled_Trip_ID`) — one registration per customer per trip.

---

### Table: `Equipment_Type`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Equipment_Type_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for the equipment type |
| `Equipment_Name` | VARCHAR(45) | NOT NULL | Descriptive name (e.g., "Osprey Aether 70L Backpack") |
| `Student_Rate` | DECIMAL(5,2) | NOT NULL | Daily rental rate for students |
| `Faculty_Alumni_Rate` | DECIMAL(5,2) | NOT NULL | Daily rental rate for faculty, staff, and alumni |
| `Guest_Rate` | DECIMAL(5,2) | NOT NULL | Daily rental rate for guests |

---

### Table: `Equipment_Item`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Item_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for each physical item |
| `Condition_Rating` | VARCHAR(4) | NOT NULL | Good, Fair, or Poor |
| `Equipment_Type_Equipment_Type_ID` | INT | FK → Equipment_Type(Equipment_Type_ID) | The type this item belongs to |

---

### Table: `Rental_Agreement`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Agreement_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for each rental agreement |
| `Rental_Date` | DATE | NOT NULL | Date the rental agreement was created |
| `Customer_ID` | INT | FK → Customer(Customer_ID) | Customer who is renting |
| `Staff_ID` | INT | FK → Staff(Staff_ID) | Staff member managing the agreement |

---

### Table: `Rental_Agreement_has_Equipment_Item`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Rental_Agreement_Agreement_ID` | INT | PK, FK → Rental_Agreement(Agreement_ID) | The parent rental agreement |
| `Equipment_Item_Item_ID` | INT | PK, FK → Equipment_Item(Item_ID) | The specific physical item rented |
| `Expected_return_date` | VARCHAR(45) | | When the item is expected to be returned |
| `Actual_return_date` | VARCHAR(45) | | When the item was actually returned (NULL if still out) |

> Composite primary key on (`Rental_Agreement_Agreement_ID`, `Equipment_Item_Item_ID`).

---

### Table: `Maintenance_Request`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Request_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for the maintenance request |
| `Request_Date` | DATE | NOT NULL | Date the request was filed |
| `Issue_Description` | VARCHAR(90) | | Description of the equipment problem |
| `Priority` | VARCHAR(15) | | High, Medium, or Low |
| `Status` | VARCHAR(20) | | Open, In Progress, or Closed |
| `Item_ID` | INT | FK → Equipment_Item(Item_ID) | The item requiring maintenance |
| `Staff_ID` | INT | FK → Staff(Staff_ID) | Staff member assigned to the request |

---

### Table: `Maintenance_Log`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Log_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for the log entry |
| `Maintenance_Date` | DATE | NOT NULL | Date maintenance was performed |
| `Work_Done_Desc` | VARCHAR(90) | | Description of work performed |
| `Outcome` | VARCHAR(15) | | Resolved, Ongoing, etc. |
| `Maintenance_Count` | INT | | Number of maintenance events for this item |
| `Cost_of_Maintenance` | DECIMAL(5,2) | | Total cost of the work |
| `Maintenance_Request_ID` | INT | FK → Maintenance_Request(Request_ID) | The originating request |
| `Performed_by_Staff_ID` | INT | FK → Staff(Staff_ID) | Staff member who performed the work |
| `Item_ID` | INT | FK → Equipment_Item(Item_ID) | The item that was serviced |

---

### Table: `Maintenance_Part`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Maintenance_Part_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for this part-usage record |
| `Quantity_Used` | INT | NOT NULL | Number of parts consumed |
| `Maintenance_Log_ID` | INT | FK → Maintenance_Log(Log_ID) | The log entry where the part was used |
| `Part_ID` | INT | FK → Part(Part_ID) | The part that was consumed |

---

### Table: `Part`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Part_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for each part |
| `Part_Name` | VARCHAR(45) | NOT NULL | Name of the part |
| `Part_Unit_Cost` | DECIMAL(5,2) | NOT NULL | Cost per unit |
| `Part_Stock_Quantity` | INT | NOT NULL | Current quantity in stock |
| `Supplier_ID` | INT | FK → Supplier(Supplier_ID) | The supplier who provides this part |

---

### Table: `Supplier`

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `Supplier_ID` | INT | PK, AUTO_INCREMENT | Unique identifier for the supplier |
| `Supplier_Name` | VARCHAR(50) | NOT NULL | Supplier's business name |
| `Supplier_Contact_F_name` | VARCHAR(45) | | Contact person's first name |
| `Supplier_Contact_L_name` | VARCHAR(45) | | Contact person's last name |
| `Supplier_Phone` | VARCHAR(45) | | Supplier's phone number |
| `Supplier_Email` | VARCHAR(30) | | Supplier's email address |
| `Supplier_Address` | VARCHAR(45) | | Supplier's mailing address |

---

## Sample Queries

### Query 1 — List all upcoming confirmed registrations with customer and trip details

```sql
SELECT
    c.Customer_F_Name, c.Customer_L_Name, c.Customer_Type,
    tt.Trip_Type_Name, st.Trip_Date,
    r.Registration_Status, r.Registration_Waiver
FROM Registration r
JOIN Customer c ON r.Customer_ID = c.Customer_ID
JOIN Scheduled_Trip st ON r.Scheduled_Trip_ID = st.Scheduled_Trip_ID
JOIN Trip_Type tt ON st.Trip_Type_ID = tt.Trip_Type_ID
WHERE st.Trip_Date >= CURDATE() AND r.Registration_Status = 'Confirmed'
ORDER BY st.Trip_Date, tt.Trip_Type_Name;
```
**Purpose:** Generates trip rosters for upcoming scheduled trips so staff can prepare.

---

### Query 2 — Find all guests and their sponsors

```sql
SELECT
    CONCAT(gc.Customer_F_Name, ' ', gc.Customer_L_Name) AS GuestName,
    gc.Customer_Phone AS GuestPhone,
    CONCAT(sc.Customer_F_Name, ' ', sc.Customer_L_Name) AS SponsorName,
    sc.Customer_Type AS SponsorType
FROM Guest g
JOIN Customer gc ON g.Customer_ID = gc.Customer_ID
JOIN Customer sc ON g.Sponsor_ID  = sc.Customer_ID
ORDER BY SponsorName, GuestName;
```
**Purpose:** Verifies every guest has a valid sponsoring university member.

---

### Query 3 — Count registrations per scheduled trip broken down by status

```sql
SELECT
    tt.Trip_Type_Name, st.Trip_Date,
    SUM(CASE WHEN r.Registration_Status = 'Confirmed'  THEN 1 ELSE 0 END) AS Confirmed,
    SUM(CASE WHEN r.Registration_Status = 'Waitlisted' THEN 1 ELSE 0 END) AS Waitlisted,
    SUM(CASE WHEN r.Registration_Status = 'Cancelled'  THEN 1 ELSE 0 END) AS Cancelled,
    COUNT(*) AS Total
FROM Scheduled_Trip st
JOIN Trip_Type tt ON st.Trip_Type_ID = tt.Trip_Type_ID
LEFT JOIN Registration r ON st.Scheduled_Trip_ID = r.Scheduled_Trip_ID
GROUP BY st.Scheduled_Trip_ID, tt.Trip_Type_Name, st.Trip_Date
ORDER BY st.Trip_Date;
```
**Purpose:** Gives management a snapshot of enrollment health across all trips.

---

### Query 4 — List equipment items currently checked out (not yet returned)

```sql
SELECT
    ei.Item_ID, et.Equipment_Name, ei.Condition_Rating,
    ra.Rental_Date, raei.Expected_return_date,
    CONCAT(c.Customer_F_Name, ' ', c.Customer_L_Name) AS RentedBy
FROM Rental_Agreement_has_Equipment_Item raei
JOIN Equipment_Item ei ON raei.Equipment_Item_Item_ID = ei.Item_ID
JOIN Equipment_Type et ON ei.Equipment_Type_Equipment_Type_ID = et.Equipment_Type_ID
JOIN Rental_Agreement ra ON raei.Rental_Agreement_Agreement_ID = ra.Agreement_ID
JOIN Customer c ON ra.Customer_ID = c.Customer_ID
WHERE raei.Actual_return_date IS NULL
ORDER BY raei.Expected_return_date;
```
**Purpose:** Tracks all gear in the field and highlights overdue items.

---

### Query 5 — Calculate estimated rental revenue by equipment type and customer category

```sql
SELECT
    et.Equipment_Name, c.Customer_Type,
    COUNT(*) AS TimesRented,
    SUM(
        DATEDIFF(raei.Expected_return_date, ra.Rental_Date) *
        CASE c.Customer_Type
            WHEN 'Student' THEN et.Student_Rate
            WHEN 'Guest'   THEN et.Guest_Rate
            ELSE                et.Faculty_Alumni_Rate
        END
    ) AS EstimatedRevenue
FROM Rental_Agreement_has_Equipment_Item raei
JOIN Rental_Agreement ra ON raei.Rental_Agreement_Agreement_ID = ra.Agreement_ID
JOIN Customer c ON ra.Customer_ID = c.Customer_ID
JOIN Equipment_Item ei ON raei.Equipment_Item_Item_ID = ei.Item_ID
JOIN Equipment_Type et ON ei.Equipment_Type_Equipment_Type_ID = et.Equipment_Type_ID
GROUP BY et.Equipment_Name, c.Customer_Type
ORDER BY EstimatedRevenue DESC;
```
**Purpose:** Identifies highest-revenue gear and most profitable customer segments.

---

### Query 6 — Show all trip prerequisite chains

```sql
SELECT
    tt.Trip_Type_Name AS TripType, tt.Trip_Type_Diff_Lvl AS Difficulty,
    pre.Trip_Type_Name AS PrerequisiteName, pre.Trip_Type_Diff_Lvl AS PrerequisiteDifficulty
FROM Trip_Prerequisite tp
JOIN Trip_Type tt  ON tp.Trip_Type_ID              = tt.Trip_Type_ID
JOIN Trip_Type pre ON tp.Prerequisite_Trip_Type_ID = pre.Trip_Type_ID
ORDER BY tt.Trip_Type_Name;
```
**Purpose:** Lets staff communicate prerequisite requirements to participants.

---

### Query 7 — Find confirmed participants who have not signed their waiver

```sql
SELECT
    CONCAT(c.Customer_F_Name, ' ', c.Customer_L_Name) AS CustomerName,
    c.Customer_Phone, tt.Trip_Type_Name, st.Trip_Date, r.Registration_Status
FROM Registration r
JOIN Customer c ON r.Customer_ID = c.Customer_ID
JOIN Scheduled_Trip st ON r.Scheduled_Trip_ID = st.Scheduled_Trip_ID
JOIN Trip_Type tt ON st.Trip_Type_ID = tt.Trip_Type_ID
WHERE r.Registration_Waiver = 0
  AND r.Registration_Status = 'Confirmed'
  AND st.Trip_Date >= CURDATE()
ORDER BY st.Trip_Date, c.Customer_L_Name;
```
**Purpose:** Produces a contact list of confirmed participants still needing waivers.

---

### Query 8 — Show each staff member's trip leadership activity

```sql
SELECT
    CONCAT(s.Staff_F_Name, ' ', s.Staff_L_Name) AS StaffName,
    SUM(CASE WHEN st.Lead_Staff_ID      = s.Staff_ID THEN 1 ELSE 0 END) AS TripsLed,
    SUM(CASE WHEN st.Assistant_Staff_ID = s.Staff_ID THEN 1 ELSE 0 END) AS TripsAssisted
FROM Staff s
JOIN Scheduled_Trip st ON s.Staff_ID = st.Lead_Staff_ID OR s.Staff_ID = st.Assistant_Staff_ID
GROUP BY s.Staff_ID, StaffName
ORDER BY TripsLed DESC;
```
**Purpose:** Helps management evaluate staff workload and leadership experience.

---

### Query 9 — List open maintenance requests prioritized by urgency

```sql
SELECT
    mr.Request_ID, mr.Request_Date, mr.Priority, mr.Status,
    et.Equipment_Name, ei.Condition_Rating, mr.Issue_Description,
    CONCAT(s.Staff_F_Name, ' ', s.Staff_L_Name) AS AssignedStaff
FROM Maintenance_Request mr
JOIN Equipment_Item ei ON mr.Item_ID = ei.Item_ID
JOIN Equipment_Type et ON ei.Equipment_Type_Equipment_Type_ID = et.Equipment_Type_ID
JOIN Staff s ON mr.Staff_ID = s.Staff_ID
WHERE mr.Status != 'Closed'
ORDER BY CASE mr.Priority WHEN 'High' THEN 1 WHEN 'Medium' THEN 2 ELSE 3 END, mr.Request_Date;
```
**Purpose:** Gives the maintenance team a prioritized work queue.

---

### Query 10 — Find parts running low in stock with supplier contact details

```sql
SELECT
    p.Part_Name, p.Part_Stock_Quantity, p.Part_Unit_Cost,
    sup.Supplier_Name,
    CONCAT(sup.Supplier_Contact_F_name, ' ', sup.Supplier_Contact_L_name) AS SupplierContact,
    sup.Supplier_Phone, sup.Supplier_Email
FROM Part p
JOIN Supplier sup ON p.Supplier_ID = sup.Supplier_ID
WHERE p.Part_Stock_Quantity < 10
ORDER BY p.Part_Stock_Quantity ASC;
```
**Purpose:** Flags low-stock parts so staff can reorder before running out.

