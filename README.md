# MySQL Quick Start Guide for Ubuntu

A reference guide for installing, configuring, and working with a local MySQL database on Ubuntu. This document tracks our environment configuration, core setup, and foundational database operations.

---

## 1. Installation and Service Setup

MySQL runs as a background system daemon on Linux. Use the following commands to download, enable, and start the service:

```bash
# Update local package index and install the server
sudo apt update
sudo apt install mysql-server

# Enable MySQL to start automatically on system boot
sudo systemctl enable mysql

# Start the MySQL service immediately
sudo systemctl start mysql

# Verify the database service status (Look for green "active (running)")
sudo systemctl status mysql
```

---

## 2. Setting Up Root Authentication & Passwords

By default, Ubuntu locks MySQL administrative access behind `auth_socket` (requiring `sudo` to login). To transition to secure password-based login for development tools:

1. Log into the active instance using Linux system privileges:
   ```bash
   sudo mysql
   ```

2. Alter the root user authentication plugin and define your secure password (ensure you use the `BY` keyword):
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '7^@#LKJAKJKEHye';
   FLUSH PRIVILEGES;
   EXIT;
   ```

3. Log in going forward from your regular terminal prompt without `sudo`:
   ```bash
   mysql -u root -p
   ```

> **Note on Password Policies:** If MySQL throws `ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`, your string is too weak. You can bypass this temporarily inside the prompt via:
> ```sql
> SET GLOBAL validate_password.policy = LOW;
> SET GLOBAL validate_password.length = 4;
> ```

---

## 3. Core Database & Table Operations

SQL statements require an explicit **semicolon (`;`)** at the end of every statement to execute correctly. 

### Create and Container Setup
You cannot construct a table without explicitly initializing and stepping into a database container first:

```sql
-- 1. Create a namespace container
CREATE DATABASE school;

-- 2. Target the namespace context
USE school;
```

### Table Definitions (DDL)
Creating our initial structure. Note the usage of `BIGINT` for phone numbers to prevent out-of-range structural caps (standard `INT` limits out at roughly 2.1 Billion).

```sql
-- Create the initial table schema
CREATE TABLE student_info (
    roll INTEGER, 
    name TEXT, 
    section TEXT
);

-- Check active tables in the database namespace
SHOW TABLES;
```

---

## 4. 🚨 Life-Saving, Modifying & Diagnostic Commands

When your structure layout needs to change, or you need to inspect your database blueprint, use these essential commands:

### Inspecting Layout Blueprints
If you forget your column names or data types, inspect them without opening the table:
```sql
-- View the structure, data types, and null settings of a table
DESCRIBE student_info;
```

### Modifying Structure (ALTER)
Use these when you accidentally set the wrong data type, misspelled a column, or need to add/remove fields:

```sql
-- Add a brand new column to an existing table
ALTER TABLE student_info ADD phone_number BIGINT;

-- CHANGE DATA TYPE: Fix an "Out of range" error by upgrading an INT to a BIGINT or VARCHAR
ALTER TABLE student_info MODIFY phone_number BIGINT;

-- RENAME A COLUMN: Change a column name (e.g., from 'section' to 'class_section')
ALTER TABLE student_info RENAME COLUMN section TO class_section;

-- DROP A COLUMN: Permanently remove a column and all its contents
ALTER TABLE student_info DROP COLUMN phone_number;

-- RENAME A TABLE: Change the name of the entire table container
ALTER TABLE student_info RENAME TO students;
```

### Resetting & Deleting Safely (The Nuclear Options)
Use these with extreme caution when you need to wipe out mistakes and start over:

```sql
-- Wipe out ALL rows inside a table but keep the layout structure intact
TRUNCATE TABLE student_info;

-- Nuke the entire table layout and all its data completely
DROP TABLE student_info;

-- Nuke the entire database container and everything inside it
DROP DATABASE school;
```

---

## 5. Manipulating Data (DML)

### Inserting New Records
When inserting raw string records like names and sections, wrap the text data arguments in **single quotes (`'`)**:

```sql
INSERT INTO student_info (roll, name, section) 
VALUES (1271064, 'Muhammad Radif Hassan', 'Science 4');
```

### Modifying Pre-existing Rows
To assign or modify values for lines that already exist inside your table array, use an `UPDATE` statement anchored with a specific targeting identifier (`WHERE`). *Warning: If you omit the `WHERE` clause, every single row in the database will be updated!*

```sql
-- Safely update a specific record targeting a single roll number
UPDATE student_info 
SET phone_number = 8801504221579 
WHERE roll = 1271064;
```

### Deleting Specific Rows
```sql
-- Delete a single student row without destroying the rest of the table data
DELETE FROM student_info WHERE roll = 1271064;
```

### Reviewing the Layout Output
Query the relational grid data arrays out to view them inside your active console screen:

```sql
SELECT * FROM student_info;
```
```text
+---------+-----------------------+-----------+--------------+

| roll    | name                  | section   | phone_number |
+---------+-----------------------+-----------+--------------+

| 1271064 | Muhammad Radif Hassan | Science 4 | 8801504221579|
+---------+-----------------------+-----------+--------------+
```

