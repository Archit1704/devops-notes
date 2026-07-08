# PostgreSQL Remote Connection Between AWS EC2 Instances (Private IP)

A hands-on DevOps lab demonstrating how to connect a PostgreSQL database running on one AWS EC2 instance from a **separate** EC2 instance, using private VPC networking instead of exposing the database to the public internet.

> **Why this matters:** In real production setups, databases should never be publicly accessible. This lab replicates a common real-world pattern — an application server (or client) talking to a database server privately within the same VPC — the same principle used in three-tier architectures, microservices, and internal tooling.

---

## Table of Contents
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Verification](#verification)
- [What Went Wrong (and How I Fixed It)](#what-went-wrong-and-how-i-fixed-it)
- [Key Takeaways](#key-takeaways)
- [Possible Improvements](#possible-improvements)

---

## Architecture

```
 ┌────────────────────────┐            ┌────────────────────────┐
 │        EC2-1           │            │        EC2-2           │
 │   PostgreSQL Server    │            │   PostgreSQL Client    │
 │   Private IP: 10.0.x.x │◄──────────►│   psql / application    │
 └───────────┬────────────┘   Private   └────────────┬───────────┘
             │              VPC Network               │
             └───────────────────┬───────────────────┘
                       Same VPC / Subnet
                 Security Group allows port 5432
                     only from EC2-2's private IP
```

Both instances live in the **same VPC**, communicate over **private IPs only**, and the PostgreSQL port (5432) is locked down via **Security Groups** to accept traffic only from the client instance — not from the internet.

---

## Tech Stack

| Component        | Details            |
|-------------------|--------------------|
| Cloud Provider    | AWS EC2            |
| OS                | Ubuntu             |
| Database          | PostgreSQL 18      |
| Networking        | AWS VPC, Private IP |
| Access Control    | AWS Security Groups |

---

## Prerequisites

- Two EC2 instances launched in the **same VPC** (and ideally the same subnet)
- SSH access to both instances
- Basic familiarity with Linux and AWS Security Groups

---

## Step-by-Step Setup

### 1. Install PostgreSQL on the server instance (EC2-1)
```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

### 2. Configure PostgreSQL to listen on the private IP
Edit the main config file (path may vary by version, e.g. `/etc/postgresql/18/main/postgresql.conf`):
```bash
sudo nano /etc/postgresql/18/main/postgresql.conf
```
Update the `listen_addresses` line:
```conf
listen_addresses = 'localhost,10.0.14.xxx'
```

### 3. Allow client access in `pg_hba.conf`
```bash
sudo nano /etc/postgresql/18/main/pg_hba.conf
```
Add a rule allowing the client's private IP (or subnet) to connect:
```conf
host    all     all     10.0.14.0/24     md5
```
Restart PostgreSQL to apply changes:
```bash
sudo systemctl restart postgresql
```

### 4. Open port 5432 in the Security Group
On **EC2-1's** Security Group, add an inbound rule:

| Type       | Protocol | Port | Source                          |
|------------|----------|------|----------------------------------|
| Custom TCP | TCP      | 5432 | EC2-2's private IP (or SG ID)    |

> Scoping the source to a specific IP/SG — instead of `0.0.0.0/0` — is what keeps this setup production-realistic.

### 5. Set a password for the `postgres` user (server side)
```bash
sudo -u postgres psql
\password postgres
```

### 6. Connect from the client instance (EC2-2)
```bash
sudo apt update
sudo apt install -y postgresql-client
psql -h 10.0.14.xxx -U postgres
```

---

## Verification

Once connected, create a database, a table, and insert sample data to confirm everything works end-to-end:

```sql
CREATE DATABASE devops_lab;
\c devops_lab

CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    course VARCHAR(100)
);

INSERT INTO students (name, course)
VALUES ('Archit', 'DevOps');

SELECT * FROM students;
```

Expected output:
```
 id |  name  | course
----+--------+---------
  1 | Archit | DevOps
```

---

## What Went Wrong (and How I Fixed It)

No lab is complete without hitting a wall — here's what actually happened:

1. **Different VPCs:** My client and server EC2 instances were initially launched in different VPCs. Private IPs are only routable within the same VPC (or peered VPCs), so the connection silently timed out.
   - **Fix:** Relaunched both instances in the same VPC.

2. **Stale Security Group rule:** After fixing the VPC issue, the connection *still* failed. The Security Group on the server was allowing an old client IP that no longer matched the client's actual private IP.
   - **Fix:** Updated the inbound rule to the client's current private IP.

Once both issues were resolved, `psql` connected immediately.

---

## Key Takeaways

- Private IP communication only works within the same VPC (or across VPCs explicitly peered/routed).
- Security Groups are stateful firewalls — the source IP/SG in the inbound rule must match reality, not just what you *think* the client's IP is.
- `pg_hba.conf` and `postgresql.conf` are two separate gates — PostgreSQL needs **both** configured correctly:
  - `postgresql.conf` → *can* the server listen on that address?
  - `pg_hba.conf` → *who* is allowed to authenticate once they reach it?
- Debugging network issues is often less about code and more about systematically checking each layer: VPC → Subnet → Security Group → PostgreSQL config.

---

## Possible Improvements

- Automate this setup with a shell script or Ansible playbook
- Use AWS Systems Manager (SSM) instead of SSH for a more secure connection to instances
- Add TLS/SSL encryption for PostgreSQL connections
- Deploy across peered VPCs to simulate a multi-account setup
- Store credentials in AWS Secrets Manager instead of setting them manually

---

## Author

**Archit** — DevOps enthusiast, learning cloud infrastructure by building and breaking things.

Feel free to connect, fork this repo, or suggest improvements via a pull request or issue.
