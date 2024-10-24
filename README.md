# Efficient Migration from AWS RDS MySQL 5.7 to AWS RDS MySQL 8.35(A Online Migration Approach without Production Server Access)

# About The Company 
My client is a new-age financial technology company that addresses evolving needs of digital payments infrastructure. Their businesses to embrace digital payments ecosystem using integrated and agile innovations, enabling smooth and flexible payments across all consumer touchpoints- Browser, mobile, instore and remote. Our client provides end-to-end, unified, omnichannel digital payments platform to businesses, banks and networks to address their complex digital payment needs and delivering a simplified, secure, scalable, robust and frictionless payments experience to all stakeholders. 

# The Challenge 
The client was already running web applications on AWS RDS MySQL 5.7 instance and the volume of data was huge, around 1 TB. Based on performance bench-marking, the client was looking to migrate the database to AWS RDS MySQL 8.35 on a serverless architecture. Hence it was decided to migrate the database to AWS RDS with minimal downtime. We had to ensure the end user is least impacted by this activity.

# Exploring possibilities 
Generally, one can use AWS database migration services to migrate data from AWS RDS MySQL 5.7 to AWS RDS MySQL 8.35 but here that can’t be done because AWS migration service does not support online migration from AWS RDS MySQL 5.7 and not a cost-efficient approach. So, a normal migration cannot be performed. Hence, any built-in tool did not support this kind of migration and an online migration had to be performed to seamlessly migrate the huge volume of data with minimal downtime. An offline migration could not be afforded as the required downtime was close to 18+ hours.

# The Solution  
I had a detailed discussion with the customer and carried out the PoC for the same.The output of the PoC was as follows: 

* The data was exported from the source to the destination server using 
the export-import process.

* Data-in Replication process was followed which allows the user to 
synchronize data from any external MySQL server (in our case, the 
server is AWS RDS MySQL 5.7 read replica) into the AWS RDS MySQL 
8.35. The external server can be any of the following: A virtual 
machine, an on-premise server or a database service hosted by any 
other cloud provider.

* The primary usage of this method of replication is when data needs to 
be kept synchronized between an on-premise server and AWS 
Database when one needs to keep multiple cloud databases in sync.
The scenario for this client is a complex cloud solution where data has 
to synchronize between AWS RDS and MySQL 5.7 Read Replica.

* Configuring replication from AWS RDS MySQL 5.7 master database to 
AWS RDS MySQL 8.35 can cause performance issues. The client 
highlighted this, leading us to configure the replication from AWS RDS 
MySQL 5.7 Read Replica to AWS RDS MySQL 8.35.

Once the POC was successful, we had carried out the process in the 
production environment and the required output was achieved. 


# Overview:
In this migration process, the first step involves creating a read replica from the RDS MySQL 5.7 instance, followed by the creation of a second read replica from the first read replica. Once the second read replica is fully synchronized with the first, the replication between the two replicas is halted. A backup of all databases is then taken from the second read replica, noting the binlog position where replication stopped, and the backup files are modified for compatibility with MySQL 8.35. These modified backup files are restored into the RDS MySQL 8.35 instance. Additionally, replication is configured from the first read replica to the MySQL 8.35 instance, starting from the noted binlog position. This configuration helps avoid performance issues on the RDS MySQL 5.7 production server, ensuring that data synchronization is successfully maintained throughout the migration process.

# Overall Architectural Diagram:

