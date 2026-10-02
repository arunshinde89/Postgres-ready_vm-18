# PostgreSQL 18 Ready Lab VM

This project creates a ready-to-use PostgreSQL 18 lab environment using Vagrant, VMware Workstation, and Rocky Linux 8.

The goal is to quickly create a PostgreSQL practice VM for DBA learning without manually installing and configuring everything every time.

## Project Overview

Running:

```powershell
vagrant up
```

will automatically:

- Create a Rocky Linux 8 virtual machine
- Configure 4 GB RAM
- Configure 2 CPUs
- Configure hostname as `postgres-vm`
- Configure private IP `192.168.136.10`
- Enable SSH password authentication
- Install PostgreSQL 18
- Initialize the PostgreSQL database cluster
- Enable PostgreSQL service at startup
- Start PostgreSQL automatically
- Create separate PostgreSQL practice filesystems

## Environment

| Component | Configuration |
|---|---|
| Operating System | Rocky Linux 8 |
| Virtualization | VMware Workstation |
| Provisioning | Vagrant |
| PostgreSQL | PostgreSQL 18 |
| PostgreSQL Port | 5432 |
| Hostname | postgres-vm |
| Private IP | 192.168.136.10 |
| RAM | 4 GB |
| CPU | 2 |

## PostgreSQL Configuration

PostgreSQL is installed using the official PostgreSQL PGDG repository.

Default PostgreSQL data directory:

```text
/var/lib/pgsql/18/data
```

PostgreSQL service:

```text
postgresql-18.service
```

Default PostgreSQL port:

```text
5432
```

## Storage Layout

The VM creates separate XFS filesystems for PostgreSQL DBA practice.

| Filesystem | Size | Purpose |
|---|---:|---|
| `/pgwal` | 10 GB | WAL practice |
| `/pgarchive` | 10 GB | WAL archive practice |
| `/pgtabs` | 20 GB | PostgreSQL tablespaces |
| `/pgbackup` | 20 GB | PostgreSQL backups |

The PostgreSQL data directory remains at the default location:

```text
/var/lib/pgsql/18/data
```

Note: `/pgwal` is currently prepared as a separate filesystem for future WAL exercises. PostgreSQL still uses the default `pg_wal` directory inside `PGDATA`.

## Prerequisites

Install the following software on your host machine:

- VMware Workstation
- Vagrant
- Vagrant VMware Desktop plugin

Check Vagrant:

```powershell
vagrant --version
```

Check VMware plugin:

```powershell
vagrant plugin list
```

You should see:

```text
vagrant-vmware-desktop
```

## Clone the Repository

Clone this project:

```powershell
git clone https://github.com/YOUR-USERNAME/Postgres-ready_vm-18.git
```

Go to the project directory:

```powershell
cd Postgres-ready_vm-18
```

## Create the VM

Run:

```powershell
vagrant up
```

Vagrant will create the VM and automatically provision PostgreSQL 18.

The first build can take several minutes because Rocky Linux packages and PostgreSQL packages must be downloaded and installed.

## Check VM Status

Run:

```powershell
vagrant status
```

Expected result:

```text
default    running (vmware_desktop)
```

## Connect to the VM

Using Vagrant:

```powershell
vagrant ssh
```

You can also use PuTTY with:

```text
IP: 192.168.136.10
Port: 22
```

## Check PostgreSQL Service

Inside the VM:

```bash
systemctl status postgresql-18
```

The service should show:

```text
Active: active (running)
```

## Connect to PostgreSQL

Switch to the PostgreSQL OS user:

```bash
su - postgres
```

Connect using `psql`:

```bash
/usr/pgsql-18/bin/psql
```

or:

```bash
psql
```

## Verify PostgreSQL Version

Inside `psql`:

```sql
SELECT version();
```

Example:

```text
PostgreSQL 18.x
```

## Check PostgreSQL Data Directory

```sql
SHOW data_directory;
```

Expected:

```text
/var/lib/pgsql/18/data
```

## Check PostgreSQL Port

```sql
SHOW port;
```

Expected:

```text
5432
```

## Check Practice Filesystems

Run:

```bash
df -hT | grep -E 'pgwal|pgarchive|pgtabs|pgbackup'
```

Expected filesystems:

```text
/pgwal
/pgarchive
/pgtabs
/pgbackup
```

## Vagrant Commands

Start the VM:

```powershell
vagrant up
```

Check VM status:

```powershell
vagrant status
```

SSH into the VM:

```powershell
vagrant ssh
```

Gracefully stop the VM:

```powershell
vagrant halt
```

Start it again:

```powershell
vagrant up
```

Suspend the VM:

```powershell
vagrant suspend
```

Resume the VM:

```powershell
vagrant resume
```

Delete the VM:

```powershell
vagrant destroy -f
```

Recreate the complete PostgreSQL lab:

```powershell
vagrant up
```

## Project Structure

```text
Postgres-ready_vm-18/
│
├── Vagrantfile
├── README.md
└── .gitignore
```

The `.vagrant` directory and packaged `.box` files are excluded from GitHub.

Example `.gitignore`:

```text
.vagrant/
*.box
```

## Future Lab Exercises

This VM can later be used for PostgreSQL DBA practice such as:

- PostgreSQL roles and users
- Databases and schemas
- `pg_hba.conf`
- `postgresql.conf`
- Tablespaces
- WAL management
- WAL archiving
- `pg_dump`
- `pg_restore`
- `pg_dumpall`
- `pg_basebackup`
- Point-in-Time Recovery
- Streaming replication
- PostgreSQL performance tuning
- Autovacuum
- Backup and recovery
- Multiple PostgreSQL instances
- PgBouncer
- Patroni
- HAProxy
- etcd
- Prometheus and Grafana monitoring

## Purpose

This project is created as a PostgreSQL DBA learning and practice environment.

Instead of manually creating a Linux VM and installing PostgreSQL every time, the complete PostgreSQL lab can be recreated using:

```powershell
vagrant up
```

This makes the environment reproducible and useful for PostgreSQL administration, backup, recovery, performance, replication, and high-availability practice.
