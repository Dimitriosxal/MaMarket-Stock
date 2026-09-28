# MaMarket Stock — FR/NFR Catalogue

**Version:** 1.1 (Parent Version: 1.0)
**Changelog:**
* R1: Added Product Deletion (FR-010).
* R2: Replaced CSV/Excel sales upload with Chat-Based Sales Entry (FR-008).
* R3: Added Unrecognised Product Fuzzy Match (FR-009).
* R4: Removed all file upload features for sales data (Removed FR-003, FR-004, FR-007, NFR-002).

## 1. Project Context
*   **Project Name:** MaMarket Stock
*   **Domain:** Retail Market Inventory Management
*   **Scope:** Greenfield, standalone, single-machine web-based application (React + Node.js + SQLite).
*   **Constraints:** No external network sharing, single user at a time, no customer-facing interface.
*   **Motivation:** Replace error-prone manual/spreadsheet tracking with a structured, auditable batch ledger tailored to the end-of-shift operational rhythm of retail markets.

## 2. Stakeholder & User Map

| Role | Responsibilities | Key Tasks | Pain Points | Success Criteria | Tech Proficiency |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Administrator** | System oversight, catalogue management, configuration | Add products, edit reorder thresholds, delete products, record sales via chat | Lacks structured catalogue tools; reactive stock management | Zero discrepancies in catalogue; proactive low-stock alerts | Moderate |
| **Staff** | Daily operations, inventory tracking | Record sold items via chat, view inventory, act on alerts | Manual decrementing is slow/error-prone; no clear low-stock signal | Fast end-of-shift entry; immediate visual alerts | Basic |

## 3. Business Analysis

**Problem/Opportunity:** Retail market staff struggle to track stock across diverse categories without a dedicated tool. Spreadsheets lack real-time visibility and automated reorder signals. The opportunity is to build a lean, auditable ledger that validates the core inventory loop (chat entry → decrement → view → reorder).

**Business Objectives:**
*   **B-001:** Streamline product catalogue entry to lower onboarding friction.
*   **B-002:** Automate stock decrementing based on end-of-shift sales data.
*   **B-003:** Provide immediate visibility into inventory levels and low-stock conditions.

## 4. Functional Requirements (FR)

### FR-001: Chat-Style Product Creation
*   **Description:** The system shall allow users to create a new product via a conversational chat interface.
    *   *Actor:* Administrator, Staff
    *   *Trigger:* User types product details (Name, Price, Barcode) in the chat input.
    *   *Precondition:* The barcode does not currently exist in the database.
    *   *Input:* Name (string), Price (numeric), Barcode (string/scanner).
    *   *Processing Rule:* System validates input, prompts for initial quantity. Upon quantity input, creates the record with a default threshold of 0.
    *   *State Change:* New product record created in SQLite.
    *   *Output:* Success confirmation message in chat.
    *   *Error Behaviour:* If barcode exists, block entry and display "Barcode already exists. Use top-up flow."
    *   *Acceptance Boundary:* Product is immediately visible in the Inventory View with the specified initial quantity.
*   **Business Objective:** B-001
*   **Priority:** MUST HAVE
*   **Source:** Concept Document §4, OQ-01, OQ-02
*   **Dependencies:** None
*   **Acceptance Criteria:** AC-001, AC-002

### FR-002: Manual Stock Top-Up
*   **Description:** The system shall allow users to add quantity to an existing product via the chat interface.
    *   *Actor:* Administrator, Staff
    *   *Trigger:* User scans/enters an existing barcode in the chat input.
    *   *Precondition:* The barcode exists in the database.
    *   *Input:* Barcode (string), Quantity to add (integer).
    *   *Processing Rule:* System identifies the product, prompts for quantity, and increments the current stock.
    *   *State Change:* Product quantity updated in SQLite.
    *   *Output:* Success message with the new total quantity.
    *   *Error Behaviour:* Invalid quantity format shows an error prompt.
    *   *Acceptance Boundary:* Inventory View reflects the incremented quantity immediately.
