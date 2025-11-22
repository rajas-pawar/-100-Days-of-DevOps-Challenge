# Day 17: Setting Up PostgreSQL Database and User
**100 Days Cloud DevOps Challenge – KodeKloud**

---

## Objective

The Nautilus application development team is planning to deploy a newly developed application on Nautilus infrastructure in Stratos DC. The application uses a PostgreSQL database, so we need to set up the PostgreSQL database server as per the requirements:

**Requirements:**
- PostgreSQL database server is already installed on the Nautilus database server
- Create a database user `kodekloud_gem` with password `BruCStnMT5`
- Create a database `kodekloud_db10` and grant full permissions to user `kodekloud_gem`

**Note:** Do not restart PostgreSQL server service.

---

## Environment Details

| Server    | User     | Service    |
|-----------|----------|------------|
| DB Server | postgres | postgresql |

---

## Step-by-Step Implementation

### 1. SSH into Database Server
```bash
ssh peter@stdb01
```

### 2. Switch to PostgreSQL User
```bash
sudo su - postgres
```

### 3. Access PostgreSQL Shell
```bash
psql
```

### 4. Create Database User
```sql
CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';
```

**Output:**
```
CREATE ROLE
```

### 5. Create Database
```sql
CREATE DATABASE kodekloud_db10;
```

**Output:**
```
CREATE DATABASE
```

### 6. Grant Full Permissions
```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db10 TO kodekloud_gem;
```

**Output:**
```
GRANT
```

### 7. Verify User Creation
```sql
\du
```

**Output:**
```
                                     List of roles
   Role name   |                         Attributes                         | Member of 
---------------+------------------------------------------------------------+-----------
 kodekloud_gem |                                                            | {}
 postgres      | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

### 8. Verify Database and Permissions
```sql
\l
```

**Output:**
```
                                  List of databases
      Name      |  Owner   | Encoding  | Collate | Ctype |     Access privileges      
----------------+----------+-----------+---------+-------+----------------------------
 kodekloud_db10 | postgres | SQL_ASCII | C       | C     | =Tc/postgres              +
                |          |           |         |       | postgres=CTc/postgres     +
                |          |           |         |       | kodekloud_gem=CTc/postgres
```

**Note:** The access privileges show that `kodekloud_gem` has been granted privileges (CTc) on the database.

### 9. Exit PostgreSQL
```sql
\q
```

---

## Full Command Block (Copy-Paste)
```bash
# Switch to postgres user
sudo su - postgres

# Access PostgreSQL
psql
```

**Execute these commands one by one:**
```sql
CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';

CREATE DATABASE kodekloud_db10;

GRANT ALL PRIVILEGES ON DATABASE kodekloud_db10 TO kodekloud_gem;

\du

\l

\q
```

---

## Understanding Access Privileges

In the `\l` output, the access privileges column shows:
```
kodekloud_gem=CTc/postgres
```

This means:
- **C** = CREATE - Can create new schemas in the database
- **T** = TEMPORARY - Can create temporary tables
- **c** = CONNECT - Can connect to the database

The format is: `grantee=privileges/grantor`

---

## Alternative: One-Line Commands from Bash

Execute all commands without entering psql shell:
```bash
sudo su - postgres

psql -c "CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';"
psql -c "CREATE DATABASE kodekloud_db10;"
psql -c "GRANT ALL PRIVILEGES ON DATABASE kodekloud_db10 TO kodekloud_gem;"
psql -c "\du"
psql -c "\l"
```

---

## Verification Steps

### Test Database Connection as New User
```bash
# Exit from postgres user
exit

# Test connection (will prompt for password)
psql -U kodekloud_gem -d kodekloud_db10 -h localhost -W
```

Enter password: `BruCStnMT5`

**Expected:**
```
kodekloud_db10=>
```

Test creating a table:
```sql
CREATE TABLE test_table (id INT);
```

**Output:**
```
CREATE TABLE
```

List tables:
```sql
\dt
```

Exit:
```sql
\q
```

---

## Troubleshooting

### Issue 1: Stuck in Multi-line Mode (`postgres-#`)

**Symptoms:** You see `postgres-#` instead of `postgres=#`

**Cause:** Incomplete command or accidental text entry

**Solution:**
```sql
;
```
Or press `Ctrl + C` to cancel

### Issue 2: User Already Exists

**Error:** `ERROR: role "kodekloud_gem" already exists`

**Solution:**
```sql
DROP USER kodekloud_gem;
CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';
```

### Issue 3: Database Already Exists

**Error:** `ERROR: database "kodekloud_db10" already exists`

**Solution:**
```sql
DROP DATABASE kodekloud_db10;
CREATE DATABASE kodekloud_db10;
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db10 TO kodekloud_gem;
```

### Issue 4: Permission Denied

Ensure you're running as the `postgres` user:
```bash
sudo su - postgres
```

---

## PostgreSQL Prompt Guide

| Prompt      | Meaning                           |
|-------------|-----------------------------------|
| `postgres=#` | Ready for command (superuser)    |
| `postgres=>` | Ready for command (normal user)  |
| `postgres-#` | Waiting for command continuation |
| `postgres-'` | Waiting for closing quote        |

**Tip:** Always ensure you see `=#` or `=>` before entering a new command.

---

## Key PostgreSQL Commands

| Command                  | Description                    |
|--------------------------|--------------------------------|
| `CREATE USER`            | Create a new database user     |
| `CREATE DATABASE`        | Create a new database          |
| `GRANT ALL PRIVILEGES`   | Grant full permissions         |
| `\du`                    | List all users/roles           |
| `\l`                     | List all databases             |
| `\c database_name`       | Connect to a database          |
| `\dt`                    | List tables in current DB      |
| `\q`                     | Exit PostgreSQL shell          |
| `;`                      | Terminate incomplete command   |
| `Ctrl + C`               | Cancel current command         |

---

## Key Takeaways

* PostgreSQL uses role-based access control (users are roles)
* `GRANT ALL PRIVILEGES` provides full database access
* Access privileges are displayed in format: `grantee=privileges/grantor`
* Always terminate SQL commands with semicolon (`;`)
* Use `\du` and `\l` to verify user and database creation
* No PostgreSQL service restart is needed for these operations
* Test connectivity with `psql -U username -d database`

---

## Task Completion Checklist

- [x] User `kodekloud_gem` created with password `BruCStnMT5`
- [x] Database `kodekloud_db10` created
- [x] Full privileges granted to user on database
- [x] Verified user exists using `\du`
- [x] Verified database exists using `\l`
- [x] Confirmed access privileges in database list

---

## Completion Details

- **Completion Date:** November 22, 2025
- **Day:** 17 / 100
- **Challenge:** KodeKloud 100 Days Cloud DevOps
- **Topic:** PostgreSQL Database and User Configuration
- **Status:** ✅ Successfully Completed