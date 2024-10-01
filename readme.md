I can’t create zip files directly, but I can provide you with the complete code and instructions to set up a Node.js application that connects to a Microsoft SQL Server database, fetches tables, and generates insert scripts. Here’s how to do it:

### Project Structure
```
nodejs-mssql-insert-generator/
├── README.md
├── package.json
├── index.js
└── .env
```

### Step 1: Initialize the Project
1. Create a directory for your project and navigate into it:
   ```bash
   mkdir nodejs-mssql-insert-generator
   cd nodejs-mssql-insert-generator
   ```

2. Initialize a new Node.js project:
   ```bash
   npm init -y
   ```

### Step 2: Install Dependencies
You'll need the `mssql` package to connect to SQL Server:

```bash
npm install mssql dotenv
```

### Step 3: Create `.env` File
Create a `.env` file in the root of your project to store your database connection credentials:

```
DB_USER=your_username
DB_PASSWORD=your_password
DB_SERVER=your_server_address
DB_DATABASE=your_database_name
DB_PORT=1433
```

### Step 4: Create `index.js`
Create `index.js` to handle the logic of connecting to the database and generating SQL scripts.

```javascript
// index.js
require('dotenv').config();
const sql = require('mssql');

async function generateInsertScripts() {
    try {
        const pool = await sql.connect({
            user: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            server: process.env.DB_SERVER,
            database: process.env.DB_DATABASE,
            port: parseInt(process.env.DB_PORT),
            options: {
                encrypt: true, // Use this if you're on Windows Azure
                trustServerCertificate: true, // Change to true for local dev / self-signed certs
            }
        });

        const result = await pool.request().query("SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE'");
        const tables = result.recordset;

        for (const table of tables) {
            const tableName = table.TABLE_NAME;
            const rowsResult = await pool.request().query(`SELECT * FROM ${tableName}`);
            const rows = rowsResult.recordset;

            const insertScripts = rows.map(row => {
                const columns = Object.keys(row).map(col => `[${col}]`).join(", ");
                const values = Object.values(row).map(value => {
                    if (value === null) return 'NULL';
                    return `'${String(value).replace(/'/g, "''")}'`;
                }).join(", ");
                return `INSERT INTO [${tableName}] (${columns}) VALUES (${values});`;
            }).join("\n");

            console.log(`-- Insert statements for table ${tableName}:\n${insertScripts}\n`);
        }

        await pool.close();
    } catch (err) {
        console.error('SQL error', err);
    }
}

generateInsertScripts();
```

### Step 5: Create `README.md`
Add usage instructions in `README.md`.

```markdown
# Node.js MS SQL Server Insert Script Generator

This Node.js application connects to a Microsoft SQL Server database and generates SQL insert scripts for all tables.

## Prerequisites

- Node.js installed
- Microsoft SQL Server database

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your_username/nodejs-mssql-insert-generator.git
   cd nodejs-mssql-insert-generator
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env` file in the root directory and add your database credentials:
   ```
   DB_USER=your_username
   DB_PASSWORD=your_password
   DB_SERVER=your_server_address
   DB_DATABASE=your_database_name
   DB_PORT=1433
   ```

## Usage

Run the script:
```bash
node index.js
```

The insert scripts for all tables will be printed in the console.
```

### Step 6: Create `package.json`
Your `package.json` will already be created when you initialized your project. It should look like this:

```json
{
  "name": "nodejs-mssql-insert-generator",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "mssql": "^8.0.0",
    "dotenv": "^16.0.0"
  }
}
```

### Step 7: Zip the Project
After creating all the files, you can zip your project folder:

```bash
cd ..
zip -r nodejs-mssql-insert-generator.zip nodejs-mssql-insert-generator/
```

### Conclusion
Now you have a zip file named `nodejs-mssql-insert-generator.zip` that contains your entire project. You can share or run it as needed! If you have any questions or need further modifications, let me know!
