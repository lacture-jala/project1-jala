# Installation Postgresql

## Step-1: Setup the updates

```bash
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"
sudo apt update
# sudo apt -y install postgresql
```

```bash
# Therepository contains many different packages including third party addons. The most common and important packages are (substitute the version number as required):

postgresql-client-17	#client libraries and client binaries
postgresql-17	#core database server
postgresql-doc-17	#documentation
libpq-dev	#libraries and headers for C language frontend development
postgresql-server-dev-17	#libraries and headers for C language backend development

```
## Step-2: Install Postgresql 10.x

```bash
#always install client
sudo apt install -y postgresql-client-10
```
## Step-3: Check Version
```bash
psql --version
```
### Output:
```bash
psql (PostgreSQL) 10.23 (Ubuntu 10.23-7.pgdg24.04+1)
```

# RDS Configuration

## 1st db details
```bash
Master username: postgres
Master password: sharepoint
Endpoint: jalaproject.ch28gmcyet55.ap-south-1.rds.amazonaws.com
db: jalaproject
```

## 2nd db details
```bash
Master username: postgres
Master password: sharepoint
Endpoint: jalaproject1.ch28gmcyet55.ap-south-1.rds.amazonaws.com
db: jalaproject1
```

## 1. Install PostgreSQL Client
```bash
sudo apt update
sudo apt install postgresql-client -y
```

## 2. Test Connections
```bash
psql -h jalaproject.ch28gmcyet55.ap-south-1.rds.amazonaws.com -U postgres -d postgres

psql -h jalaproject1.ch28gmcyet55.ap-south-1.rds.amazonaws.com -U postgres -d postgres
```

## 3. Run Migration
Dump from Source DB (10.x)
```bash
pg_dump -h jalaproject.ch28gmcyet55.ap-south-1.rds.amazonaws.com -U postgres -d jalaproject -Fc -f db.dump

# Restore into Target DB (12.x)
pg_restore -h jalaproject1.ch28gmcyet55.ap-south-1.rds.amazonaws.com -U postgres -d jalaproject1 -c db.dump
```

## 4. Send Slack Notification After Migration

Replace <your-webhook-url> with your Slack Webhook:
```bash
STATUS="✅ Database migration from 10.x to 12.x completed successfully"
curl -X POST -H 'Content-type: application/json' \
  --data "{\"text\":\"$STATUS\"}" \
  https://hooks.slack.com/services/<your-webhook-url>
  ```

  Perfect 👍 let’s fix this using **AWS Console UI** step by step.

---

### 🔧 Steps to allow EC2 to access RDS through UI

1. **Find EC2 Security Group**

   * Go to **EC2 → Instances → Your Bastion/EC2**.
   * Scroll down to **Security** tab.
   * Copy the **Security group ID** (example: `sg-0674c31eeb3e08df8`).

---

2. **Modify RDS Security Group**

   * Go to **RDS → Databases → Your DB (jalaproject)**.
   * Under **Connectivity & security**, find **VPC security groups**.
   * Click the **active security group** (`sg-05bf282649afd2f55`).
   * This opens the **EC2 → Security Groups** page.

---

3. **Add Inbound Rule**

   * In the **Inbound rules** tab → click **Edit inbound rules**.

   * **Remove** the rule with `0.0.0.0/0` (for safety).

   * Add new rule:

     * **Type**: PostgreSQL
     * **Port range**: 5432
     * **Source**: Choose **Custom** → paste your EC2 security group ID (`sg-0674c31eeb3e08df8`).

   * Save rules ✅

---

4. **Test from Bastion EC2**

   * SSH into your EC2.
   * Run:

     ```bash
     psql -h jalaproject.ch28gmcyet55.ap-south-1.rds.amazonaws.com -U postgres -d jalaproject
     ```

If username/password are correct → you should get connected 🎉

---

👉 After this, only your **EC2 can connect to RDS**, not the whole internet (much safer).

Do you want me to also give you the **UI steps to install `psql` client** on your EC2 (so you can run the connection test)?












```bash
#Error
postgres=> show databases;
ERROR:  unrecognized configuration parameter "databases"
postgres=> show database;
ERROR:  unrecognized configuration parameter "database"
postgres=> \l
ERROR:  column d.daticulocale does not exist
LINE 8:   d.daticulocale as "ICU Locale",
          ^
HINT:  Perhaps you meant to reference the column "d.datlocale".
postgres=> \list
ERROR:  column d.daticulocale does not exist
LINE 8:   d.daticulocale as "ICU Locale",
          ^
HINT:  Perhaps you meant to reference the column "d.datlocale".
```

