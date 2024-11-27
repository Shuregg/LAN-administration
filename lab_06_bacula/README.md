# lab_06_bacula

## 0. Prerequirements

### 0.1 Server side

```bash
sudo apt install bacula
sudo apt install bacula-console-qt
```

### 0.2 Client side

```bash
sudo apt install bacula-fd
```

## 1. PostgreSQL configuration

My *cluster number* is equal to *11*

```bash
sudo nano /etc/postgresql/${your_cluster_number}/main/postgresql.conf
```

Set all addresses to be listened

```bash
listen_addresses='*'
```

```bash
sudo nano /etc/postgresql/${your_cluster_number}/main/pg_hba.conf
```

```bash
local all postgres trust
local all all trust
host all all 127.0.0.1/32 trust
host all all 192.168.122.13/24 trust
```

Restart cluster

```bash
sudo pg_ctlcluster ${your_cluster_number} main restart
psql template1 -U postgres -h 192.168.122.13 -p 5432
```


<!-- sudo -i -u postgres -->

<!-- psql -->

```sql
CREATE ROLE bacula; -- if not exists

-- DO $$
-- BEGIN
--    IF NOT EXISTS (SELECT FROM pg_catalog.pg_roles WHERE rolname = 'bacula') THEN
--       CREATE ROLE bacula;
--    END IF;
-- END$$;

ALTER USER bacula PASSWORD 'bacula';
ALTER USER bacula LOGIN SUPERUSER CREATEDB CREATEROLE;

\q
```

```bash
psql postgres -p 5432 -U postgres
```

```sql
CREATE DATABASE bacula WITH ENCODING = 'SQL_ASCII' LC_COLLATE = 'C' LC_CTYPE = 'C' TEMPLATE = 'template0'; -- if not exists

ALTER DATABASE bacula OWNER TO bacula;
```

```bash
sudo usermod -a -G shadow postgres
sudo setfacl -d -m u:postgres:r /etc/parsec/macdb
sudo setfacl -R -m u:postgres:r /etc/parsec/macdb
sudo setfacl -m u:postgres:rx /etc/parsec/macdb
sudo setfacl -d -m u:postgres:r /etc/parsec/capdb
sudo setfacl -R -m u:postgres:r /etc/parsec/capdb
sudo setfacl -m u:postgres:rx /etc/parsec/capdb
```

```bash
sudo pdpl-user bacula -l 0:0
```

```bash
sudo nano /usr/share/bacula-director/make_postgresql_tables
```

```bash
db_name=${db_name:-bacula}
psql -U bacula -h 192.168.122.13 -p 5432 -f - -d ${db_name} $* <<END-OF-DATA
```

Entire [make_postgresql_tables](scripts/grant_postgresql_privileges) bash script

```bash
sudo nano /usr/share/bacula-director/grant_postgresql_privileges
```

```bash
db_user=${db_user:-bacula}
bindir=/usr/bin
db_name=${db_name:-bacula}
db_password=${db_password:-bacula}
if [ "$db_password" != "" ]; then
   pass="password '$db_password'"
fi


$bindir/psql -U bacula -h 192.168.122.13 -p 5432 -f - -d ${db_name} $* <<END-OF-DATA
```

Entire [make_postgresql_tables](scripts/make_postgresql_tables) bash script

Run this scripts to fill database and set grants

```bash
sudo /usr/share/bacula-director/make_postgresql_tables
sudo /usr/share/bacula-director/grant_postgresql_privileges
```

## 2. Bacula-director configuration

```bash
sudo nano /etc/bacula/bacula-dir.conf
```

Entire [bacula-dir.conf](configs/bacula-dir.conf) configuration file

```bash
sudo mkdir /etc/bacula/job.d/
sudo nano /etc/bacula/job.d/backup-dir-fd.conf
```

Entire [backup-dir-fd.conf](configs/backup-dir-fd.conf) configuration file

```bash
sudo nano /etc/bacula/job.d/restore-dir-fd.conf
```

Entire [restore-dir-fd.conf](configs/restore-dir-fd.conf) configuration file

## 3. Storage deamon configuration

```bash
sudo chmod 644 /etc/bacula/bacula-sd.conf
sudo chown root:bacula /etc/bacula/bacula-sd.conf
sudo systemctl restart bacula-sd.service
```

Entire [bacula-sd.conf](configs/bacula-sd.conf) configuration file

```bash
sudo mkdir -p /backups/files1/
sudo chmod 755 /backups/files1/
sudo chown bacula:bacula /backups/files1/
```

Check syntax correctness

