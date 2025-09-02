Perfect 🚀 Ashish — I’ll give you **pure SQL + psql commands** that work in **all PostgreSQL versions** (no version-specific `\` tricks).

---

### 1️⃣ **Get list of databases**

```sql
SELECT datname FROM pg_database;
```

---

### 2️⃣ **Create a new database**

```sql
CREATE DATABASE mydb;
```

---

### 3️⃣ **Connect to that database** (from `psql`)

```bash
\c mydb;
```

(or from shell:)

```bash
psql -h <endpoint> -U postgres -d mydb
```

---

### 4️⃣ **Create a table**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100),
    age INT,
    location VARCHAR(100)
);
```

---

### 5️⃣ **Insert 5 records**

```sql
INSERT INTO users (name, email, age, location) VALUES
('Ashish', 'ashish@example.com', 28, 'Pune'),
('Ravi', 'ravi@example.com', 32, 'Delhi'),
('Sneha', 'sneha@example.com', 25, 'Mumbai'),
('Karan', 'karan@example.com', 30, 'Bangalore'),
('Meera', 'meera@example.com', 27, 'Hyderabad');
```

---

### 6️⃣ **Get data**

```sql
SELECT * FROM users;
```

---

### 7️⃣ **Create backup (Export)**

Run this **from shell (not inside psql):**

```bash
pg_dump -h <endpoint> -U postgres -d mydb -F c -f mydb_backup.dump
```

* `-F c` → custom format (best for restore).
* `-f` → output file.

---

### 8️⃣ **Restore backup (Import)**

First create the database if it doesn’t exist:

```sql
CREATE DATABASE mydb_restore;
```

Then from shell:

```bash
pg_restore -h <endpoint> -U postgres -d mydb_restore -F c mydb_backup.dump
```
