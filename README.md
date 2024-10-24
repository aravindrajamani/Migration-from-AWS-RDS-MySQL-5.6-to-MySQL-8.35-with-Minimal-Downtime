# Efficient Migration from AWS RDS MySQL 5.7 to AWS RDS MySQL 8.35(A Online Migration Approach without Production Server Access)

# Overview:
In this migration process, the first step involves creating a read replica from the RDS MySQL 5.7 instance, followed by the creation of a second read replica from the first read replica. Once the second read replica is fully synchronized with the first, the replication between the two replicas is halted. A backup of all databases is then taken from the second read replica, noting the binlog position where replication stopped, and the backup files are modified for compatibility with MySQL 8.35. These modified backup files are restored into the RDS MySQL 8.35 instance. Additionally, replication is configured from the first read replica to the MySQL 8.35 instance, starting from the noted binlog position. This configuration helps avoid performance issues on the RDS MySQL 5.7 production server, ensuring that data synchronization is successfully maintained throughout the migration process.

# Overall Architectural Diagram:

![image](https://github.com/user-attachments/assets/c9f4adf7-e575-46e2-8887-e45984311348)