Got it 👍 — this error is unusual, it usually happens when the **PostgreSQL client (`psql`) version** is **different from the RDS PostgreSQL engine version**.

Basically, your `psql` tool is sending a query with a column (`d.daticulocale`) that doesn’t exist in your RDS PostgreSQL version.

---

✅ You can fix/try these options:

### 1. Check version mismatch

Run in your `psql` session:

```sql
SELECT version();
```

This tells you the **server (RDS) version**.

And on your EC2 run:

```bash
psql --version
```

This tells you your **client version**.
👉 If your `psql` client is newer than the RDS engine, you may see errors like this.

---

### 2. Workaround: Use SQL instead of `\l`

Instead of `\l`, run:

```sql
SELECT datname FROM pg_database;
```

This will list all databases without relying on client meta-commands.

---

### 3. Upgrade/downgrade your `psql`

If possible, install the same major version of `psql` as your RDS PostgreSQL. For example, if RDS is **PostgreSQL 13**, install `postgresql-client-13`.

On Ubuntu:

```bash
sudo apt update
sudo apt install postgresql-client-13 -y
```

---

👉 For now, try this SQL query in your session:

```sql
SELECT datname FROM pg_database;
```

Do you want me to also give you the commands to install the correct **psql client version** on your EC2 so it matches RDS?


Perfect project 🚀 — this is a **hands-on simulation of AWS networking + RDS + migration + automation**.
Let’s break it step by step so you can **simulate this in AWS** (not necessarily with huge data, but full workflow).

---

## 🔹 Simulation Steps

### **1. Create Two VPCs**

* **VPC-1 (10.x)** → Source DB (Postgres 10 or 11 RDS instance)
* **VPC-2 (12.x)** → Target DB (Postgres 12 RDS instance)
* Each VPC must have:

  * Subnets (private for RDS, public for bastion if needed)
  * Security groups configured for DB ports (5432 for PostgreSQL)

---

### **2. Setup VPC Peering**

* Go to **VPC → Peering connections → Create peering connection**

  * Requester = VPC-1
  * Accepter = VPC-2
* Update **Route Tables** in both VPCs to allow traffic via the peering connection.
* Update **Security Groups** → allow PostgreSQL (5432) access from the Bastion Host and from the peered VPC CIDR.

---

### **3. Create a Bastion Host (EC2)**

* Launch EC2 instance (Ubuntu preferred) in VPC-1 public subnet.
* Install PostgreSQL client tools:

  ```bash
  sudo apt update
  sudo apt install postgresql-client-17 -y
  ```
* This Bastion will connect to both RDS instances.

---

### **4. Create RDS Instances**

* RDS in VPC-1 → PostgreSQL 10.x (source)
* RDS in VPC-2 → PostgreSQL 12.x (target)
* Enable **Public Access = No** (keep private, access via Bastion only).
* Ensure security groups allow Bastion to connect.

---

### **5. Connect & Test**

From Bastion:

```bash
psql -h <source-db-endpoint> -U postgres -d sourcedb
psql -h <target-db-endpoint> -U postgres -d targetdb
```

---

### **6. Data Migration**

Use `pg_dump` + `pg_restore` (or `psql`) for migration:

**Export from Source (10.x):**

```bash
pg_dump -h <source-db-endpoint> -U postgres -d sourcedb -F c -f sourcedb.dump
```

**Import into Target (12.x):**

```bash
pg_restore -h <target-db-endpoint> -U postgres -d targetdb -c sourcedb.dump
```

---

### **7. Automate with Shell Script**

Create `db_migration.sh`:

```bash
#!/bin/bash
SOURCE_HOST="source-rds.endpoint"
TARGET_HOST="target-rds.endpoint"
USER="postgres"
SRC_DB="sourcedb"
TGT_DB="targetdb"
BACKUP="backup.dump"

# Export
pg_dump -h $SOURCE_HOST -U $USER -d $SRC_DB -F c -f $BACKUP

# Import
pg_restore -h $TARGET_HOST -U $USER -d $TGT_DB -c $BACKUP

# Send notification to Slack
curl -X POST -H 'Content-type: application/json' \
--data '{"text":"✅ Database migration from Source to Target completed successfully!"}' \
https://hooks.slack.com/services/<your-slack-webhook>
```

