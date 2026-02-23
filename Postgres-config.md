# PostgreSQL Remote Access and Scoring User Setup  
Full guide to enable remote access, create a scoring user, and grant read/write access.

## 1. Start PostgreSQL and Verify Service

```bash
sudo systemctl start postgresql
sudo systemctl status postgresql
ss -tlnp | grep 5432
```
Check your Postrges version
```bash
psql --version
```
---

## 2. Enable Remote Connections

### A) Edit postgresql.conf

Path:
```bash
/etc/postgresql/<version>/main/postgresql.conf
```

Edit:
```bash
sudo nano /etc/postgresql/<version>/main/postgresql.conf
```

Find:
```conf
#listen_addresses = 'localhost'
```

Replace with:
```conf
listen_addresses = '*'
```

Ensure:
```conf
port = 5432
```

---

### B) Edit pg_hba.conf

Path:
```bash
/etc/postgresql/<version>/main/pg_hba.conf
```

Edit:
```bash
sudo nano /etc/postgresql/<version>/main/pg_hba.conf
```

Add:
```conf
host    scoringdb    scoring    0.0.0.0/0       md5
```

Reload:
```bash
sudo systemctl restart postgresql
```

---

## 3. Connect as postgres Superuser

```bash
sudo -u postgres psql
```

---

## 4. Create Database, User, and Grant Permissions

```sql
CREATE DATABASE scoringdb;

CREATE USER scoring WITH PASSWORD 'strong_password_here';

GRANT CONNECT ON DATABASE scoringdb TO scoring;

\c scoringdb

GRANT USAGE ON SCHEMA public TO scoring;

GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO scoring;

GRANT ALL PRIVILEGES
ON ALL SEQUENCES IN SCHEMA public
TO scoring;
```

---

## 5. Set Default Privileges for Future Tables

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO scoring;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT ALL PRIVILEGES ON SEQUENCES TO scoring;
```

---

## 6. Create Example Table

```sql
CREATE TABLE IF NOT EXISTS score_test (
    id SERIAL PRIMARY KEY,
    note VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);
```

---

## 7. Useful PostgreSQL Commands

```sql
\l
\du
\dt
SELECT * FROM score_test;
```

---

## 8. Test Remote Connection

```bash
psql -h <SERVER_IP> -U scoring -d scoringdb
```

Then inside psql:
```sql
INSERT INTO score_test (note) VALUES ('hello from remote');
SELECT * FROM score_test;
```

---
