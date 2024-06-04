---
layout: post
title: postgresql
date: 2024-06-04
last_modified: 2024-06-04
comments: true
---

tldr; install PostgreSQL on Ubuntu 22.04/24.04 or Fedora Linux.

## Ubuntu

### Install

Install PostgreSQL and start the command line tool psql.

```bash
sudo apt update
sudo apt install postgresql
sudo -u postgres psql
```

## Fedora

### Install

```bash
sudo dnf install postgresql-server postgresql-contrib
sudo systemctl enable postgresql
sudo postgresql-setup --initdb --unit postgresql
sudo systemctl start postgresql
sudo -u postgres psql
```

## configuration


> By default, only connections from the local system are allowed. To enable all other computers to connect to your PostgreSQL server, edit the file /etc/postgresql/*/main/postgresql.conf. Locate the line: #listen_addresses = ‘localhost’ and change it to *:

```conf
listen_addresses = '*'
```

> After configuring the password, edit the file /etc/postgresql/*/main/pg_hba.conf to use scram-sha-256 authentication with the postgres user, allowed for all databases, from any network connection

```conf
# TYPE  DATABASE USER CIDR-ADDRESS  METHOD
host    all      all  0.0.0.0/0     scram-sha-256
```

Enable sql account for postgres user.

```sql
ALTER USER postgres WITH PASSWORD 'your_new_password';
```

## References


* [Ubuntu Install and configure PostgreSQL](https://ubuntu.com/server/docs/install-and-configure-postgresql)
* [Fedora PostgreSQL](https://docs.fedoraproject.org/en-US/quick-docs/postgresql/)


