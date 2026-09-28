# MOD-CAT: Catalog & Product Entry Requirements

## 1. Overview
MOD-CAT is responsible for the creation and management of the product catalog. It accepts input manually via the keyboard or automatically via a hardware barcode scanner acting as a keyboard emulator.

## 2. API Surface

### `GET /api/v1/products`
- *Description:* Retrieve all products.
- *Response:* `200 OK` with array of product objects.

### `GET /api/v1/products/:barcode`
- *Description:* Retrieve a specific product by barcode.
- *Response:* `200 OK` or `404 Not Found`.

### `POST /api/v1/products`
- *Description:* Create a new product.
- *Payload:* `{ barcode, description, price, uom, initial_quantity }`
- *Behavior:* Inserts into `products` table. Automatically creates a corresponding row in the `inventory` table with `stock_level = initial_quantity` and `threshold = 0`.
- *Error:* If barcode already exists, returns `409 Conflict` with message "Barcode already exists. Use top-up flow."

### `PATCH /api/v1/products/:barcode/topup`
- *Description:* Add stock quantity to an existing product.
- *Payload:* `{ quantity: Number }`
- *Behavior:* Increments `inventory.stock_level` by the given quantity.
- *Error:* `404 Not Found` if barcode does not exist; `400 Bad Request` if quantity is not a positive integer.

## 3. UI/Frontend Implementation Notes
- **Scanner Integration:** The frontend React component must listen for rapid sequential keystrokes followed by an `Enter` key event (standard behavior for USB barcode scanners) to automatically trigger the product lookup or creation flow.
- **Chat Flow:** On barcode entry, the system checks existence: if new → prompt for description, price, uom, initial quantity; if existing → prompt for top-up quantity.
- **Validation:** Ensure `barcode` uniqueness check before submission to prevent 409 errors.

## 4. FR/NFR Traceability
| FR/NFR | Covered By |
| :--- | :--- |
| FR-001 | `POST /api/v1/products` (new barcode path) |
| FR-002 | `PATCH /api/v1/products/:barcode/topup` |
| NFR-004 | Browser keyboard event listener for scanner |
