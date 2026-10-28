# MongoDB Backup and Recovery — Complete Notes

Backup and Recovery is an important topic in MongoDB database administration. It ensures that data can be restored if it is accidentally deleted, corrupted, or lost because of hardware failure, software errors, or other disasters.

This topic has three main parts:

1. **Creating backups of replica sets**
2. **Restoring data from backups**
3. **Strategies for disaster recovery**

---

# 1. Creating Backups of Replica Sets

## 1.1 What is a Backup?

A **backup** is a copy of database data stored separately so that it can be recovered if the original data is lost or damaged.

### Why do we need backups?

- Accidental deletion of documents or collections.
- Hardware failure or server crashes.
- Database corruption.
- Cyberattacks or unauthorized changes.
- Recovery after a failed database upgrade.
- Migration of data to another server.

**Example:** Suppose a college stores student records in MongoDB. If the database server fails, a backup allows the college to recover student information.

## 1.2 What is a Replica Set?

A **replica set** is a group of MongoDB servers that maintain copies of the same data.

A typical replica set consists of:

- **Primary:** Receives normal write operations.
- **Secondary:** Replicates data from the primary.
- **Arbiter (optional):** Participates in elections but does not store a data copy.

### How does replication work?

Suppose a student record is inserted into the primary server:

1. The primary stores the student record.
2. The secondary servers replicate the operation.
3. If the primary fails, an eligible secondary can be elected as the new primary.

**Important:** Replication is not a replacement for backups. If someone accidentally deletes data, that deletion can also replicate to the secondary servers.

Therefore, backups are necessary even when replication is enabled.

## 1.3 Methods of Backing Up MongoDB Replica Sets

### Method 1: Using `mongodump`

`mongodump` is a MongoDB Database Tools utility that creates a logical backup of database contents. It exports data and relevant metadata into BSON and other files.

**Basic syntax:**

```bash
mongodump --uri="mongodb://localhost:27017" --out="./backup"
```

This command creates a backup of the databases accessible through the specified MongoDB server.

**Example: Backing up a specific database**

Suppose your database is named `college`.

```bash
mongodump --uri="mongodb://localhost:27017/college" --out="./backup"
```

The backup directory will contain a folder for the `college` database.

Example structure:

```text
backup/
└── college/
    ├── students.bson
    ├── students.metadata.json
    ├── teachers.bson
    └── teachers.metadata.json
```

The files contain the backed-up collection data and associated metadata.

**Advantages:**
- Easy to use.
- Suitable for logical backups.
- Supports backing up individual databases or collections.
- Works well for small and medium-sized databases.

**Limitations:**
- Can take considerable time for large databases.
- A consistent backup of a changing replica set requires appropriate coordination.
- Does not, by itself, provide a complete point-in-time recovery solution.

### Method 2: Backing Up a Replica Set Using `mongodump`

A replica set can be backed up by connecting to its members through an appropriate connection string.

Example:

```bash
mongodump --uri="mongodb://user:password@host1:27017,host2:27017,host3:27017/?replicaSet=rs0" --out="./backup"
```

Here:

- `user:password` represents the database credentials.
- `host1`, `host2`, and `host3` represent replica set members.
- `replicaSet=rs0` specifies the replica set name.
- `--out="./backup"` specifies the backup directory.

**Important:** Use secure credential handling instead of placing real passwords directly in shell commands. Connecting to a replica set does not automatically guarantee that the backup is consistent across all collections while writes are occurring.

### Method 3: Using `mongodump` with `--oplog`

For supported replica set deployments, `--oplog` captures operations from the replica set's oplog during the dump.

Example:

```bash
mongodump \
  --uri="mongodb://localhost:27017/?replicaSet=rs0" \
  --oplog \
  --out="./backup"
```

The oplog records changes made to the database.

This option can help create a point-in-time-consistent logical backup when its requirements are satisfied.

**Important:** `--oplog` has restrictions. For example, it is not supported when dumping only an individual database or collection, and the backup must be restored appropriately to use the captured oplog.

