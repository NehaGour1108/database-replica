Setting up a MySQL read replica locally on a single laptop involves configuring a primary MySQL instance as the "master" and another instance as the "replica" to replicate data. Here’s a step-by-step guide on how to set this up locally, assuming MySQL is installed on your laptop.

### 1. Install MySQL and Prepare Configuration
If not already installed, download and install MySQL from the [official MySQL website](https://dev.mysql.com/downloads/mysql/) or using Homebrew:
```bash
brew install mysql
```

### 2. Start MySQL and Create Configuration Files
Run two MySQL instances: one will serve as the master, and the other as the replica. Use unique ports and server IDs to distinguish between them.

#### Configure Master
1. Locate or create the MySQL configuration file for the master. Typically, this file is at `/usr/local/etc/my.cnf` for a Homebrew installation.
2. Edit the master’s configuration to include replication settings:
   ```ini
   [mysqld]
   server-id=1
   log-bin=mysql-bin
   bind-address=127.0.0.1
   port=3306
   ```
3. Restart the MySQL server to apply these settings:
   ```bash
   brew services restart mysql
   ```

#### Configure Replica
1. Create a separate configuration file for the replica, such as `/usr/local/etc/my-replica.cnf`.
2. In the replica configuration file, set the server ID to a unique value and specify a different port:
   ```ini
   [mysqld]
   server-id=2
   relay-log=mysql-relay-bin
   bind-address=127.0.0.1
   port=3307
   ```
3. Start the replica instance with this configuration:
   ```bash
   mysqld --defaults-file=/usr/local/etc/my-replica.cnf &
   ```

### 3. Create a Replication User on the Master
Log into the master MySQL instance and create a user specifically for replication:
```sql
CREATE USER 'replication_user'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%';
FLUSH PRIVILEGES;
```

### 4. Get the Master’s Status
To configure the replica, you need the master’s binary log file name and position. Run the following on the master:
```sql
SHOW MASTER STATUS;
```
Take note of the `File` and `Position` values in the output; you’ll need these for the replica setup.

### 5. Configure the Replica
Log into the replica instance on port `3307` and configure it to replicate from the master:
```sql
CHANGE MASTER TO
  MASTER_HOST='127.0.0.1',
  MASTER_PORT=3306,
  MASTER_USER='replication_user',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',  -- Use the actual file name from the master
  MASTER_LOG_POS=4;                    -- Use the actual position from the master

START SLAVE;
```

### 6. Verify Replication
On the replica, check the replication status:
```sql
SHOW SLAVE STATUS\G
```
Ensure that `Slave_IO_Running` and `Slave_SQL_Running` are both `Yes`. This indicates that the replica is successfully replicating data from the master.

### 7. Test the Setup
To test, create a sample database and table on the master and insert data. Then, check the replica to confirm that the data has been replicated.

#### On the Master:
```sql
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));
INSERT INTO users (id, name) VALUES (1, 'Alice');
```

#### On the Replica:
Confirm that the data appears on the replica:
```sql
USE test_db;
SELECT * FROM users;
```

### Summary
- **Master** is configured with binary logging and has a replication user.
- **Replica** connects to the master using `CHANGE MASTER TO` command and replicates data changes.

This setup allows you to simulate a MySQL master-replica environment on your local machine.