![image](https://github.com/user-attachments/assets/c63de0ea-5b92-4b87-b742-fba4c9d8e115)


# Steps to perform the migration :

Following is a detailed explanation of how to go about with the migration process. 

# Create Read replicas:

In the first step, a read replica is created from RDS MySQL 5.7, referred to as read replica1. Subsequently, another replica is created from read replica1, named read replica2. 
Please refer to the diagram below for a visual representation:

![image](https://github.com/user-attachments/assets/bba13845-9ef2-4cb8-874a-19fcc78a8d3c)

# Check Read replica status:

After creating the read replicas, log in to read replica2 and check the replication status. The crucial parameter to verify is "Seconds Behind Master," which should ideally be 0. This indicates that read replica2 is synchronized with read replica1. 
Refer to the snapshot below for a visual representation:

![image](https://github.com/user-attachments/assets/5abae317-ce02-41ec-a6cf-0e8090f1d93d)

# Extend the binlog retention period to maximum:

Logging into read replica1, it's essential to extend the binlog retention period to the maximum value 168 hours. This ensures that the binlog file, which indicates the point where replication stopped between read replica1 and read replica2, does not expire. By extending the binlog retention period, you guarantee that the necessary binlog information remains available for replication purposes, allowing for a seamless continuation of data synchronization between the two replicas.
Refer to the snapshot below for a visual representation:

![image](https://github.com/user-attachments/assets/55489f46-25bb-4ace-a876-6b41e8b8af2a)

# Stop Replication in Read replica2:

To stop the replication in read replica2 and note the binlog file name along with the binlog position where replication stopped, log in to the MySQL server on read replica2. Locate the replication settings or status information, which typically includes details such as the binlog file name (Relay_Master_Log_File) and the binlog position (Exec_Master_Log_Pos) where replication is currently at. These values indicate the point in the replication stream where read replica2 is synchronized with read replica1. Note these values for reference during the migration process.
Refer to the snapshot below for a visual representation:

![image](https://github.com/user-attachments/assets/f7bbccac-3252-43d3-a39b-aa89bbf56410)

# Backup from Read replica2:

To take a backup of all databases, including stored procedures, triggers, and functions, log in to the MySQL server on read replica2. Use mysqldump to create a backup of each database individually, ensuring that stored procedures, triggers, and functions are included in the backup. This ensures that all database objects are captured in the backup files, which can then be used to restore the databases to the new MySQL 8.35 instance.
Refer to the snapshot below for a visual representation:

![image](https://github.com/user-attachments/assets/dae817bc-6519-48bf-a21a-b9ea6c1583d7)

# Considerations for Restoring Backups: Handling 'sql_mode' Errors, Access Denied, and Generated Columns:

When restoring backup files, it's common to encounter errors related to the 'sql_mode' settings, such as 'NO_AUTO_CREATE_USER', and access denied issues. Additionally, if any table contains generated columns, it's necessary to identify these tables and remove the generated columns before restoring the backup. After the restoration is complete, the tables can be altered to add back the generated columns, ensuring the database is fully functional.
Refer to the snapshot below error snapshot:

![image](https://github.com/user-attachments/assets/4b26577a-a998-4532-bcd4-49926316808d)

# Restore backup file in destination:

After modifying the backup files to address errors related to 'sql_mode' and access denied issues, as well as removing any generated columns from tables, the next step is to restore the backup files without encountering any errors. Use a MySQL utility such as mysql to restore the modified backup files to the MySQL 8.35 instance. Ensure that the restoration process completes successfully without any errors, ensuring that the database is fully restored and operational.
Refer to the snapshot below for a visual representation:

![image](https://github.com/user-attachments/assets/3e46b423-6b2a-4eeb-8bc8-22c1c6909ee8)

# Adjusting Column Data Size in Replication: Considerations for slave_type_conversion:

After restoring the database, our client requested changes to the dataset size of certain columns. While increasing the size of columns such as decimal(12,4) to decimal(16,4) does not typically create issues in replication, changing the dataset type can pose challenges. By directly altering the column dataset size, we ensured a smooth transition without impacting replication. However, if there is a need to change the dataset type, it is crucial to adjust the slave_type_conversion parameter. By default, if the dataset of a slave column differs from that of the primary column, replication will fail. The slave_type_conversion parameter allows for the conversion of column types on the slave side, ensuring compatibility and successful replication.

# Type conversion modes (slave_type_conversions variable):  
The setting of the slave_type_conversions global server variable controls the type conversion mode used on the replica. This variable takes a set of values from the following table, which shows the effects of each mode on the replica's type-conversion behavior:

| Mode                     | Effect                                                                                                                                                                                                                                         |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **ALL_LOSSY**             | • In this mode, type conversions that would mean loss of information are permitted.                                                                                                                                                          |
|                          | • This does not imply that non-lossy conversions are permitted, merely that only cases requiring either lossy conversions or no conversion at all are permitted.                                         |
|                          | • For example, enabling only this mode permits an `INT` column to be converted to `TINYINT` (a lossy conversion), but not a `TINYINT` column to an `INT` column (non-lossy).                                                                   |
|                          | • Attempting the latter conversion in this case would cause replication to stop with an error on the replica.                                                                                           |
| **ALL_NON_LOSSY**         | • This mode permits conversions that do not require truncation or other special handling of the source value; it permits conversions where the target type has a wider range than the source type.                                            |
|                          | • Setting this mode has no bearing on whether lossy conversions are permitted.                                                                                                                          |
|                          | • If only `ALL_NON_LOSSY` is set, but not `ALL_LOSSY`, then attempting a conversion that would result in the loss of data (such as `INT` to `TINYINT` or `CHAR(25)` to `VARCHAR(20)`) causes the replica to stop with an error.             |
| **ALL_LOSSY, ALL_NON_LOSSY** | • When this mode is set, all supported type conversions are permitted, whether or not they are lossy conversions.                                                                                                                         |
| **ALL_SIGNED**            | • Treat promoted integer types as signed values (the default behavior).                                                                                                                                                                      |
| **ALL_UNSIGNED**          | • Treat promoted integer types as unsigned values.                                                                                                                                                                                           |
| **ALL_SIGNED, ALL_UNSIGNED** | • Treat promoted integer types as signed if possible, otherwise as unsigned.                                                                                                                                                               |
| **[empty]**               | • When `slave_type_conversions` is not set, no attribute promotion or demotion is permitted; all columns in the source and target tables must be of the same types.                                                                          |
|                          | • This mode is the default.                                                                                                                                                                                                                   |

When an integer type is promoted, its signedness is not preserved. By default, the replica treats all such values as signed. Beginning with MySQL 5.7.2, you can control this behavior using ALL_SIGNED, ALL_UNSIGNED, or both. (Bug#15831300) ALL_SIGNED tells the replica to treat all promoted integer types as signed; ALL_UNSIGNED instructs it to treat these as unsigned. Specifying both causes the replica to treat the value as signed if possible, otherwise to treat it as unsigned; the order in which they are listed is not significant. Neither ALL_SIGNED nor ALL_UNSIGNED has any effect if at least one of ALL_LOSSY or ALL_NONLOSSY is not also used.
Changing the type conversion mode requires restarting the replica with the new slave_type_conversions setting.

**Supported conversions:**  Supported conversions between different but similar data types are shown in the following list:

# Between any of the integer types TINYINT, SMALLINT, MEDIUMINT, INT, and BIGINT:
This includes conversions between the signed and unsigned versions of these types.
Lossy conversions are made by truncating the source value to the maximum (or minimum) permitted by the target column. For ensuring non-lossy conversions when going from unsigned to signed types, the target column must be large enough to accommodate the range of values in the source column. For example, you can demote TINYINT UNSIGNED non-lossily to SMALLINT, but not to TINYINT.

# Between any of the decimal types DECIMAL, FLOAT, DOUBLE, and NUMERIC:
FLOAT to DOUBLE is a non-lossy conversion; DOUBLE to FLOAT can only be handled lossily. A conversion from DECIMAL(M,D) to DECIMAL(M',D') where D' >= D and (M'-D') >= (M-D) is non-lossy; for any case where M' < M, D' < D, or both, only a lossy conversion can be made.
For any of the decimal types, if a value to be stored cannot be fit in the target type, the value is rounded down according to the rounding rules defined for the server elsewhere in the documentation. See Rounding Behavior, for information about how this is done for decimal types.

# Between any of the string types CHAR, VARCHAR, and TEXT, including conversions between different widths:
Conversion of a CHAR, VARCHAR, or TEXT to a CHAR, VARCHAR, or TEXT column the same size or larger is never lossy. Lossy conversion is handled by inserting only the first N characters of the string on the replica, where N is the width of the target column.
Important:
Replication between columns using different character sets is not supported.

# Between any of the binary data types BINARY, VARBINARY, and BLOB, including conversions between different widths:

Conversion of a BINARY, VARBINARY, or BLOB to a BINARY, VARBINARY, or BLOB column the same size or larger is never lossy. Lossy conversion is handled by inserting only the first N bytes of the string on the replica, where N is the width of the target column.

# Between any 2 BIT columns of any 2 sizes:
When inserting a value from a BIT(M) column into a BIT(M') column, where M' > M, the most significant bits of the BIT(M') columns are cleared (set to zero) and the M bits of the BIT(M) value are set as the least significant bits of the BIT(M') column.
When inserting a value from a source BIT(M) column into a target BIT(M') column, where M' < M, the maximum possible value for the BIT(M') column is assigned; in other words, an “all-set” value is assigned to the target column.

# Migrating Users with Grants:

To migrate users from MySQL 5.7 to MySQL 8.35 without dumping and restoring the mysql database, fetch all users from the MySQL 5.7 read replica2 along with their grants. Generate CREATE USER queries for each user with their respective host and grant privileges. Execute these queries on the MySQL 8.35 instance to create the users. Assign passwords uniformly to each user. Finally, modify the CREATE USER queries to set the passwords for each user. Adjust your security policy to require users to change their passwords upon first login for added security.

# Create user in source:

To create a separate user for replication on the production server directly,you would typically access the MySQL database management interface or command line interface. From there, you would navigate to the user management section and add a new user with the necessary privileges for replication. These privileges typically include REPLICATION CLIENT and REPLICATION SLAVE. Once the user is created, you would set a password for the user to ensure secure access. This user will be used specifically for replication purposes, separate from regular database users, to ensure the integrity and security of the replication process.

# Replication from Read replica1 to destination:

This is the last leg of the migration process where the database backup is available in the destination database, and data sync from source to destination can be initiated. While configuring replication, it is crucial to mention the binlog file with the position where it is stopped from read replica 1 to read replica 2. This ensures that all data from read replica 2 is migrated, including legacy data. With the replication configuration using the binlog file where replication stopped, delta data will also migrate to MySQL 8.35, ensuring a complete and seamless migration process.
Refer to the snapshot below for a visual representation:

![image](https://github.com/user-attachments/assets/09006c74-405a-4e80-bc94-dfd12a570508)

 
# Check the status of  Replication:
After configuring replication, it is important to check the replication status to ensure that data is syncing correctly. One key parameter to monitor is "Seconds Behind Master," which should ideally be 0. This indicates that there is no lag between read replica 1 and RDS MySQL 8.35, signifying a successful synchronization. Continuous monitoring of the replication status is essential to ensure that any potential issues are promptly addressed, and data integrity is maintained throughout the migration process.
Refer to the snapshot below for a visual representation:

![image](https://github.com/user-attachments/assets/3965524a-3966-47b2-b540-e6aca9438719)

# Cutover:
Once we receive approval from the client, we can proceed with the cut-over process. This involves stopping the replication between read replica 1 and RDS MySQL 8.35. Afterward, we will designate MySQL 8.35 as the master database. The application team will then need to update the IP address in the application configuration from RDS MySQL 5.7 to RDS MySQL 8.35. This cut-over process will require downtime, but we aim to keep it minimal, completing the migration within just 20 minutes. And this migration completed with 20 minutes of down time.

# Testing the destination database:
The table and row counts need to be monitored in the destination database to ensure replication is functional. An added point to be noted here is to use the count function and not the information schema. Reason being, since this database uses InnoDB Database engine, we would get only approximate row count and not be able to get exact row count using the information schema. 

For Example:
SELECT SUM(TABLE_ROWS)
   FROM INFORMATION_SCHEMA.TABLES
   WHERE TABLE_SCHEMA = 'yourDatabaseName';

Instead of using above query, use below

(Select ‘Table_Name’,count(*) from ‘Table_Name’) union all
(Select ‘Table_Name’,count(*) from ‘Table_Name’) union all
(Select ‘Table_Name’,count(*) from ‘Table_Name’) union all
(Select ‘Table_Name’,count(*) from ‘Table_Name’) union all
                                               .
                                               .
                                               .
(Select ‘Table_Name’,count(*) from ‘Table_Name’);


# Cutover and Delete RDS:

Once there is no replication lag, based on client confirmation, the IP can be cutover from AWS RDS MySQL 5.7 to AWS RDS MySQL 8.35. This is the only time during which downtime is required.
Following continuous monitoring, the source database in AWS RDS can be decommissioned.









