# DVWA & Database Setup

Documentation of the DVWA web application deployment, MariaDB installation on the DB-Server, remote database architecture, user permissions, and the fake data populated for SQL injection practice.

## DVWA Installation

DVWA was installed on the Web-Server (172.16.0.200) using a third-party automated script that handles Apache, PHP, and DVWA setup:

```bash
sudo bash -c "$(curl --fail --show-error --silent --location https://raw.githubusercontent.com/IamCarron/DVWA-Script/main/Install-DVWA.sh)"
```

DVWA is accessible at `http://172.16.0.200/DVWA`.

## Architecture

DVWA's database runs on a separate server from the web application, connected across subnets through a pfSense firewall rule allowing port 3306.

| Component | Host | IP | Subnet |
|---|---|---|---|
| DVWA Web App | Web-Server | 172.16.0.200 | DMZ_NET |
| MariaDB | DB-Server | 192.168.10.50 | LAN_NET |

![DVWA to Database network topology](/links/screenshots/db/network-topo-db.png)

## MariaDB Installation & Configuration

### Bind Address

MariaDB is configured to listen on all interfaces so the webserver can connect remotely.

In `/etc/mysql/mariadb.conf.d/50-server.cnf`:

```
bind-address = 0.0.0.0
```

```bash
sudo systemctl restart mariadb
```

### Database and User Creation

```sql
sudo mysql

CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'172.16.0.200' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'172.16.0.200';
FLUSH PRIVILEGES;
EXIT;
```

The `dvwa` user is locked to `172.16.0.200` (the webserver's IP) so only that host can authenticate.

## DVWA Remote Database Configuration

On the webserver, `/var/www/html/DVWA/config/config.inc.php` is configured to point at the remote DB-Server:

```php
$_DVWA['db_server']   = '192.168.10.50';
$_DVWA['db_database'] = 'dvwa';
$_DVWA['db_user']     = 'dvwa';
$_DVWA['db_password'] = 'ezpass';
$_DVWA['db_port']     = '3306';
```

DVWA's default tables (`users`, `guestbook`) were created by navigating to `http://172.16.0.200/DVWA/setup.php` and clicking **Create / Reset Database**.

## Fake Data for SQL Injection Practice

A custom SQL script was imported to populate the database with realistic fake data for SQL injection exercises:

```bash
sudo mysql dvwa < /path/to/dvwa_fake_data.sql
```

### Tables

| Table | Records | Contents |
|---|---|---|
| `customers` | 30 | PII — names, emails, phone numbers, SSNs, addresses |
| `credit_cards` | 30 | Card numbers, CVVs, expiry dates, balances (linked to customers) |
| `employees` | 20 | Employee records — salaries, SSNs, department hierarchy |
| `medical_records` | 15 | Blood types, medications, conditions, insurance info (linked to customers) |
| `user_credentials` | 20 | Usernames, MD5 hashes, plain-text passwords, security Q&A (linked to customers) |
| `transactions` | 40 | Purchase history with merchants, amounts, statuses (linked to customers/cards) |
| `admin_notes` | 8 | Internal notes — fake credentials, incident reports, compliance findings |

All data is entirely fictional and intentionally insecure (plain-text passwords, unencrypted card numbers, etc.) to serve as realistic targets for SQL injection discovery.
