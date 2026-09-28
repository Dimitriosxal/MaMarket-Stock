# Solution Architecture: MaMarket Stock

## 1. Executive Summary
The MaMarket Stock application is a standalone, single-machine web application designed to manage product inventory, process sales data via file uploads, and maintain an audit trail of operations. As a greenfield MVP, the architecture prioritizes simplicity, ease of local deployment, and zero-configuration operation. It operates entirely on a single local machine without network sharing or cloud dependencies.

## 2. System Architecture Diagram

```text
+--------------------------------------------------+
|                  User Interface                  |
|  (React SPA, Browser on Local Machine, Scanner)  |
+------------------------+-------------------------+
                         | HTTP/REST (localhost)
+------------------------v-------------------------+
|                  Backend API                     |
|             (Node.js / Express.js)               |
|                                                  |
|  +-----------+   +-----------+   +-----------+   |
|  |  MOD-CAT  |   |  MOD-INV  |   |  MOD-AUD  |   |
|  +-----------+   +-----------+   +-----------+   |
+------------------------+-------------------------+
                         | SQL (Local File)
+------------------------v-------------------------+
|                  Local Database                  |
|                     (SQLite)                     |
+--------------------------------------------------+
```

## 3. Technology Stack
- **Frontend:** React.js (Single Page Application). Chosen for rapid UI development and component reusability.
- **Backend:** Node.js with Express.js. Lightweight, asynchronous, and easy to bundle for local execution.
- **Database:** SQLite. Serverless, zero-configuration, file-based relational database perfectly suited for a single-machine, single-user MVP.
- **Hardware Integration:** Standard USB/Bluetooth Barcode Scanner (configured as a keyboard emulator). No custom drivers required.

## 4. Component Architecture
- **MOD-CAT (Catalog/Product Entry):** Handles manual and scanner-based product entry.
- **MOD-INV (Inventory & Upload):** Manages current stock levels, inline threshold editing, and processes CSV/Excel files to decrement stock.
- **MOD-AUD (Audit/History):** Provides a dedicated screen to view the history and status of uploaded sales files.

## 5. Deployment Architecture
The application will be deployed locally on the user's machine.
- **Packaging:** The Node.js backend and React static build can be wrapped using a tool like PM2 or packaged via Electron/pkg for a simple double-click execution.
- **Data Storage:** The SQLite database file (`mamarket.sqlite`) will reside in a local application data folder, ensuring persistence across restarts.