### Method 4: Backing Up from a Secondary

In some deployments, administrators prefer to perform backups using a secondary member to reduce the load on the primary.

However, this approach requires care because:

- A secondary may lag behind the primary.
- Its data may not reflect the most recent writes.
- A consistent backup still needs an appropriate backup procedure.
- Read preferences and permissions must be configured correctly.

For large production deployments, managed backup services or coordinated filesystem snapshots may be more appropriate.

## 1.4 Logical Backup vs Physical Backup

MongoDB backups can be classified into two major categories.

| Feature | Logical Backup | Physical Backup |
|---|---|---|
| Meaning | Exports database contents logically | Copies underlying database files |
| Common method | `mongodump` | Filesystem snapshots or supported storage snapshots |
| Format | BSON and metadata files | Database storage files |
| Portability | Generally easier to move between compatible environments | Depends on storage engine, configuration, and MongoDB version |
| Speed for large databases | May be slower | Can be faster |
| Recovery | Restore using `mongorestore` | Restore files or snapshots using the appropriate procedure |

**Exam point:** `mongodump` is used to create logical backups, whereas physical backups copy the underlying database files.

---

# 2. Restoring Data from Backups

## 2.1 What is Database Recovery?

**Database recovery** is the process of bringing a database back to a usable state after data loss, corruption, or system failure.

MongoDB provides `mongorestore` to restore backups created using `mongodump`.

The general process is:

1. Identify the required backup.
2. Prepare the destination MongoDB deployment.
3. Restore the backup.
4. Verify that the collections and documents are available.
5. Confirm that the application can access the restored data.

## 2.2 Restoring a Complete Backup

Suppose you created a backup using:

```bash
mongodump --uri="mongodb://localhost:27017" --out="./backup"
```

To restore the backup, run:

```bash
mongorestore --uri="mongodb://localhost:27017" ./backup
```

This restores the databases represented in the backup directory, subject to the restore options and destination configuration.

### Restoring a Specific Database

Suppose the backup directory contains a `college` database.

```bash
mongorestore \
  --uri="mongodb://localhost:27017" \
  --nsInclude="college.*" \
  ./backup
```

Here:

- `--nsInclude="college.*"` selects collections belonging to the `college` database.
- `./backup` specifies the directory containing the backup.

A namespace is the combination of a database name and collection name. For example, `college.students` is a namespace.

### Restoring a Single Collection

To restore only the `students` collection:

```bash
mongorestore \
  --uri="mongodb://localhost:27017" \
  --nsInclude="college.students" \
  ./backup
```

This is useful when only one collection has been deleted or damaged.

## 2.3 Overwriting Existing Data During Recovery

By default, `mongorestore` does not drop existing collections before restoring them.

If you want to replace existing collections with the versions in the backup, you can use `--drop`.

Example:

```bash
mongorestore \
  --uri="mongodb://localhost:27017" \
  --drop \
  ./backup
```

The `--drop` option drops each collection being restored before restoring its backed-up version.

**Warning:** This can destroy newer data in the destination. Use it only when you intend to replace the affected collections.

### Important Restore Options

| Option | Purpose |
|---|---|
| `--drop` | Drops each target collection before restoring it. |
| `--nsInclude` | Selects the namespaces to restore. |
| `--nsFrom` | Specifies the source namespace pattern for namespace mapping. |
| `--nsTo` | Specifies the destination namespace pattern for namespace mapping. |
| `--gzip` | Indicates that the dump files are compressed using gzip. |
| `--oplogReplay` | Replays oplog entries included in a compatible dump. |

## 2.4 Restoring a Backup Created with `--oplog`

If a backup was created with `mongodump --oplog`, its captured operations can be replayed during restoration.

Example:

```bash
mongorestore \
  --uri="mongodb://localhost:27017" \
  --oplogReplay \
  ./backup
```

The `--oplogReplay` option applies the operations captured in the dump's oplog file.

