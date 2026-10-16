# MongoDB Learning Plan — DBA Track (L1 → L2 → L3)

> **Audience:** Database Administrator working at **L2 proficiency** (can handle routine operational tasks, run backups/restores, troubleshoot common incidents with guidance) who wants to reach **L3 / senior DBA** level.
> **Companion resource:** `mongodb-cheatsheet.html` (commands reference)
> **Video course:** [YouTube playlist](https://www.youtube.com/watch?v=cLsawKBUdTE&list=PLSmSa8KVdfSu-XFvjdWoN7z9WRoLly4my)

---

## 1. How to Use This Plan

- **Duration:** 12 weeks (~10–12 hrs/week). Compress to 6 weeks at full-time pace.
- **Method per module:** Read docs → Watch video → Lab on local replica set → Document in runbook → Quiz yourself.
- **Golden rule:** *Never learn a command without running it against a real replica set.* Single-node mongod hides 80% of production issues.
- **Deliverable per module:** a runbook entry in `runbooks/` you could hand to another L2.

### Skill-Level Definitions (self-assessment)

| Level | You can... |
|-------|-----------|
| **L1** | Install MongoDB, run basic CRUD, read logs |
| **L2 (you)** | Manage users/roles, backups & restores, replica set basics, index creation, monitor with basic tools, resolve known incidents via runbooks |
| **L3 (target)** | Design sharding topologies, diagnose performance regressions from first principles, plan capacity, do zero-downtime upgrades/migrations, write automation, own P1 incidents |

### Lab Environment Setup (Week 0)

| Component | Recommended |
|-----------|-------------|
| OS | Ubuntu 22.04 LTS (VM or Docker) |
| MongoDB | 7.0 Community (match production major version) |
| Topology | 3-node replica set + 1 arbiter OR 3-member PSA on Docker |
| Tools | `mongosh`, MongoDB Compass, `mongoimport`, `mongodump/restore`, `mtools`, `Percona Monitoring` or Ops Manager (trial) |
| Automation | Docker Compose + a Git repo for scripts |

**Week 0 checklist**
- [ ] Spin up a 3-node replica set with Docker
- [ ] Connect with `mongosh` and Compass
- [ ] Load the sample datasets (`sample_mflix`, `sample_airbnb`)
- [ ] Create a Git repo `mongo-dba-lab`

---

## 2. Roadmap Overview

| Phase | Weeks | Focus | Outcome |
|-------|-------|-------|---------|
| **A. Foundations Refresh** | 1–2 | Architecture, storage engine, data modeling | Explain how WiredTiger stores data |
| **B. Core DBA Operations** | 3–4 | Backup/restore, auth, users/roles, upgrades | Run production-grade backup strategy |
| **C. Replication & HA** | 5–6 | Replica sets, elections, failover, oplog | Survive a node failure with no data loss |
| **D. Performance & Indexing** | 7–8 | Indexing, query planner, profiling, tuning | Diagnose and fix a slow query end-to-end |
| **E. Sharding & Scale** | 9–10 | Sharding, chunk balancing, zone sharding | Design a sharded cluster |
| **F. Operations & Automation** | 11 | Monitoring, alerting, automation, security hardening | Build monitoring + alerting |
| **G. Capstone** | 12 | Incident simulation + design review | Own a simulated P1 |

---

## Phase A — Foundations Refresh (Weeks 1–2)

**Goal:** Rebuild mental model of MongoDB internals to L3 depth.

### Topics
1. **Architecture**
   - `mongod` vs `mongos` vs `mongosh`
   - JSON/BSON, documents, collections, databases
   - Storage engines: **WiredTiger** (default), in-memory
   - WiredTiger internals: B-tree, WAL (journal), checkpoints, cache size
2. **Data Modeling**
   - Embedding vs referencing, cardinality rules
   - Schema versioning & migrations
   - Anti-patterns: unbounded arrays, massive `$lookup`s
3. **CRUD & Query Language**
   - `find`, `insertOne/Many`, `updateOne/Many`, `delete*`
   - Aggregation pipeline (`$match`, `$group`, `$lookup`, `$facet`, `$graphLookup`)
   - Transactions (multi-document, sessions)

### Lab Tasks
- [ ] Point out where the journal, WAL, and data files live on disk
- [ ] Tune `wiredTigerCacheSizeGB` and observe memory usage
- [ ] Model a blog (posts/comments) two ways and benchmark reads
- [ ] Write an aggregation pipeline joining 2 collections

### Checkpoint Questions
- What happens on write if the journal is enabled/disabled?
- When do you embed and when do you reference?
- What is a checkpoint interval and why does it matter for recovery?

### Resources
- MongoDB Manual: "Storage Engines", "Data Modeling Introduction"
- *MongoDB: The Definitive Guide* (Ch. 1–6)

---

## Phase B — Core DBA Operations (Weeks 3–4)

**Goal:** Own day-to-day operations confidently.

### Topics
1. **Backup & Restore**
   - `mongodump` / `mongorestore` (logical)
   - Filesystem snapshots (physical)
   - **Oplog backup** & **Point-in-Time Recovery (PITR)**
   - Consistency on replica sets (secondary read preference)
2. **Authentication & Authorization**
   - SCRAM-SHA-256, x.509, LDAP/Kerberos (enterprise)
   - Built-in roles, custom roles, privilege actions
   - `authSource`, internal authentication (keyfiles)
3. **User & Role Management**
   - Least-privilege design per app
   - Rotating credentials without downtime
4. **Upgrades & Maintenance**
   - Upgrade path (e.g., 6.0 → 7.0), FCV (`featureCompatibilityVersion`)
   - Rolling upgrade on replica set
   - `db.currentOp()`, `killOp()`, `compact`, `reIndex`

### Lab Tasks
- [ ] Script a full + incremental backup with PITR to an S3-like store
- [ ] Restore to a separate instance and verify document counts
- [ ] Create app users with custom roles; test privilege escalation fails
- [ ] Perform a rolling upgrade from 6.0 to 7.0 in Docker

### Checkpoint Questions
- Logical vs physical backup — when do you pick each?
- How does PITR work with the oplog?
- What breaks during an upgrade if FCV isn't updated?

### Resources
- MongoDB Manual: "Backup Methods", "Security", "Upgrade MongoDB"
- `mongodb-cheatsheet.html` → Backup & Restore section

---

## Phase C — Replication & High Availability (Weeks 5–6)

**Goal:** Design and operate resilient replica sets.

### Topics
1. **Replica Set Concepts**
   - Members: primary, secondary, arbiter; priorities
   - **Oplog**: sizing, window, replication lag
   - Elections, voting, majority write concern
2. **Read/Write Concern**
   - `w`, `j`, `wtimeout`, `readConcern` (`local`, `majority`, `linearizable`)
   - Read preferences: primary, secondary, nearest, tags
3. **Failure Handling**
   - Simulating node failure & failover
   - Recovering a stale/downed member (resync strategies)
   - Rollback & why it happens
4. **Advanced**
   - Delayed members, hidden members, changestreams

### Lab Tasks
- [ ] Size the oplog for a 24-hour window under load; verify
- [ ] Kill the primary and observe election + app behavior
- [ ] Add a delayed member for accidental-delete protection
- [ ] Force a rollback and demonstrate recovery

### Checkpoint Questions
- What is the oplog window and how do you ensure it's safe?
- Majority write concern vs majority read concern — what each guarantees
- When does a rollback occur and how do you prevent it?

### Resources
- MongoDB Manual: "Replication", "Write Concern", "Read Concern"
- Mongo University: **M201 — MongoDB for DBAs**

---

## Phase D — Performance & Indexing (Weeks 7–8)

**Goal:** Diagnose and fix performance issues from first principles.

### Topics
1. **Indexing**
   - B-tree, compound (ESR rule), multikey, partial, sparse, TTL, text, wildcard, geospatial
   - Covered queries, index intersection
2. **Query Planner & Explain**
   - `explain("executionStats")`: `COLLSCAN` vs `IXSCAN`
   - `totalDocsExamined`, `totalKeysExamined`, `nReturned`
3. **Monitoring & Profiling**
   - Database Profiler, slowms setting
   - `serverStatus`, `dbStats`, `collStats`, `$indexStats`
   - `mongostat`, `mongotop`, `currentOp`
   - Ops Manager / PMM dashboards
4. **Tuning**
   - WiredTiger cache, connection pooling, working set → RAM sizing
   - Identifying lock contention and long-running ops

### Lab Tasks
- [ ] Generate 1M docs, run a query before/after adding the correct index
- [ ] Use `explain` to prove a query is covered
- [ ] Profile a workload and find the top 3 slow queries
- [ ] Fix a COLLSCAN-caused CPU spike

### Checkpoint Questions
- State the **ESR rule** for compound indexes.
- What does `totalDocsExamined >> nReturned` tell you?
- How do you size RAM from your working set?

### Resources
- MongoDB Manual: "Indexes", "Explain Results", "Performance Best Practices"
- Mongo University: **M201**, **M320 Data Modeling**

---

## Phase E — Sharding & Scale (Weeks 9–10)

**Goal:** Design and operate a sharded cluster.

### Topics
1. **Sharding Fundamentals**
   - Shards, `mongos`, config servers (CSRS)
   - Shard keys: hashed vs ranged, cardinality, monotonic growth
2. **Chunks & Balancing**
   - Chunk splits, migrations, balancer windows
   - Jumbo chunks
3. **Zone Sharding**
   - Data residency, tiered storage
4. **Operations**
   - `sh.status()`, `sh.enableSharding()`, `sh.shardCollection()`
   - Resharding, moving chunks, adding/removing shards

### Lab Tasks
- [ ] Stand up a 2-shard cluster in Docker
- [ ] Shard a collection with a hashed key and inspect chunk distribution
- [ ] Configure zones for a `region` field
- [ ] Compare performance: sharded vs unsharded for a hot-key workload

### Checkpoint Questions
- What makes a good shard key? What makes a bad one?
- Why do jumbo chunks form and how do you clear them?
- When would you *not* shard?

### Resources
- MongoDB Manual: "Sharding", "Shard Keys", "Zone Sharding"
- Mongo University: **M330 Diagnostics**

---

## Phase F — Operations, Monitoring & Automation (Week 11)

**Goal:** Move from manual to automated, observable operations.

### Topics
1. **Monitoring & Alerting**
   - Key metrics: connections, opcounters, replication lag, WT cache pressure, page faults
   - Ops Manager / PMM / Prometheus + Grafana exporter
   - Alert thresholds that actually matter
2. **Security Hardening**
   - Network: TLS, firewall, bindIp
   - Encryption at rest (Enterprise / KMIP)
   - Auditing, RBAC review
3. **Automation**
   - Idempotent scripts, config management (Ansible), CI for DB changes
   - Backup verification jobs, restore drills
4. **Capacity Planning & Runbooks**
   - Growth forecasting, disk/IOPS/RAM models
   - Writing actionable runbooks (symptom → diagnosis → fix)

### Lab Tasks
- [ ] Deploy Prometheus + mongodb_exporter + Grafana
- [ ] Create 5 alerts (lag, connection saturation, cache pressure)
- [ ] Write a runbook for "replica set member down"
- [ ] Automate a nightly backup + restore-verify job

### Checkpoint Questions
- Which 5 metrics would you put on a NOC dashboard?
- How do you enable TLS on a running replica set?
- What belongs in a good runbook?

### Resources
- MongoDB Manual: "Monitoring", "Security Checklist"
- PMM / mongodb_exporter docs

---

## Phase G — Capstone (Week 12)

**Goal:** Prove L3 readiness by owning a full incident and a design.

### Capstone 1 — Incident Simulation
Break your lab deliberately and resolve under time pressure:
- [ ] Delete the primary's data directory → recover via resync
- [ ] Fill the oplog → force a resync scenario
- [ ] Introduce a bad index → detect slowdown → fix
- [ ] Simulate disk-full and recover safely
- Write a post-incident report (timeline, root cause, prevention).

### Capstone 2 — Design Review
- [ ] Design a multi-region, sharded, HA cluster for a hypothetical SaaS
- Include: topology, shard key choice, backup/PITR strategy, security model, capacity model, monitoring.
- Present it as if to an architecture board (1-page + diagrams).

### Final Self-Assessment
Re-score yourself against the L1/L2/L3 table in Section 1. You are L3 when you can handle a **novel** incident without a runbook.

---

## 3. Weekly Study Schedule (Template)

| Day | Activity | Time |
|-----|----------|------|
| Mon | Read official docs (deep) | 1.5 h |
| Tue | Watch video course | 1 h |
| Wed | Hands-on lab | 2 h |
| Thu | Read docs + take notes | 1.5 h |
| Fri | Lab / troubleshooting | 2 h |
| Sat | Write runbook + review week | 2 h |
| Sun | Rest / catch-up | — |

**Total:** ~10 hrs/week.

---

## 4. Assessments & Milestones

| Week | Milestone | Evidence |
|------|-----------|----------|
| 2 | Architecture quiz | Notes + `explain` of storage layout |
| 4 | Backup/restore drill | Script + verified restore |
| 6 | Failover drill | Logs + post-mortem |
| 8 | Performance case study | Before/after `explain` |
| 10 | Sharded cluster | `sh.status()` output + diagram |
| 11 | Monitoring stack | Grafana dashboard screenshot |
| 12 | Capstone | Incident report + design doc |

---

## 5. Certifications (Optional, Aligned)

| Cert | Level | When |
|------|-------|------|
| MongoDB Associate DBA | Entry | After Week 4 |
| MongoDB Certified DBA | Professional | After Week 12 |

- Official learning path: **MongoDB University**
- Recommended courses: **M001, M103, M201, M320, M330**

---

## 6. Essential Tools Reference

| Task | Tool |
|------|------|
| Shell | `mongosh` |
| GUI | MongoDB Compass |
| Backup | `mongodump`, `mongorestore`, `mtools` (oplog replay) |
| Monitoring | `mongostat`, `mongotop`, Ops Manager, PMM, Prometheus |
| Load testing | `mongoimport`, `YCSB`, `mgeneratejs` |
| Cluster mgmt | Ops Manager / Cloud Manager, Ansible |

---

## 7. Key Runbooks to Build

1. Replica set member down / resync
2. Backup verification & PITR restore
3. Slow query triage (explain → index)
4. Replication lag investigation
5. Disk-full recovery
6. User/role provisioning & rotation
7. Rolling upgrade procedure
8. Shard balancing & jumbo chunk resolution

---

## 8. Progress Tracker

- [ ] Phase A — Foundations
- [ ] Phase B — Core DBA Ops
- [ ] Phase C — Replication & HA
- [ ] Phase D — Performance & Indexing
- [ ] Phase E — Sharding & Scale
- [ ] Phase F — Ops, Monitoring, Automation
- [ ] Phase G — Capstone

---

*Last updated: auto-generated learning plan. Pair each phase with the companion `mongodb-cheatsheet.html` and the linked video playlist.*
