# Kali Lab Rebuild: SQL, SSH, Users, and Splunk Prep

## Project Context

This lab documents a fresh Kali Linux machine created specifically for hands-on lab work. The goal was to build a cleaner environment for SQL practice, Splunk preparation, Linux administration practice, and future SOC-style workflows.

This was not only a SQL lab. It was a full lab rebuild and setup exercise: updating Kali, preparing tools, practicing user/root administration, restoring SSH, rebuilding the SQL tax lab database, and beginning the Splunk installation process.

## Lab Goals

- Start with a clean Kali machine dedicated to lab work
- Update and upgrade the system
- Prepare tools needed for future SQL and SOC labs
- Practice Linux user/root administration
- Restore and confirm SSH service access
- Rebuild the SQL tax lab database
- Download and prepare Splunk
- Document where I got stuck for the next troubleshooting step

## Tools Planned for This Lab Machine

- **Splunk** — for SIEM/log analysis practice
- **Konsole** — for split-view terminal work
- **MariaDB / MySQL-style SQL workflow** — for database practice
- **SSH** — for remote access and Linux service practice

## Phase 1: Fresh Kali Machine

I started a new Kali machine just for the lab. This helped separate the SQL/Splunk practice from older lab work and gave me a cleaner environment to build from.

## Phase 2: Basic System Update

The first basic command was:

```bash
sudo apt update
```

Then I moved into a full upgrade:

```bash
sudo apt upgrade -y
```

The upgrade took a while, which is normal on a new Kali setup depending on package count, internet speed, and repository status.

## Phase 3: Root and User Administration

One major lesson from this lab was that a safer workflow is to create a normal user, give that user sudo permissions, and use privilege escalation only when needed.

During this lab I practiced:

- Accessing root
- Changing the root password
- Creating a new user
- Giving the new user sudo/su power
- Using `su -` for the first time in roughly two years of working with Linux labs

Example commands related to this phase:

```bash
sudo su
passwd
adduser labuser
usermod -aG sudo labuser
su - labuser
```

## Phase 4: SSH Service Restored

I got SSH working again and confirmed the service was running.

Useful service commands:

```bash
sudo systemctl status ssh
sudo systemctl start ssh
sudo systemctl enable ssh
```

This step matters because SSH is a core Linux administration and troubleshooting skill. It also prepares the lab for future remote access and log-analysis practice.

## Phase 5: SQL Tax Lab Rebuild

I learned and used commands to delete and recreate the SQL lab database cleanly.

Commands used:

```sql
DROP DATABASE IF EXISTS tax_lab;
CREATE DATABASE tax_lab;
USE tax_lab;
```

This gave me a clean starting point instead of trying to fix messy old table data.

## Recreated SQL Table

After rebuilding the database, I recreated the table for the tax lab.

The tax lab is being used as practice data for SQL commands, table design, and future relational database work.

Example table concept:

```sql
CREATE TABLE tax_dept (
  dept_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  job_title VARCHAR(100),
  tax_area VARCHAR(100),
  email VARCHAR(100),
  phone VARCHAR(20),
  hire_date DATE,
  status VARCHAR(30),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Why DROP DATABASE Was Useful

The command:

```sql
DROP DATABASE IF EXISTS tax_lab;
```

means:

- If the database exists, delete it
- If it does not exist, do not throw an error
- Start clean before rebuilding the lab

This is useful in a lab environment, but it should be handled carefully in real environments because it deletes the database.

## Splunk Preparation

After rebuilding the Kali lab and SQL database, I moved toward downloading Splunk for future SIEM/log-analysis work.

This is where I got stuck.

That is now the next troubleshooting point for the lab.

## Current Status

Completed:

- New Kali lab machine started
- System update started
- Full upgrade attempted
- Root password changed
- New user created
- User privilege practice completed
- `su -` practiced
- SSH restored and service running
- SQL tax lab database dropped/recreated
- SQL table recreated
- Splunk download process started

Blocked / Next issue:

- Splunk download/install process needs troubleshooting

## What I Learned

1. A clean lab machine makes work easier to document.
2. `apt update` and `apt upgrade -y` can take time on a fresh Kali install.
3. Root should be handled carefully; a normal sudo user is better for daily lab work.
4. `su -` gives a cleaner full login shell when switching users.
5. SSH service practice is useful for Linux administration and future log analysis.
6. `DROP DATABASE IF EXISTS` is useful for rebuilding lab databases cleanly.
7. Getting stuck is still documentation-worthy because it shows the next troubleshooting target.

## Next Steps

- Troubleshoot Splunk download or installation
- Confirm Splunk package type for Kali/Debian
- Install Splunk successfully
- Start Splunk service
- Confirm Splunk Web access
- Begin ingesting Linux logs
- Connect SQL/Splunk work into a stronger SOC/data analytics portfolio story

## Portfolio Value

This lab shows more than one tool. It shows environment setup, Linux administration, service troubleshooting, SQL database rebuilds, and preparation for SIEM work.

The professional value is not that everything worked perfectly. The value is that the work was built, checked, documented, and continued even after getting stuck.
