# PostgreSQL Remote Connection using AWS EC2 (Private IP)

## Overview

This project demonstrates how to connect a PostgreSQL database running on one AWS EC2 instance from another EC2 instance using private networking.

## Architecture

EC2-1
- PostgreSQL Server
- Private IP: 10.0.14.xxx

↓

Private VPC Network

↓

EC2-2
- PostgreSQL Client

## Technologies

- AWS EC2
- Ubuntu
- PostgreSQL 18
- AWS Security Groups
- Private IP Networking

## Steps Performed

- Installed PostgreSQL
- Configured PostgreSQL to listen on private IP
- Configured pg_hba.conf
- Opened port 5432 in Security Group
- Connected using private IP
- Created database
- Created table
- Inserted data
- Verified data from server

## Important Learning

I initially failed because:

- The client and server were in different VPCs.
- AWS Security Group still allowed the old client IP.

After fixing the VPC placement and Security Group rule, the connection worked successfully.

## Connection Command

```bash
psql -h <server-private-ip> -U postgres
```

## Sample SQL

```sql
CREATE DATABASE devops_lab;

\c devops_lab

CREATE TABLE students(
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    course VARCHAR(100)
);

INSERT INTO students(name,course)
VALUES('Archit','DevOps');

SELECT * FROM students;
```
