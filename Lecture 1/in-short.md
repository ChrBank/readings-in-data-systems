# In short

## 1. Introduction
* RDBMSs are still based on System R’s 1970s architecture.
* Hardware and workloads have changed, but DBMS design has not.
* Previous work showed RDBMSs underperform in warehouses, streams, text, and scientific data.
* This paper shows they also fail in OLTP, using the new prototype H-Store (82× faster than a commercial RDBMS).
* Conclusion: legacy RDBMS must be rewritten from scratch.

## 2. OLTP Design Considerations
Modern OLTP workloads differ from 1970s assumptions. Key points:
* **Main memory**: OLTP DBs fit in RAM, so disk-oriented architectures are outdated.
* **Single-threading**: Lightweight OLTP transactions make multi-threading overhead unnecessary.
* **Grid/cluster support**: New systems should scale out easily without “forklift upgrades.”
* **High availability**: Peer-to-peer replication replaces complex logging.
* **No knobs**: Systems must be self-tuning, not dependent on databaseadministrators (DBAs).

## 3. Transaction Processing & Environment Assumptions
* Assumes: in-memory storage, high availability, short transactions, no user stalls.
* Implications:
  * Redo logging, JDBC/ODBC overhead, dynamic locking, and 2PC are bottlenecks and should be avoided.
* Schema & workload observations:
  * Many OLTP schemas are **tree-shaped** (CTA “constrained tree applications”), enabling partitioning so transactions run at a single site.
  * Read-only tables can be replicated, making apps easier to partition.
  * **Two-phase transactions** (abort decision before updates) allow eliminating undo logs.
  * **Sterile transactions** (commutative) and one-shot transactions (executed independently at sites) simplify execution.

## 4. H-Store Sketch
* H-Store’s architecture exploits these assumptions:
  * **Grid-based, in-memory, single-threaded sites** (one per CPU core).
  * Transactions are predefined stored procedures (not ad-hoc SQL).
  * **No redo log**, undo log only if necessary (and in-memory).
  * Optimizer is simple (OLTP queries don’t need complex joins).
  * **Automatic database designer** partitions data to maximize single-site execution.
  * **Replication** ensures HA without redo logging.
  * **Optimistic concurrency control** with fallback strategies:
    * Run sterile/single-site transactions without control.
    * If conflicts → escalate to more careful coordination.

## 5. Performance Comparison
* Benchmark: TPC-C (standard OLTP).
* Strategy: partition warehouses across sites, replicate read-only data.
* With tuning, all 5 TPC-C transactions can be made one-shot, strongly two-phase, and sterile.
* Results:
  * H-Store: ~70,000 TPS
  * Commercial RDBMS: ~850 TPS
  * → H-Store is **82× faster** on same hardware.
* Commercial system bottlenecks: redo logging and concurrency control.
* Even compared to record-setting large TPC-C systems, H-Store (on a desktop!) approaches similar per-core performance.

## 6. One Size Does Not Fit All
* Specialized DBMSs will dominate, not monolithic RDBMS.
* Different markets require different data models (not always relational) and languages (not always SQL).
* Example:
  * Warehouses → star/snowflake schemas (better with E-R models).
  * Streams → hierarchical messages, not flat SQL tables.
  * Text → not relational at all.
  * Scientific → arrays instead of tables.
* SQL should be replaced with simpler, domain-specific languages.
* Future DBMS may embed storage and queries into little languages (like Ruby-on-Rails), not via JDBC/ODBC.

## 7. Summary & Future Work
* RDBMS architecture is outdated for all markets.
* Specialized engines with rethought data models and languages are necessary.
* H-Store proves orders-of-magnitude performance gains.
* Future work: automate partitioning, concurrency control strategies, optimize in-memory structures, integrate with warehouses.
* Predicts next 15 years will mirror 1970–85 DBMS upheaval: rapid experimentation and major industry shifts.