This can help restore a replica set backup to the point represented by the backup procedure.

**Important:** Oplog replay has specific requirements. The backup must include the necessary oplog data, and the destination and restore options must be compatible with the backup. Do not assume that any ordinary dump can support point-in-time recovery.

## 2.5 Verifying Restored Data

After restoration, connect to MongoDB using the shell:

```bash
mongosh
```

Select the database:

```javascript
use college
```

List collections:

```javascript
show collections
```

Check the number of student records:

```javascript
db.students.countDocuments({})
```

Display some student records:

```javascript
db.students.find().limit(5)
```

If the collections and expected records are present, the basic restoration has succeeded.

For production systems, verification should also include indexes, users and permissions, application connectivity, data consistency, and other necessary configuration.

### Example Scenario

Suppose the `students` collection originally contained 1,000 documents. An accidental operation deletes 200 documents.

If a suitable backup contains the original 1,000 documents, you can restore that backup.

However, restoring the entire collection may overwrite legitimate changes made after the backup.

A safer approach may be to restore the backup to a separate database, identify the missing documents, and recover only the affected records.

---

# 3. Disaster Recovery Strategies

## 3.1 What Is Disaster Recovery?

**Disaster recovery (DR)** is a planned process for restoring database services and data after a major failure.

Disasters can include:

- Complete server failure.
- Data-center outages.
- Accidental deletion of important databases.
- Database corruption.
- Ransomware or unauthorized changes.
- Natural disasters.
- Cloud infrastructure failures.

A disaster recovery plan defines how an organization will restore its data and resume operations.

## 3.2 Important Disaster Recovery Strategies

### Strategy 1: Regular Automated Backups

Schedule backups at regular intervals instead of relying on manual backups.

For example:

- Full logical backup every day.
- More frequent backups of critical data or changes.
- Periodic verification of backup integrity.

The actual frequency should depend on how much data the organization can afford to lose.

**Advantage:** Reduces the risk of permanent data loss.

**Limitation:** Data created after the latest recoverable backup may be lost.

### Strategy 2: Off-Site Backups

Store backups in a separate location or an independent storage system.

For example, a company might store database backups in cloud object storage separate from the database server.

This protects against failures affecting the original server or location.

**Advantage:** Protects backups against local hardware failures and site-level disasters.

**Limitation:** Restoring large backups can take time, depending on network bandwidth and backup size.

### Strategy 3: Replica Sets and Redundancy

Use replica sets to maintain multiple copies of data on separate MongoDB members.

If the primary server fails, an eligible secondary may be elected as the new primary.

**Advantage:** Improves availability and reduces downtime during certain failures.

**Limitation:** Replication can also reproduce accidental changes and deletions. It is not a substitute for independent backups.

### Strategy 4: Point-in-Time Recovery (PITR)

Point-in-time recovery allows a database to be recovered to a particular time within the available recovery window.

For example, suppose a user accidentally deletes important records at 3:00 PM.

If you have a suitable backup from midnight and sufficient continuous operation-log history, you may be able to restore the backup and replay operations up to just before 3:00 PM.

This can minimize data loss compared with restoring only the midnight backup.

**Advantage:** Helps recover data close to the moment of failure.

**Limitation:** Requires an appropriate backup and operation-log retention strategy. Not every MongoDB backup setup supports PITR.

### Strategy 5: Automated Failover

MongoDB replica sets support elections that can promote an eligible secondary to primary when the existing primary becomes unavailable.

This is an availability mechanism rather than a complete backup-and-recovery solution.

**Advantage:** Reduces downtime after certain server failures.

**Limitation:** Does not protect against every form of data corruption, accidental deletion, or site-wide failure.

### Strategy 6: Regular Disaster Recovery Testing

A backup is useful only if it can be restored successfully.

Organizations should periodically test recovery procedures by restoring backups in a separate environment.

Testing should verify:

- Backup integrity.
- Successful database restoration.
- Data consistency.
- Application functionality.
- Recovery time.
- Whether the recovery objectives are met.