Make executable:

```bash
chmod +x db_migration.sh
./db_migration.sh
```

---

### **8. Validate Migration**

Connect to target DB and check data:

```sql
\c targetdb
SELECT * FROM your_table;
```

---

## 🔹 What You’ll Learn

* VPC Peering setup & routing
* Bastion host concept
* RDS setup in private subnets
* PostgreSQL backup & restore between versions
* Shell scripting for automation
* Slack integration for DevOps notifications

---

👉 Do you want me to **design this as a step-by-step lab guide** (with AWS console + CLI commands) so you can literally implement it end-to-end,
or should I prepare a **Terraform + Bash automation version** so you can spin it up quickly?



---

✅ These will **work on PostgreSQL 9 → 17** without breaking.

Do you want me to also add the **psql meta-command equivalents** (like `\dt`, `\l`) so you have both **SQL** and **shortcut** versions?


Yes 🔥 exactly — that’s why in **real migrations between different PostgreSQL major versions (10 → 11 → 12 → 17)** you cannot rely on `pg_dump` + `pg_restore` with a single version client.

Here’s the rule:

---

### 🔹 Rule of Thumb

* `pg_dump` and `pg_dumpall` → should be run using the **client version that matches the source DB**.
* `pg_restore` → should be run using the **client version that matches the target DB**.

So if you migrate **Postgres 10 → Postgres 11**:

1. Run `pg_dump10` (Postgres 10 client) against the **source (10.x)** DB.
2. Use `pg_restore11` (Postgres 11 client) into the **target (11.x)** DB.

---

### 🔹 Example Commands

**Dump from Postgres 10 source**

```bash
pg_dump10 -h <source-db-10-endpoint> -U postgres -d sourcedb -F c -f sourcedb.dump
```

**Restore into Postgres 11 target**

```bash
pg_restore11 -h <target-db-11-endpoint> -U postgres -d targetdb -c sourcedb.dump
```

---

### 🔹 Multi-version Client Installation (Ubuntu Bastion)

You can install multiple clients side by side:

```bash
sudo apt update
sudo apt install -y postgresql-client-10 postgresql-client-11 postgresql-client-12 postgresql-client-17
```

That gives you commands like:

* `psql10`, `pg_dump10`
* `psql11`, `pg_dump11`, `pg_restore11`
* `psql12`, `pg_dump12`, `pg_restore12`
* `psql17`, `pg_dump17`, `pg_restore17`

---

### 🔹 Alternative (for large production migrations)

For **big DBs** (100GB+), AWS recommends **logical replication** or **AWS DMS (Database Migration Service)** instead of dump/restore.
But for simulation / small projects → multi-version clients with `pg_dump/pg_restore` is best.

---

👉 Do you want me to write a **step-by-step simulation script** (Bash) for your project that:

1. Creates VPC Peering
2. Sets up Bastion with all Postgres clients
3. Dumps from Postgres 10
4. Restores into Postgres 12
5. Posts status to Slack?



Got it ✅ — the error is clear:

* Your **RDS server** = **PostgreSQL 17.4**
* Your **EC2 client** = **PostgreSQL 16.9**

`pg_dump` refuses because client version **16.x cannot safely dump from a 17.x server**.

---

### 🔹 Solution

You need the **Postgres 17 client tools** (`pg_dump17`, `psql17`, etc.) on your EC2 (bastion).

On Ubuntu 24.04, you can install them like this:

```bash
# Add the PostgreSQL official repo
sudo apt update
sudo apt install -y wget gnupg2

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

# Install PostgreSQL 17 client only (not server)
sudo apt update
sudo apt install -y postgresql-client-17
```

---

### 🔹 Usage

Now dump with the **matching version client (17)**:

```bash
pg_dump17 -h jalaproject.ch28gmcyet55.ap-south-1.rds.amazonaws.com -U postgres -d mydb -F c -f mydb_backup.dump
```

And restore into another 17.x instance with:

```bash
pg_restore17 -h <target-db-endpoint> -U postgres -d targetdb -c mydb_backup.dump
```

---

👉 Since you’re simulating **migration between versions (10 → 12, 16 → 17, etc.)**, always install the **source and target version clients** on your Bastion EC2.

Do you want me to give you a **ready one-liner command** that installs **all Postgres clients (10 → 17)** on Ubuntu, so you never face mismatch again?