*   **Business Objective:** B-001
*   **Priority:** MUST HAVE
*   **Source:** OQ-01
*   **Dependencies:** FR-001
*   **Acceptance Criteria:** AC-003, AC-004

### FR-008: Chat-Based Sales Entry
*   **Description:** The system shall allow users to record sold items via a conversational chat interface.
    *   *Actor:* Staff, Administrator
    *   *Trigger:* User enters sales data in the chat input.
    *   *Precondition:* Product exists in the database.
    *   *Input:* Chat message containing barcode/name and optional quantity (e.g., '1234567890', 'Cola', 'Cola 5', '1234567890 5', or comma-separated batch 'Cola 5, Fanta 3').
    *   *Processing Rule:* System parses the input. If quantity is omitted, it prompts "How many units sold?". If quantity is provided, it immediately decrements the stock. For batch inputs, it processes all items in one go.
    *   *State Change:* Product quantities decremented in DB.
    *   *Output:* Success confirmation message in chat.
    *   *Error Behaviour:* Invalid quantity format shows an error prompt.
    *   *Acceptance Boundary:* Stock is decremented accurately for all supported input formats.
*   **Business Objective:** B-002
*   **Priority:** MUST HAVE
*   **Source:** R2
*   **Dependencies:** None
*   **Acceptance Criteria:** AC-013, AC-014, AC-015

### FR-009: Unrecognised Product Fuzzy Match
*   **Description:** The system shall handle unrecognised product names or barcodes in the chat by offering a fuzzy match selection.
    *   *Actor:* Staff, Administrator
    *   *Trigger:* User enters a name or barcode that does not exactly match any existing product.
    *   *Precondition:* Chat-based sales entry is initiated.
    *   *Input:* Unrecognised string.
    *   *Processing Rule:* System searches for similar products (fuzzy/partial match). Displays a list of matches. User selects the correct one. If a quantity was needed, the system then prompts for it.
    *   *State Change:* None until selection and quantity are confirmed.
    *   *Output:* List of similar products.
    *   *Error Behaviour:* If no similar products are found, display "Product not found" and abort.
    *   *Acceptance Boundary:* User can successfully select a product from the fuzzy match list and complete the sale.
*   **Business Objective:** B-002
*   **Priority:** MUST HAVE
*   **Source:** R3
*   **Dependencies:** FR-008
*   **Acceptance Criteria:** AC-016

### FR-005: Real-Time Inventory View & Low-Stock Alerts
*   **Description:** The system shall display a real-time tabular view of all inventory, highlighting products that fall at or below their reorder threshold.
    *   *Actor:* Administrator, Staff
    *   *Trigger:* User views the inventory page or a stock-altering action completes.
    *   *Precondition:* None.
    *   *Input:* None.
    *   *Processing Rule:* Retrieve all products. If current quantity <= reorder threshold, apply a visual alert badge/colour to the row.
    *   *State Change:* None.
    *   *Output:* Tabular display of inventory.
    *   *Error Behaviour:* DB read error displays "Unable to load inventory".
    *   *Acceptance Boundary:* Table refreshes automatically after upload/chat entry; low stock items are clearly highlighted.
*   **Business Objective:** B-003
*   **Priority:** MUST HAVE
*   **Source:** Concept Document §4
*   **Dependencies:** FR-001, FR-008
*   **Acceptance Criteria:** AC-009

### FR-006: Inline Threshold Editing
*   **Description:** The system shall allow Administrators to edit the reorder threshold for any product directly within the Inventory View.
    *   *Actor:* Administrator
    *   *Trigger:* Admin clicks the threshold cell for a product.
    *   *Precondition:* User is an Administrator.
    *   *Input:* New threshold value (integer).
    *   *Processing Rule:* Validate integer. Update the threshold for the specific product.
    *   *State Change:* Product threshold updated in DB.
    *   *Output:* Cell updates to new value; alert status recalculates immediately.
    *   *Error Behaviour:* Non-integer input is rejected with an inline error tooltip.
    *   *Acceptance Boundary:* Threshold changes immediately affect the low-stock alert logic for that row.
