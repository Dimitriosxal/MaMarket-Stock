# Technical Specifications

## 1. System Overview
This document details the cross-cutting technical specifications for the MaMarket Stock MVP. The system is a local monolith exposing a RESTful API to a React frontend.

## 2. Data Architecture (SQLite Schema)

### Table: `products`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `barcode` | TEXT | PRIMARY KEY | Unique product identifier |
| `description` | TEXT | NOT NULL | Product name/description |
| `price` | REAL | NOT NULL | Unit price |
| `uom` | TEXT | NOT NULL | Unit of Measure |

### Table: `inventory`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `barcode` | TEXT | PK, FK(products.barcode) | Links to product |
| `stock_level` | INTEGER | DEFAULT 0 | Current stock (can be negative) |
| `threshold` | INTEGER | DEFAULT 0 | Alert threshold |

### Table: `upload_history`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | INTEGER | PRIMARY KEY AUTOINCREMENT | Unique ID |
| `filename` | TEXT | NOT NULL | Original uploaded filename |
| `upload_date` | DATETIME | DEFAULT CURRENT_TIMESTAMP | Timestamp of upload |
| `status` | TEXT | NOT NULL | 'SUCCESS', 'WARNING', 'ERROR' |
| `total_records` | INTEGER | NOT NULL | Total rows processed |
| `unknown_barcodes` | INTEGER | DEFAULT 0 | Count of unknown barcodes |

## 3. File Parsing Logic (CSV/Excel)
- **Libraries:** `multer` for file upload handling, `csv-parser` for CSV, `xlsx` for Excel files.
- **Format:** Fixed 5-column format: `Barcode`, `Qty Sold`, `Description`, `Price`, `UoM`.
- **Processing Rules:**
  1. Parse file row by row.
  2. Extract `Barcode` and `Qty Sold`.
  3. Query `inventory` for `Barcode`.
  4. **Known Barcode:** Decrement `stock_level` by `Qty Sold`. (Negative values allowed).
  5. **Unknown Barcode:** Create a new entry in `products` using the row's Description, Price, and UoM. Create an `inventory` entry with `stock_level` = `-Qty Sold` and `threshold` = `0`. Flag the row as a warning in the upload summary.

## 4. API Design Standards
- **Base URL:** `http://localhost:3000/api/v1`
- **Format:** JSON payloads.
- **Error Handling:** Standard HTTP status codes (200 OK, 400 Bad Request, 404 Not Found, 500 Internal Server Error). Error responses include a standard JSON envelope: `{ "error": true, "message": "..." }`.

## 5. Non-Functional Requirements (NFRs)
- **Performance:** Local execution ensures sub-100ms API response times. File uploads of up to 10,000 rows should process in under 5 seconds.
- **Concurrency:** Single-user constraint eliminates race conditions. SQLite default locking is sufficient.