```bash
sudo /usr/sbin/bacula-dir -t -c /etc/bacula/bacula-dir.conf
```

Restart director and storage deamons

```bash
sudo systemctl restart bacula-dir
sudo systemctl restart bacula-sd
```

## 4. File deamon configuration (client)

Install the bacula file deamon on the client machine

```bash
sudo apt install bacula-fd
```

```bash
sudo nano /etc/bacula/bacula-fd.conf
```

Entire [bacula-fd.conf](configs/bacula-fd.conf) configuration file

Set access rules

```bash
sudo chmod 644 /etc/bacula/bacula-fd.conf
sudo chown root:bacula /etc/bacula/bacula-fd.conf
```

Check syntax correctness

```bash
sudo /usr/sbin/bacula-fd -t -c /etc/bacula/bacula-fd.conf
```

Restart Bacula file deamon

```bash
sudo systemctl restart bacula-fd.service
```

## 5. Bacula connection configuration (server)

### 5.1 Set up console application

```bash
sudo nano /etc/bacula/bconsole.conf
```

[bconsole.conf](configs/bconsole.conf)

```bash
Director {
  Name = dir-dir
  DIRport = 9101
  # IP-адрес Директора
  address = 192.168.122.13
  Password = "dirpass"
}
```

### 5.2 Install GUI application

```bash
sudo apt install bacula-console-qt
```

## 6. Creating a backup

### 6.1 via CLI

Run Bacula console

```bash
sudo bconsole
```

#### 1. Run backup job (full backup)

<details><summary>Bacula console commands and messages</summary>

```bash
astra@server:~$ sudo bconsole 
Connecting to Director 192.168.122.13:9101
1000 OK: 103 bacula-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*run
Automatically selected Catalog: BaculaCatalog
Using Catalog "BaculaCatalog"
A job name must be specified.
The defined Job resources are:
     1: BackupClient1
     2: RestoreFiles
Select Job resource (1-2): 1
Run Backup job
JobName:  BackupClient1
Level:    Full
Client:   dir-fd
FileSet:  Full Set
Pool:     File (From Job resource)
Storage:  stor-sd (From Job resource)
When:     2024-11-27 19:52:53
Priority: 10
OK to run? (yes/mod/no): yes
Job queued. JobId=7
You have messages.
*messages 
27-ноя 19:48 stor-sd JobId 6: Elapsed time=00:00:20, Transfer rate=36.43 M Bytes/second
27-ноя 19:48 stor-sd JobId 6: Sending spooled attrs to the Director. Despooling 1,628,334 bytes ...
27-ноя 19:48 bacula-dir JobId 6: Bacula bacula-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu AstraLinux 1.7_x86-64
  JobId:                  6
  Job:                    BackupClient1.2024-11-27_19.47.55_03
  Backup Level:           Full
  Client:                 "dir-fd" 9.6.7 (10Dec20) x86_64-pc-linux-gnu,AstraLinux,1.7_x86-64
  FileSet:                "Full Set" 2024-11-24 15:38:31
  Pool:                   "File" (From Job resource)
  Catalog:                "BaculaCatalog" (From Client resource)
  Storage:                "stor-sd" (From Job resource)
  Scheduled time:         27-ноя-2024 19:47:52
  Start time:             27-ноя-2024 19:47:57
  End time:               27-ноя-2024 19:48:17
  Elapsed time:           20 secs
  Priority:               10
  FD Files Written:       5,765
  SD Files Written:       5,765
  FD Bytes Written:       727,577,806 (727.5 MB)
  SD Bytes Written:       728,636,818 (728.6 MB)
  Rate:                   36378.9 KB/s
  Software Compression:   11.5% 1.1:1
  Comm Line Compression:  None
  Snapshot/VSS:           no
  Encryption:             no
  Accurate:               no
  Volume name(s):         Vol-0001
  Volume Session Id:      1
  Volume Session Time:    1732726041
  Last Volume Bytes:      1,458,739,958 (1.458 GB)
  Non-fatal FD errors:    0
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK

27-ноя 19:48 bacula-dir JobId 6: Begin pruning Jobs older than 6 months .
27-ноя 19:48 bacula-dir JobId 6: No Jobs found to prune.
27-ноя 19:48 bacula-dir JobId 6: Begin pruning Files.
27-ноя 19:48 bacula-dir JobId 6: No Files found to prune.
27-ноя 19:48 bacula-dir JobId 6: End auto prune.

27-ноя 19:52 bacula-dir JobId 7: Start Backup JobId 7, Job=BackupClient1.2024-11-27_19.52.55_05
27-ноя 19:52 bacula-dir JobId 7: Using Device "DevStorage" to write.
27-ноя 19:52 stor-sd JobId 7: Volume "Vol-0001" previously written, moving to end of data.
27-ноя 19:52 stor-sd JobId 7: Ready to append to end of Volume "Vol-0001" size=1,458,739,958
```