*   **Business Objective:** B-003
*   **Priority:** MUST HAVE
*   **Source:** Concept Document §4, OQ-06
*   **Dependencies:** FR-005
*   **Acceptance Criteria:** AC-010, AC-011

### FR-010: Product Deletion
*   **Description:** The system shall allow users to permanently delete a product from the catalogue.
    *   *Actor:* Administrator
    *   *Trigger:* Admin clicks the delete (🗑) button on a product row in the inventory table.
    *   *Precondition:* Product exists.
    *   *Input:* Confirmation in the deletion dialog.
    *   *Processing Rule:* System prompts for confirmation. Upon confirmation, permanently removes the product record from the database.
    *   *State Change:* Product deleted from SQLite.
    *   *Output:* Product row disappears from the inventory view.
    *   *Error Behaviour:* DB lock/failure aborts the deletion and alerts the user.
    *   *Acceptance Boundary:* Product is permanently removed from DB and UI after confirmation.
*   **Business Objective:** B-001
*   **Priority:** MUST HAVE
*   **Source:** R1
*   **Dependencies:** FR-005
*   **Acceptance Criteria:** AC-017

## 5. Non-Functional Requirements (NFR)

### NFR-001: Standalone Deployment
*   **Category:** Architecture
*   **Description:** The application must run entirely on a single local machine without requiring external network access, cloud dependencies, or external APIs.
*   **Rationale:** Ensures data privacy and operational continuity in environments with unreliable internet.
*   **Verification Method:** Verify 0 external network calls during end-to-end workflow execution.
*   **Priority:** MUST HAVE
*   **Source:** Concept Document §5


### NFR-003: Data Persistence
*   **Category:** Reliability
*   **Description:** All data must be persistently stored in a local SQLite database file, ensuring no data loss between application restarts.
*   **Rationale:** Core requirement for an auditable ledger.
*   **Verification Method:** Restart the Node.js server and verify that inventory and audit logs remain intact.
*   **Priority:** MUST HAVE
*   **Source:** Concept Document §5

### NFR-004: Hardware Scanner Compatibility
*   **Category:** Interoperability
*   **Description:** The application must accept barcode input from standard hardware scanners acting as keyboard emulators, without requiring custom drivers.
*   **Rationale:** Simplifies hardware integration and lowers setup costs.
*   **Verification Method:** Connect a standard USB/Bluetooth barcode scanner and verify it populates the chat input correctly.
*   **Priority:** MUST HAVE
*   **Source:** Concept Document §5

## 6. Requirements Summary Table

| ID | Title | Category | Priority | Business Objective | Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- |
| FR-001 | Chat-Style Product Creation | Function | MUST HAVE | B-001 | None |
| FR-002 | Manual Stock Top-Up | Function | MUST HAVE | B-001 | FR-001 |
| FR-005 | Real-Time Inventory View & Alerts | Function | MUST HAVE | B-003 | FR-001, FR-008 |
| FR-006 | Inline Threshold Editing | Function | MUST HAVE | B-003 | FR-005 |
| FR-008 | Chat-Based Sales Entry | Function | MUST HAVE | B-002 | None |
| FR-009 | Unrecognised Product Fuzzy Match | Function | MUST HAVE | B-002 | FR-008 |
| FR-010 | Product Deletion | Function | MUST HAVE | B-001 | FR-005 |
| NFR-001 | Standalone Deployment | Architecture | MUST HAVE | N/A | None |
| NFR-003 | Data Persistence | Reliability | MUST HAVE | N/A | None |
| NFR-004 | Hardware Scanner Compatibility | Interoperability | MUST HAVE | N/A | None |
