# Falcon Eye: OSINT Big Data Project

**Focus:** 10-Year Middle East Conflict Analysis 
**Stack:** Arch Linux (Dev) / Windows 11 (Dev) | Hadoop | HBase | Spring Boot | MySQL

---

## Quick Start (Infrastructure)

Navigate to the `infra/` directory to manage the Big Data cluster.

| Task                     | Command                               |
| :----------------------- | :------------------------------------ |
| **Start Cluster**        | `docker-compose up -d`                |
| **Check Health**         | `docker ps`                           |
| **Stop (Save Data)**     | `docker-compose stop`                 |
| **Shutdown (Keep Data)** | `docker-compose down`                 |
| **Nuclear Reset**        | `docker-compose down -v`              |
| **Live Logs**            | `docker-compose logs -f hbase-master` |

---

## HDFS: The Intelligence Warehouse

Use these to manage the raw 10-year CSV archives.

- **Exit Safe Mode (If HDFS is locked):**
  `docker exec hdfs-namenode hdfs dfsadmin -safemode leave`
- **Create HBase Workspace:**
  `docker exec hdfs-namenode hdfs dfs -mkdir -p /hbase`
- **Upload Conflict CSV:**
  `docker exec hdfs-namenode hdfs dfs -put /local/path/data.csv /hbase/`
- **List Files:**
  `docker exec hdfs-namenode hdfs dfs -ls -R /`

---

## HBase: The Intelligence Ledger

o (Occurrence), d (Data), a (Analysis)

- **Enter HBase Shell:**
  `docker exec -it hbase-master hbase shell`

- **Essential Shell Commands:**
  ```hbase
  status                   # Check cluster health
  list                     # List all tables
  create 'falcon_intel', 'o', 'd', 'a'  # Create table with families (o:occurrence, d:data, a:analysis)
  put 'falcon_intel', 'row1', 'd:headline', 'Satellite anomaly detected'
  scan 'falcon_intel', {LIMIT => 10}
  truncate 'falcon_intel'  # Clear data, keep table
  ```

---

## Spring & Metadata

Managed by the backend lead.

#### MySQL Access

```
docker exec -it falcon-mysql mysql -u root -p (Pass: rootpassword)
```

#### API Heartbeat

```
curl http://localhost:8080/api/status
```

---

## Developer Utilities

#### Resource Monitor

```
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

#### Verification

```
docker exec hbase-master env | grep HBASE
```
