# Technical Requirements Traceability

## 1. Executive Summary
This document maps the functional and non-functional requirements to the technical implementation details for the MaMarket Stock MVP.

## 2. FR Traceability Matrix

| FR ID | Description | Addressed In | Service/Module |
| :--- | :--- | :--- | :--- |
| FR-001 | Chat-style product creation | `POST /api/v1/products` | MOD-CAT |
| FR-002 | Manual stock top-up | `POST /api/v1/products` (existing barcode path) | MOD-CAT |
| FR-003 | Sold-items file upload & parsing | `POST /api/v1/inventory/upload` | MOD-INV |
| FR-004 | Stock decrement & discrepancy handling | File Parsing Logic + DB Schema (no constraints) | MOD-INV |
| FR-005 | Real-time inventory view & low-stock alerts | `GET /api/v1/inventory` + React alert logic | MOD-INV |
| FR-006 | Inline threshold editing | `PATCH /api/v1/inventory/:barcode` | MOD-INV |
| FR-007 | Upload history (audit trail) | `GET /api/v1/uploads` | MOD-AUD |

## 3. Entity Registry

| Entity | Owning Module | Key Attributes | Data Store |
| :--- | :--- | :--- | :--- |
| Product | MOD-CAT | barcode, description, price, uom | SQLite (`products`) |
| Inventory | MOD-INV | barcode, stock_level, threshold | SQLite (`inventory`) |
| UploadLog | MOD-AUD | id, filename, status, total_records, unknown_barcodes | SQLite (`upload_history`) |

## 4. NFR Coverage Summary

| Category | Target | How Addressed | Reference |
| :--- | :--- | :--- | :--- |
| NFR-001 Standalone Deployment | Single machine, no cloud | Node.js + SQLite local bundle | Solution Architecture §5 |
| NFR-002 Upload Performance | ≤5s for 5,000 rows | SQLite local file I/O + streaming parser | Technical Specs §5 |
| NFR-003 Data Persistence | No data loss on restart | SQLite file-based storage | Technical Specs §2 |
| NFR-004 Hardware Scanner | Keyboard emulator, no drivers | Browser keyboard event listeners in React | MOD-CAT §3 |