**Advantage:** Helps identify backup failures and recovery problems before an actual disaster occurs.

## 3.3 RPO and RTO

Two important concepts in disaster recovery are **RPO** and **RTO**.

### Recovery Point Objective (RPO)

RPO defines the maximum amount of data loss, measured in time, that an organization can tolerate.

For example, if the RPO is 15 minutes, the organization aims to recover data with no more than 15 minutes of data loss.

### Recovery Time Objective (RTO)

RTO defines the maximum acceptable time required to restore a service after a disruption.

For example, if the RTO is 1 hour, the organization aims to restore the service within one hour.

### Difference Between RPO and RTO

| Feature | RPO | RTO |
|---|---|---|
| Full form | Recovery Point Objective | Recovery Time Objective |
| Focus | Data loss | Service downtime |
| Measures | Acceptable time gap in recoverable data | Acceptable time to restore service |
| Example | Maximum 15 minutes of data loss | Service restored within 1 hour |

**Easy way to remember:**

- **RPO = How much data can we afford to lose?**
- **RTO = How long can we afford to be down?**

---

# 4. Practical Example: MongoDB Backup and Recovery

Suppose you have a MongoDB database called `college` containing a collection named `students`.

### Step 1: Create a backup

Run this command in your terminal:

```bash
mongodump \
  --uri="mongodb://localhost:27017" \
  --db=college \
  --out="./backup"
```

### Step 2: Verify the backup files

The backup directory should contain files similar to:

```text
backup/
└── college/
    ├── students.bson
    └── students.metadata.json
```

Other collections, if present, will have their own files.

### Step 3: Simulate data loss

For a practice environment only, you can remove the collection:

```javascript
use college
db.students.drop()
```

**Warning:** This permanently removes the collection from the active database unless you can recover it. Do not run this command on a real database.

### Step 4: Restore the backup

Run the following in your terminal:

```bash
mongorestore \
  --uri="mongodb://localhost:27017" \
  --nsInclude="college.*" \
  ./backup
```

### Step 5: Verify the restored collection

Open MongoDB Shell:

```javascript
use college
show collections
db.students.find()
```

The restored collection should now be available.

---

# 5. Quick Revision Notes for Exams

| Topic | Key Point |
|---|---|
| Backup | A copy of database data used for recovery. |
| Replica set | Multiple MongoDB members maintain copies of data. |
| Primary | Normally accepts write operations. |
| Secondary | Replicates data from the primary. |
| `mongodump` | Creates a logical backup. |
| `mongorestore` | Restores a logical backup. |
| `--oplog` | Captures oplog operations during a supported dump. |
| `--oplogReplay` | Replays oplog operations during a compatible restore. |
| `--drop` | Drops each target collection before restoring it. |
| Disaster recovery | A plan to restore data and services after a major failure. |
| Off-site backup | Keeps backups separate from the original system or location. |
| PITR | Recovers data to a specific point in time within the available recovery window. |
| RPO | Maximum tolerable data loss measured in time. |
| RTO | Maximum acceptable recovery time. |
| Recovery testing | Confirms that backups can actually be restored. |

## Important Questions for Exams

### Short-answer questions

1. What is a database backup?
2. What is the purpose of `mongodump`?
3. What is the difference between `mongodump` and `mongorestore`?
4. Why is replication not a substitute for backups?
5. What are RPO and RTO?

### Long-answer questions

1. Explain the procedure for creating and restoring a MongoDB backup with suitable commands.
2. Explain different disaster recovery strategies in MongoDB.
3. Differentiate logical backups and physical backups.
4. Explain point-in-time recovery and its importance.
5. Discuss the role of replica sets in database availability and recovery.

---

## Final Takeaway

Remember these three concepts:

- **Backup:** Save a recoverable copy of your data.
- **Recovery:** Restore data after it is lost or damaged.
- **Disaster recovery:** Plan and prepare to restore both data and database services after a major failure.
