  # Bill Splitter App – Brief

Overview

Our company frequently places group food orders. Currently, we manually compare the receipt against our internal order list, then split the total among coworkers. This process is slow, error-prone, and confusing when handling exceptions like birthdays or shared items.

The Bill Splitter App will simplify this by allowing users to upload a receipt, match items with coworkers, and automatically calrson’s share.


---

Objectives

Eliminate manual bill calculations.

Ensure fair and transparent splitting of group orders.

Support special scenarios such as shared birthday charges.

Improve efficiency and reduce errors.



---

Key Features

1. Receipt Upload

Upload photo/PDF of receipt.

OCR extracts items, prices, and totals.



2. Order Matching

Import or manually enter group order details.

Match receipt items with coworkers.

Flag discrepancies between order and receipt.



3. Bill Splitting

Standard split: Each person pays for their own items.

Shared items: Split equally among selected coworkers.

Birthday pooling: Multiple birthdays can be grouped and the charges split among the rest of the team.

Rounding rules (e.g., round to nearest cent/peso).



4. Payment Summary

Display each person’s total.

Option to export/share summary (PDF, email, chat link).

Integration with payment apps (optional future feature).





---

Use Cases

UC1: Regular Lunch Order

Employees place an order.

Receipt is uploaded.

Items are matched to employees.

Each person pays only for their own meal.


UC2: Shared Items

A pizza or dessert is ordered for the table.

Receipt shows a single line item.

The app allows splitting equally among chosen coworkers.


UC3: Birthday Celebration

The team orders extra meals/cakes for birthdays.

Birthday charges are flagged.

Non-birthday employees split these charges equally.

If multiple birthdays happen at once, their combined charges are pooled and split.


UC4: Error/Discrepancy Check

The receipt total does not match the sum of assigned items.

The app highlights the mismatch and suggests corrections.


UC5: Final Report

The app generates a summary showing:

Each employee’s name.

Items they ordered.

Total amount due.


Report can be shared via company chat/email.



---

Users

Employees: Upload receipt, match items, view their charges.

Organizer: Oversees matching and validates totals.

Team: Views shared birthday charges and payment summary.



---





---

UC1: Regular Lunch Order

sequenceDiagram
    participant Employee
    participant App
    participant OCR
    participant Database

    Employee->>App: Upload Receipt
    App->>OCR: Extract Items & Prices
    OCR-->>App: Parsed Receipt Data
    App->>Employee: Display Items
    Employee->>App: Assign Items to Users
    App->>Database: Save Assignments
    App->>Employee: Show Total per Person


---

UC2: Shared Items

sequenceDiagram
    participant Employee
    participant App
    participant Database

    Employee->>App: Select Shared Item (e.g. Pizza)
    App->>Employee: Ask "Who shares this?"
    Employee->>App: Choose Coworkers
    App->>Database: Save Split Rule
    App->>Employee: Update Totals for Shared Item


---

UC3: Birthday Celebration

sequenceDiagram
    participant Organizer
    participant App
    participant Database

    Organizer->>App: Mark Birthday Items on Receipt
    App->>Organizer: Ask "Who pays for these?"
    Organizer->>App: Select All Non-Birthday Employees
    App->>Database: Save Birthday Pool Split
    App->>Organizer: Show Updated Totals
    App->>All Employees: Notify Charges & Birthday Split


---

UC4: Error/Discrepancy Check

sequenceDiagram
    participant App
    participant Organizer
    participant Database

    App->>Database: Calculate Sum of Assigned Items
    Database-->>App: Return Calculated Total
    App->>App: Compare with Receipt Total
    App->>Organizer: Show Discrepancy Warning
    Organizer->>App: Adjust Assignment or Add Missing Item
    App->>Database: Save Corrected Data


---

UC5: Final Report

sequenceDiagram
    participant Organizer
    participant App
    participant Database
    participant Employees

    Organizer->>App: Generate Summary Report
    App->>Database: Retrieve Assignments & Totals
    Database-->>App: Return Data
    App->>Organizer: Display Report (PDF/Share Link)
    Organizer->>Employees: Share Report
    Employees->>App: View Their Charges


---


