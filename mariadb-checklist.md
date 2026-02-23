# MariaDB / MySQL Remote Access and Scoring User Setup  
Full guide to enable remote access, create a scoring user, and grant read/write access.

---

## 1. Start MariaDB and Verify Service

```bash
sudo systemctl start mariadb
sudo systemctl status mariadb
ss -tlnp | grep 3306
```

---

## 2. Enable Remote Connections

Edit the MariaDB server config. On Debian/Ubuntu it’s usually:

```bash
/etc/mysql/mariadb.conf.d/50-server.cnf
```

Open:
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Find:
```conf
bind-address = 127.0.0.1
```

Change to:
```conf
bind-address = 0.0.0.0
```

Restart MariaDB:
```bash
sudo systemctl restart mariadb
```

---

## 3. Connect to MariaDB Shell

```bash
sudo mysql
```

---

## 4. Create Database, User, and Grant Permissions

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS scoringdb;

-- Create user
CREATE USER IF NOT EXISTS 'scoring'@'%' IDENTIFIED BY 'strong_password_here';

-- Grant read/write permissions on the database
GRANT SELECT, INSERT, UPDATE, DELETE
ON scoringdb.*
TO 'scoring'@'%';

-- Apply privileges
FLUSH PRIVILEGES;
```

---

## 5. Create Example Table

```sql
USE scoringdb;

CREATE TABLE IF NOT EXISTS score_test (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    note VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 6. Useful MariaDB/MySQL Commands

```sql
SHOW DATABASES;
USE scoringdb;
SHOW TABLES;

SELECT User, Host FROM mysql.user;
SELECT * FROM score_test;
```

---

## 7. Test Remote Connection

```bash
mysql -h <SERVER_IP> -u scoring -p scoringdb
```

Inside MySQL:

```sql
INSERT INTO score_test (note) VALUES ('hello from remote');
SELECT * FROM score_test;
```

---