</details>

#### 2. Run restore job

<details><summary>Bacula console commands and messages</summary>

```bash
*restore

First you select one or more JobIds that contain files
to be restored. You will be presented several methods
of specifying the JobIds. Then you will be allowed to
select which files from those JobIds are to be restored.

To select the JobIds, you have the following choices:
     1: List last 20 Jobs run
     2: List Jobs where a given File is saved
     3: Enter list of comma separated JobIds to select
     4: Enter SQL list command
     5: Select the most recent backup for a client
     6: Select backup for a client before a specified time
     7: Enter a list of files to restore
     8: Enter a list of files to restore before a specified time
     9: Find the JobIds of the most recent backup for a client
    10: Find the JobIds for a backup for a client before a specified time
    11: Enter a list of directories to restore for found JobIds
    12: Select full restore to a specified Job date
    13: Cancel
Select item:  (1-13): 5
Automatically selected Client: dir-fd
Automatically selected FileSet: Full Set
+-------+-------+----------+-------------+---------------------+------------+
| jobid | level | jobfiles | jobbytes    | starttime           | volumename |
+-------+-------+----------+-------------+---------------------+------------+
|     1 | F     |    5,593 | 719,265,728 | 2024-11-27 21:18:21 | Vol-0001   |
+-------+-------+----------+-------------+---------------------+------------+
You have selected the following JobId: 1

Building directory tree for JobId(s) 1 ...  ++++++++++++++++++++++++++++++++++++++++++
4,714 files inserted into the tree.

You are now entering file selection mode where you add (mark) and
remove (unmark) files to be restored. No files are initially added, unless
you used the "all" keyword on the command line.
Enter "done" to leave this mode.

cwd is: /
$ ls
home/
$ mark home/ 
5,593 files marked.
$ done
Bootstrap records written to /var/lib/bacula/bacula-dir.restore.1.bsr

The Job will require the following (*=>InChanger):
   Volume(s)                 Storage(s)                SD Device(s)
===========================================================================
   
    Vol-0001                  stor-sd                   DevStorage               

Volumes marked with "*" are in the Autochanger.


5,593 files selected to be restored.

Run Restore job
JobName:         RestoreFiles
Bootstrap:       /var/lib/bacula/bacula-dir.restore.1.bsr
Where:           /home2
Replace:         Always
FileSet:         Full Set
Backup Client:   dir-fd
Restore Client:  dir-fd
Storage:         stor-sd
When:            2024-11-27 21:22:41
Catalog:         BaculaCatalog
Priority:        10
Plugin Options:  *None*
OK to run? (yes/mod/no): yes
Job queued. JobId=3
*
You have messages.
*messages 
27-ноя 21:22 bacula-dir JobId 3: Start Restore Job RestoreFiles.2024-11-27_21.22.48_08
27-ноя 21:22 bacula-dir JobId 3: Restoring files from JobId(s) 1
27-ноя 21:22 bacula-dir JobId 3: Using Device "DevStorage" to read.
27-ноя 21:22 stor-sd JobId 3: Ready to read from volume "Vol-0001" on File device "DevStorage" (/backups/files1).
27-ноя 21:22 stor-sd JobId 3: Forward spacing Volume "Vol-0001" to addr=214
27-ноя 21:22 stor-sd JobId 3: End of Volume "Vol-0001" at addr=721016919 on device "DevStorage" (/backups/files1).
27-ноя 21:22 stor-sd JobId 3: Elapsed time=00:00:03, Transfer rate=240.0 M Bytes/second
27-ноя 21:22 dir-fd JobId 3: Warning: bxattr_linux.c:280 setxattr error on file "/home2/home/.pdp/": ERR=Отказано в доступе
27-ноя 21:22 dir-fd JobId 3: Warning: bxattr_linux.c:280 setxattr error on file "/home2/home/": ERR=Отказано в доступе
27-ноя 21:22 bacula-dir JobId 3: Bacula bacula-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu AstraLinux 1.7_x86-64
  JobId:                  3
  Job:                    RestoreFiles.2024-11-27_21.22.48_08
  Restore Client:         dir-fd
  Where:                  /home2
  Replace:                Always
  Start time:             27-ноя-2024 21:22:50
  End time:               27-ноя-2024 21:22:53
  Elapsed time:           3 secs
  Files Expected:         5,593
  Files Restored:         5,593
  Bytes Restored:         795,060,342 (795.0 MB)
  Rate:                   265020.1 KB/s
  FD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Restore OK

27-ноя 21:22 bacula-dir JobId 3: Begin pruning Jobs older than 6 months .
27-ноя 21:22 bacula-dir JobId 3: No Jobs found to prune.
27-ноя 21:22 bacula-dir JobId 3: Begin pruning Files.
27-ноя 21:22 bacula-dir JobId 3: No Files found to prune.
27-ноя 21:22 bacula-dir JobId 3: End auto prune.
```

