# MongoDB Learning

Personal study notes and resources for MongoDB, focused on the **DBA track** (L1 → L2 → L3).

## Contents

| File | Description |
|------|-------------|
| [`index.html`](index.html) | Landing page linking to every file in this folder |
| [`mongodb-cheatsheet.html`](mongodb-cheatsheet.html) | MongoDB commands reference (queries, admin, ops) |
| [`MongoDB-DBA-Learning-Plan.md`](MongoDB-DBA-Learning-Plan.md) | 12-week DBA learning roadmap (L1 → L2 → L3) |
| [`markdown-editor.html`](markdown-editor.html) | Browser-based Markdown editor tool |
| [`mongodb_wiredtiger_memory_map.jpg`](mongodb_wiredtiger_memory_map.jpg) | Diagram of the WiredTiger memory map |
| [`Notes`](Notes) | Scratch notes |
| [`Youtube`](Youtube) | Saved video links |

## Quick Start

Open the index in a browser:

```bash
xdg-open index.html      # Linux
open index.html          # macOS
```

Or serve the folder locally:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Learning Plan Summary

- **Duration:** 12 weeks (~10–12 hrs/week)
- **Method per module:** Read docs → Watch video → Lab on a local replica set → Document in a runbook → Quiz yourself
- **Golden rule:** Never learn a command without running it against a real replica set — a single-node `mongod` hides ~80% of production issues.

### Phases

| Phase | Weeks | Focus |
|-------|-------|-------|
| A. Foundations Refresh | 1–2 | Architecture, storage engine, data modeling |
| B. Core DBA Operations | 3–4 | Backup/restore, auth, users/roles, upgrades |
| C. Replication & HA | 5–6 | Replica sets, elections, failover, oplog |
| D. Performance & Indexing | 7–8 | Indexing, query planner, profiling, tuning |
| E. Sharding & Scale | 9–10 | Sharding, chunk balancing, zone sharding |

## Lab Environment

| Component | Recommended |
|-----------|-------------|
| OS | Ubuntu 22.04 LTS (VM or Docker) |
| MongoDB | 7.0 Community (match production major version) |
| Topology | 3-node replica set + 1 arbiter, or 3-member PSA on Docker |
| Tools | `mongosh`, MongoDB Compass, `mongoimport`, `mongodump`/`mongorestore`, `mtools` |
| Automation | Docker Compose + a Git repo for scripts |

## External Resources

- [Load Sample Data (local) — MongoDB Docs](https://www.mongodb.com/docs/manual/sample-data/load-sample-data-local/)
- [YouTube — MongoDB playlist](https://www.youtube.com/watch?v=cLsawKBUdTE&list=PLSmSa8KVdfSu-XFvjdWoN7z9WRoLly4my)