# MOD-INV: Inventory & File Upload Requirements

## 1. Overview
MOD-INV is the core operational module. It displays current stock levels, allows users to set low-stock thresholds inline, and processes sales data files to decrement stock.

## 2. API Surface

### `GET /api/v1/inventory`
- *Description:* Retrieve inventory joined with product details.
- *Response:* `200 OK` with array of `{ barcode, description, stock_level, threshold, alert: boolean }`.
- *Alert logic:* `alert = stock_level <= threshold`.

### `PATCH /api/v1/inventory/:barcode`
- *Description:* Update the reorder threshold for a product.
- *Payload:* `{ threshold: Number }`
- *Validation:* Must be a non-negative integer.
- *Response:* `200 OK` with updated inventory row; `400 Bad Request` on invalid input.

### `POST /api/v1/inventory/upload`
- *Description:* Upload CSV/Excel file containing sales data.
- *Payload:* `multipart/form-data` with file field `sales_file`.
- *Accepted formats:* `.csv`, `.xls`, `.xlsx`.
- *Response:* `200 OK` with upload summary `{ upload_id, total_records, decremented, unknown_barcodes, warnings[] }`.

## 3. Business Logic & Processing Rules
- **Inline Editing:** The frontend renders the `threshold` column as an editable input. On blur/enter, it triggers the `PATCH` endpoint. Alert status recalculates immediately on the frontend.
- **Negative Stock:** The system explicitly permits `stock_level` to drop below zero. No database constraints shall prevent this.
- **Unknown Barcodes:** During file processing, if a barcode is not found in the `products` table:
  1. Auto-create the product using the row's Description, Price, and UoM data.
  2. Set its `stock_level` to the negative value of the sold quantity.
  3. Set `threshold = 0`.
  4. Increment the `unknown_barcodes` counter for the upload audit log.
  5. Include the barcode in the `warnings[]` array of the response.
- **File Rejection:** If the file has fewer than 5 columns or is unreadable, reject immediately before any DB writes with `400 Bad Request`.

## 4. Data Model Dependencies
- Reads/Writes: `inventory` and `products` tables.
- Writes: `upload_history` table upon completion of file processing.

## 5. FR/NFR Traceability
| FR/NFR | Covered By |
| :--- | :--- |
| FR-003 | `POST /api/v1/inventory/upload` — parsing logic |
| FR-004 | File processing rules — negative stock + unknown barcode handling |
| FR-005 | `GET /api/v1/inventory` + React alert badge logic |
| FR-006 | `PATCH /api/v1/inventory/:barcode` |
| NFR-002 | Streaming file parser, SQLite batch writes |
| NFR-003 | SQLite persistence |