</details>

#### 3. Make differential backup

Create several files on client

```bash
touch new_file.txt
touch new_file2.txt
touch new_file3.txt
```

Run backup job on server

<details><summary>Bacula console commands and messages</summary>

```bash
astra@server:~$ sudo bconsole 
Connecting to Director 192.168.122.13:9101
1000 OK: 103 bacula-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*run
Automatically selected Catalog: BaculaCatalog
Using Catalog "BaculaCatalog"
A job name must be specified.
The defined Job resources are:
     1: BackupClient1
     2: RestoreFiles
Select Job resource (1-2): 1
Run Backup job
JobName:  BackupClient1
Level:    Full
Client:   dir-fd
FileSet:  Full Set
Pool:     File (From Job resource)
Storage:  stor-sd (From Job resource)
When:     2024-11-27 21:54:12
Priority: 10
OK to run? (yes/mod/no): mod
Parameters to modify:
     1: Level
     2: Storage
     3: Job
     4: FileSet
     5: Client
     6: When
     7: Priority
     8: Pool
     9: Plugin Options
Select parameter to modify (1-9): 1
Levels:
     1: Full
     2: Incremental
     3: Differential
     4: Since
     5: VirtualFull
Select level (1-5): 3
Run Backup job
JobName:  BackupClient1
Level:    Differential
Client:   dir-fd
FileSet:  Full Set
Pool:     File (From Job resource)
Storage:  stor-sd (From Job resource)
When:     2024-11-27 21:54:12
Priority: 10
OK to run? (yes/mod/no): yes
Job queued. JobId=4
*
You have messages.
*messages 
27-ноя 21:54 bacula-dir JobId 4: Start Backup JobId 4, Job=BackupClient1.2024-11-27_21.54.28_10
27-ноя 21:54 bacula-dir JobId 4: Using Device "DevStorage" to write.
27-ноя 21:54 stor-sd JobId 4: Volume "Vol-0001" previously written, moving to end of data.
27-ноя 21:54 stor-sd JobId 4: Ready to append to end of Volume "Vol-0001" size=721,016,919
27-ноя 21:54 stor-sd JobId 4: Elapsed time=00:00:01, Transfer rate=9.359 K Bytes/second
27-ноя 21:54 stor-sd JobId 4: Sending spooled attrs to the Director. Despooling 1,519 bytes ...
27-ноя 21:54 bacula-dir JobId 4: Bacula bacula-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu AstraLinux 1.7_x86-64
  JobId:                  4
  Job:                    BackupClient1.2024-11-27_21.54.28_10
  Backup Level:           Differential, since=2024-11-27 21:18:21
  Client:                 "dir-fd" 9.6.7 (10Dec20) x86_64-pc-linux-gnu,AstraLinux,1.7_x86-64
  FileSet:                "Full Set" 2024-11-27 21:18:19
  Pool:                   "File" (From Job resource)
  Catalog:                "BaculaCatalog" (From Client resource)
  Storage:                "stor-sd" (From Job resource)
  Scheduled time:         27-ноя-2024 21:54:12
  Start time:             27-ноя-2024 21:54:30
  End time:               27-ноя-2024 21:54:30
  Elapsed time:           1 sec
  Priority:               10
  FD Files Written:       7
  SD Files Written:       7
  FD Bytes Written:       8,542 (8.542 KB)
  SD Bytes Written:       9,359 (9.359 KB)
  Rate:                   8.5 KB/s
  Software Compression:   84.7% 6.5:1
  Comm Line Compression:  None
  Snapshot/VSS:           no
  Encryption:             no
  Accurate:               no
  Volume name(s):         Vol-0001
  Volume Session Id:      3
  Volume Session Time:    1732730935
  Last Volume Bytes:      721,026,864 (721.0 MB)
  Non-fatal FD errors:    0
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK

27-ноя 21:54 bacula-dir JobId 4: Begin pruning Jobs older than 6 months .
27-ноя 21:54 bacula-dir JobId 4: No Jobs found to prune.
27-ноя 21:54 bacula-dir JobId 4: Begin pruning Files.
27-ноя 21:54 bacula-dir JobId 4: No Files found to prune.
27-ноя 21:54 bacula-dir JobId 4: End auto prune.
```
</details>

