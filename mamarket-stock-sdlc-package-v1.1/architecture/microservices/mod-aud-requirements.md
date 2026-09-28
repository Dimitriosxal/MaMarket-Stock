# MOD-AUD: Upload History & Audit Requirements

## 1. Overview
MOD-AUD provides visibility into past file upload operations. It allows the user to verify that sales data was processed correctly and review any warnings generated during the upload (e.g., unknown barcodes).

## 2. API Surface

### `GET /api/v1/uploads`
- *Description:* Retrieve a full list of past file uploads, ordered by date descending.
- *Response:* `200 OK` with array of `{ id, filename, upload_date, status, total_records, unknown_barcodes }`.

### `GET /api/v1/uploads/:id`
- *Description:* Retrieve detail for a specific upload, including the list of unknown barcodes flagged.
- *Response:* `200 OK` with `{ id, filename, upload_date, status, total_records, unknown_barcodes, warnings[] }`.

## 3. UI/Frontend Implementation Notes
- **Dedicated Screen:** A specific route in the React SPA (e.g., `/history`) will display this data in a tabular format.
- **Status Indicators:** Use visual cues (Green for SUCCESS, Yellow/Orange for WARNING, Red for ERROR) to highlight uploads that contained unknown barcodes or failures.
- **Read-Only:** This module is strictly read-only. Upload history records cannot be deleted or modified by the user in this MVP.
- **Detail Drill-Down:** Clicking a row expands or navigates to the detail view showing the specific unknown barcodes flagged during that upload.

## 4. Data Model Dependencies
- Reads strictly from the `upload_history` table.
- Detail view may join with a future `upload_warnings` table if warning detail is persisted (optional enhancement).

## 5. FR/NFR Traceability
| FR/NFR | Covered By |
| :--- | :--- |
| FR-007 | `GET /api/v1/uploads` + dedicated `/history` React screen |
| NFR-003 | SQLite persistence of upload_history records |
