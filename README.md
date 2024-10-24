# Migration-from-AWS-RDS-MySQL-5.6-to-MySQL-8.35-with-Minimal-Downtime
This repository details the migration of a 1TB AWS RDS MySQL 5.7 database to MySQL 8.35, minimizing downtime to just 20 minutes. The process involved using read replicas, taking a backup, restoring it to the new MySQL 8.35 instance, and configuring replication to ensure seamless data transfer with no performance impact.