#### 4. Make incremental backup

Edit  some of the created files; Delete some of them

```bash
rm -rf new_file.txt
echo "hello" >> new_file2.txt
echo "world" >> new_file3.txt
```

Run incremental backup

<details><summary>Bacula console commands and messages</summary>

```bash
*run
A job name must be specified.
The defined Job resources are:
     1: BackupClient1
     2: RestoreFiles
Select Job resource (1-2): 1
Run Backup job
JobName:  BackupClient1
Level:    Full
Client:   dir-fd
FileSet:  Full Set
Pool:     File (From Job resource)
Storage:  stor-sd (From Job resource)
When:     2024-11-27 22:06:28
Priority: 10
OK to run? (yes/mod/no): mod
Parameters to modify:
     1: Level
     2: Storage
     3: Job
     4: FileSet
     5: Client
     6: When
     7: Priority
     8: Pool
     9: Plugin Options
Select parameter to modify (1-9): 1
Levels:
     1: Full
     2: Incremental
     3: Differential
     4: Since
     5: VirtualFull
Select level (1-5): 2
Run Backup job
JobName:  BackupClient1
Level:    Incremental
Client:   dir-fd
FileSet:  Full Set
Pool:     File (From Job resource)
Storage:  stor-sd (From Job resource)
When:     2024-11-27 22:06:28
Priority: 10
OK to run? (yes/mod/no): yes
Job queued. JobId=5
*messages 
27-ноя 22:10 bacula-dir JobId 5: Start Backup JobId 5, Job=BackupClient1.2024-11-27_22.10.20_11
27-ноя 22:10 bacula-dir JobId 5: Using Device "DevStorage" to write.
27-ноя 22:10 stor-sd JobId 5: Volume "Vol-0001" previously written, moving to end of data.
27-ноя 22:10 stor-sd JobId 5: Ready to append to end of Volume "Vol-0001" size=721,026,864
27-ноя 22:10 stor-sd JobId 5: Elapsed time=00:00:01, Transfer rate=322  Bytes/second
27-ноя 22:10 stor-sd JobId 5: Sending spooled attrs to the Director. Despooling 564 bytes ...
27-ноя 22:10 bacula-dir JobId 5: Bacula bacula-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu AstraLinux 1.7_x86-64
  JobId:                  5
  Job:                    BackupClient1.2024-11-27_22.10.20_11
  Backup Level:           Incremental, since=2024-11-27 21:54:30
  Client:                 "dir-fd" 9.6.7 (10Dec20) x86_64-pc-linux-gnu,AstraLinux,1.7_x86-64
  FileSet:                "Full Set" 2024-11-27 21:18:19
  Pool:                   "File" (From Job resource)
  Catalog:                "BaculaCatalog" (From Client resource)
  Storage:                "stor-sd" (From Job resource)
  Scheduled time:         27-ноя-2024 22:06:28
  Start time:             27-ноя-2024 22:10:22
  End time:               27-ноя-2024 22:10:22
  Elapsed time:           1 sec
  Priority:               10
  FD Files Written:       3
  SD Files Written:       3
  FD Bytes Written:       28 (28 B)
  SD Bytes Written:       322 (322 B)
  Rate:                   0.0 KB/s
  Software Compression:   None
  Comm Line Compression:  1.7% 1.0:1
  Snapshot/VSS:           no
  Encryption:             no
  Accurate:               no
  Volume name(s):         Vol-0001
  Volume Session Id:      4
  Volume Session Time:    1732730935
  Last Volume Bytes:      721,027,664 (721.0 MB)
  Non-fatal FD errors:    0
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK

27-ноя 22:10 bacula-dir JobId 5: Begin pruning Jobs older than 6 months .
27-ноя 22:10 bacula-dir JobId 5: No Jobs found to prune.
27-ноя 22:10 bacula-dir JobId 5: Begin pruning Files.
27-ноя 22:10 bacula-dir JobId 5: No Files found to prune.
27-ноя 22:10 bacula-dir JobId 5: End auto prune.

```

</details>

### 6.2 via GUI app

Run Bacula Admin Tool (BAT)

```bash
sudo bat # or sudo /etc/sbin/bat
```
