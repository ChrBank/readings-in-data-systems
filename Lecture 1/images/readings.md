# The end of an architecural era notes



## Goal of the paper
> These papers presented reasons and experimental evidence that showed that the major RDBMS vendors can be outperformed by 1-2 orders of magnitude by specialized engines in the data warehouse, stream processing, text, and scientific database markets.

> Show that current RDBMSs can be beaten by nearly two orders of magnitude in the OLTP market as well.

## What it does
> **Comparing** a new OLTP prototype, **H-Store**, which we have built at M.I.T., to a popular **RDBMS** on the standard transactional benchmark, TPC-C.

## Conclusion
> * “Current RDBMS code is outdated ‘one-size-fits-all’ that excels at nothing. 
> * These 25-year-old legacy lines should be retired and replaced with specialized engines, designed from scratch for tomorrow’s needs—not yesterday’s.”
> * The DBMS vendors (and the research community) should start with a clean sheet of paper and design systems for tomorrow’s requirements, not continue to push code lines and architectures designed for yesterday’s needs.

### The new needs for the marked
> In the **Data warehouse market**, nearly 100% of all schemas are stars or snowflakes, containing a central fact table with 1-n joins to surrounding dimension tables, which may in turn participate in further 1-n joins to second level dimension tables, and so forth.
> An entity-relationship model (E-R model) would be simpler in this environment and more natural.
> Lastly, warehouse operations that are incredibly expensive with a relational implementation, for example changing the key of a row in a dimension table, might be made faster with some sort of E-R implementation.

> In the **stream processing market**, there is a need to:
> 1. Process streams of messages at *high speed*
> 2. Correlate such streams with *stored data*
> 
> To accomplish both tasks, there is widespread enthusiasm for **StreamSQL**, a generalization of SQL that allows a programmer to mix stored tables and streams in the FROM clause of a SQL statement.
> 
> Stream processing systems, such asStreamBase and Coral8, *currently support only flat (relational) messages*. In such systems, a front-end adaptor must normalize hierarchical objects into several flat message types for processing. 
> It is rather painful to join the constituent pieces of a source message back together when processing on multiple parts of a hierarchy is necessary.
> *To solve this problem*, we expect the stream processing vendors to move aggressively to hierarchical data models.

> **Text processing** obviously has never used a relational model.

> Any **scientific-oriented DBMS**, such as ASAP, will probably implement arrays, not tables as their basic data type.

> There is certainly fierce debate over the excessive complexity of XMLSchema. There are fans of using RDF for such data and some who argue that RDF can be efficiently implemented by a relational column store.

### SQL is Not the Answer
> SQL is a “one size fits all” language.

In the programming language community, there has been an explosion of “little languages” such as Python, Perl, Ruby and PHP. Also little languages are attractive because they are easier to learn than general purpose languages. From afar, this phenomenon *appears to be the death of “one size fits all”* in the programming language world.

## Keywords
* **DBMS** = Database Management System
* **RDBMS** = Relational Database Management System
* **OLTP** = Online transaction processing
* **IBM System R** = "System R was a seminal project as the first implementation of SQL, which has since become the standard relational data query language"
* **DB2** = "DB2 is a family of data management products, including database servers. It initially supported the relational model, but was extended to support object–relational features and non-relational structures like JSON and XML"
* **Sybase System** = "A Sybase server is *one copy of the database engine executing on a computer*. A server has a name, and when a program wants to access the database controlled by a server, the program asks for a connection to that server by name. Multiple servers can be executing on a single machine, controlling different databases"
* **Bitmap indexes** = "Bitmap indexes use bit arrays (commonly called bitmaps) and answer queries by performing bitwise logical operations on these bitmaps. A gender column, which has only two distinct values (male and female), is ideal for a bitmap index."
* **Star schema** = is a multi-dimensional data model used to organize data in a database so that it is easy to understand and analyze. Star schemas can be applied to data warehouses, databases, data marts, and other tools. The star schema design is optimized for querying large data sets.
* **Snowflake** = "A Snowflake database is where an organization's uploaded structured and semistructured data sets are held for processing and analysis. Snowflake automatically manages all parts of the data storage process, including organization, structure, metadata, file size, compression, and statistics."
* **RDF** = Resource Description Framework is a method to describe and exchange graph data.
* **TPC-C** = "Transaction Processing Performance Council Benchmark C, is a benchmark used to compare the performance of online transaction processing (OLTP) systems.".
* **Grid computing** = "is a group of networked computers that work together as a virtual supercomputer to perform large tasks, such as analyzing huge sets of data or weather modeling."
* **SMP** = Session Multiplex Protocol (SMP) is an application-layer protocol that provides session management capabilities between a database client and a database server.
* **C-Store** = "C-Store differs from most traditional relational database management system (RDBMS) designs in many ways, primarily in that it stores data by column and not by row, optimizing the database for reading of data rather than writing."
* **K-Safety** = "K-safety sets the fault tolerance in your Enterprise Mode database cluster. The value K represents the number of times the data in the database cluster is replicated. These replicas allow other nodes to take over query processing for any failed nodes."
* **B tree index** = "A B-tree index creates a multi-level tree structure that breaks a database down into fixed-size blocks or pages. Each level of this tree can be used to link those pages via an address location, allowing one page (known as a node, or internal page) to refer to another with leaf pages at the lowest level."
* **JDBC** JDBC stands for Java Database Connectivity. It is a Java application programming interface (API) that manages connecting to a database, issuing queries and commands, and handling the results.
* **NTP** = Network Time Protocol (synchronizes whatches)

> Sybase ASE was renamed to **SAP ASE**. A *high-performance* SQL database server that uses a *relational management model* to meet rising demand for *performance*, *reliability*, and *efficiency* in every industry.


## OLTP Design Considerations
Presents five major issues, which a new engine such as H-Store can leverage to achieve dramatically better performance than current RDBMSs.
### Main memory 
* In the late 1970’s a large machine had somewhere around a
**megabyte** of main memory. Today, several **Gbytes** are common and large machines are approaching 100 Gbytes.
* In a few years a **terabyte** of main memory will not be unusual.

>The overwhelming majority of OLTP databases are less than 1 Tbyte in size and growing in size quite slowly. 
We believe that OLTP should be *considered a main memory market*, if not now then within a very small number of years.
> 
> The current RDBMS vendors have *disk-oriented* solutions for a main memory problem.

### Multi-threading and Resource Control
* **OLTP** transactions are very lightweight. (the heaviest transaction in TPC-C reads about 400 records).
* Current **RDBMSs** have elaborate multi-threading systems to try to fully utilize CPU and disk resources.
  * Allows *several-to-many queries* to be running in *parallel*.
  * They have *resource governors to limit the multiprogramming load*, so that other resources (IP connections, file handles, main memory for sorting, etc.) *do not become exhausted*.
  * **These features are irrelevant in a single threaded execution model.** No resource governor is required in a single threaded system.

> For example, when an Amazon user clicks “buy it” , he activates an OLTP transaction which will only report back to the user when it finishes. Because of an absence of disk operations and user stalls, the elapsed time of an OLTP transaction is minimal. 
> In such a world it makes sense to run all SQL commands in a transaction to completion with a **single-threaded execution** model, rather than paying for the overheads of isolation between concurrently executing statements.

> In a **single-threaded execution model**, there is also **no reason to have multi-threaded data structures**. *This results in a more reliable system, and one with higher performance*.


#### “What about long running commands?” 
> In real-world OLTP systems, there aren’t any for two reasons: 
> 1. First, operations that appear to involve long-running transactions, such as a user inputting data for a purchase on a web store, are usually split into several transactions to keep transaction time short. In other words, good application design will keep OLTP queries small. 
> 2. Second, longer-running ad-hoc queries are not processed by the OLTP system; instead such queries are directed to a data warehouse system, optimized for this activity. There is no reason for an OLTP system to solve a non-OLTP problem. Such thinking only applies in a “one size fits all” world.

### Grid Computing and Fork-lift Upgrades
* Current RDBMSs were originally written for the prevalent architecture of the 1970s, namely *shared-memory multiprocessors*.
* It seems plausible that the next decade will bring domination by *shared-nothing computer systems*, often called grid computing or blade computing. 

> Hence, any DBMS should be optimized for this configuration.
> An obvious strategy is to **horizontally partition** data over the nodes of a grid.
> 
> In addition, no user wants to perform a **“fork-lift” upgrade** (heavy lifting or change). Hence, any new system should be architected for incremental expansion. If N grid nodes do not provide enough horsepower, then one should be able to add another K nodes, producing a system with N+K nodes.
>
> To achieve incremental upgrade without going down requires significant capabilities, *not found in existing systems*.

### High Availability
* **Single point of failure**: Relational DBMSs were designed in an era (1970s) when an organization had a single machine. If it went down, then the company lost money due to system unavailability.

> Today, there are numerous organizations that run a **hot standby** within the enterprise, so that real-time failover can be accomplished.
> Alternately, some companies run **multiple primary sites**, so failover is even quicker.
>
> Businesses are much more willing to pay for multiple systems in order to avoid the crushing financial consequences of down time, often estimated at thousands of dollars per minute.

* In the future, we see high availability and built-in disaster recovery as essential features in the OLTP (and other) markets. 

Why?
1. First, **every OLTP DBMS will need to keep multiple replicas consistent**, requiring the ability to run seamlessly on a grid of geographically dispersed systems.
2. Second, most existing RDBMS vendors have glued multi-machine support onto the top of their original SMP architectures. In contrast, **it is clearly more efficient to start with shared-nothing support** at the bottom of the system.
3. Third, the best way to support shared nothing is to use multiple machines in a **peer-to-peer configuration**. In this way, the OLTP load can be dispersed across multiple machines, and inter- machine replication can be utilized for fault tolerance. That way, all machine resources are available during normal operation. Failures only cause degraded operation with fewer resources. In contrast, many commercial systems implement a hot standby, whereby a second machine sits effectively idle waiting to take over if the first one fails.

> In an HA (high availablity) system, regardless of whether it is hot-standby or peer-to-peer, **logging can be dramatically simplified**. 
> One must continue to have an undo log, in case a transaction fails and needsto roll back. However, the undo log does not have to persist beyond the completion of the transaction. 
> As such, it can be a main memory data structure that is discarded on transaction commit. **There is never a need for redo**, because that will be accomplished via network recovery from a remote site. When the dead site resumes activity, it can be refreshed from the data on an operational site.

### No Knobs
* Current systems were built in an era where resources were incredibly expensive and and people were cheap to help take care of the machines. 
* Today we have the reverse. Personnel costs are the dominant expense in an IT shop.
* As such “self-everything” (self-healing, self-maintaining, self-tuning, etc.) systems are the only answer.
* All RDBMSs have a vast array of complex tuning knobs (for humans to adjust), which are legacy features from a bygone era.
* *A much better answer is to completely rethink the tuning process and produce a new system with no visible knobs*.

### In short
**OLTP Design Considerations**
Modern OLTP workloads differ from 1970s assumptions. Key points:
* **Main memory**: OLTP DBs fit in RAM, so disk-oriented architectures are outdated.
* **Single-threading**: Lightweight OLTP transactions make multi-threading overhead unnecessary.
* **Grid/cluster support**: New systems should scale out easily without “forklift upgrades.”
* **High availability**: Peer-to-peer replication replaces complex logging.
* **No knobs**: Systems must be self-tuning, not dependent on databaseadministrators (DBAs).

## Transaction, Processing and Environment Assumptions
### **Core Assumptions**
> * Main memory storage (most OLTP DBs < 1TB, fits in RAM)
> * High availability (HA) and replication → no need for complex redo logging.
> * Transactions are short-lived (hundreds of records, <1ms runtime).
> * No user stalls (e.g., Amazon “Buy” → transaction finishes before response).
> 
> Because of this, the overhead of old features like redo logging, dynamic locks, and two-phase commit dominate execution time.

### Immediate Implications
The authors argue several architectural features must go:
1. **Redo log → bottleneck**
   * Writing commit records to disk adds milliseconds.
   * In an HA system, recovery comes from replicas instead of redo logs.
   * → Redo logging should be eliminated.
2. **Client/server overhead**
   * JDBC/ODBC calls are too expensive (inter-process overhead).
   * → Instead, run application logic in-process as stored procedures.
   * Example: stored procedures written in C++ embedded directly in H-Store.
3. **Undo log → bottleneck**
   * Rolling back updates adds overhead.
   * If transactions are **two-phase** (abort decision before updates), undo logging is unnecessary.
4. **Dynamic locking → bottleneck**
   * Traditional RDBMS use locks for concurrency.
   * But OLTP transactions are so short that locking overhead dominates.
   * → Use single-threaded execution and optimistic concurrency.
5. **Multi-threaded latching overhead**
   * Shared data structures need locks/latches.
   * If each site is single-threaded, you eliminate these costs.
6. **Two-phase commit (2PC)**
   * Distributed transactions require network round trips → milliseconds.
   * OLTP transactions can often be partitioned to a single site.
   * → Avoid 2PC wherever possible.



### Transaction and Schema Characteristics
The authors analyze typical OLTP schemas and workloads to justify their optimizations.

1. **Predefined Workloads**
   * H-Store assumes the workload is a fixed set of **transaction classes**
   * Each class = fixed SQL + program logic, only parameters change.
   * No ad-hoc queries in OLTP → realistic assumption.
   * Stored procedures must be registered in advance.
2. **Tree Schemas (Constrained Tree Applications, CTAs)**
   * Many OLTP schemas have a tree shape:
     * Root table (e.g., Customer)
     * Children (Orders → LineItems → Shipments)
     * Edges are 1-to-many relationships.
   * → Schema can be partitioned so all data for a customer (root) lives at one site.
   * If each transaction only touches one root, it’s single-sited → no distributed coordination.
3. **Constrained Tree Applications (CTAs)**
   * Definition: schema + workload where all transactions can be executed at a single site.
   * Example: e-commerce app where all actions are rooted in customer_id.
   * Result:
     * No distributed locks
     * No 2PC
     * Transactions finish locally, fast.
4. **Transformations to Achieve CTAs**
   * Even if schema/workload is not naturally a CTA, you can sometimes transform it:
     1. **Replicate read-only tables everywhere** → avoids cross-site joins. 
   E.g., product catalog can be fully copied to each node.
      2. **Vertical partitioning** → replicate certain non-updated columns across sites.
      Example: Stock table in TPC-C can be split so static attributes are replicated.
      3. **One-shot transactions** → transactions decomposable into independent subplans that can run in parallel across sites without interdependencies.
5. **Two-Phase Transactions**
   * **Phase 1**: read-only checks (may abort).
   * **Phase 2**: updates (cannot abort).
   * → With this pattern, **undo logs aren’t needed**.
   * Many OLTP workloads naturally fit this (e.g., TPC-C new_order → first check item availability, then insert/update).
   * **Strongly two-phase transactions**:
     * Phase 1 decisions are consistent across all replicas.
     * Ensures deterministic abort/commit, simplifying replication.
6. **Sterile Transactions**
   * Definition: two transactions **commute** (regular journey of some distance) if executing them in any order yields the same result.
   * Example: two payments that update different customers.
   * Sterile transactions commute with everything.
   * If transactions are sterile:
     * No concurrency control is needed.
     * Replicas don’t even need global ordering.

> Many OLTP workloads every table except a single one called the root, has exactly one join term which is a n-1 relationship to its ancestor. 
> Hence, the schema is a tree of 1-n relationships.
>
> Tree schemas have an obvious horizontal partitioning over the nodes in a grid. Specifically, the root table can be range or hash partitioned on the primary key(s). Every descendent table can be partitioned such that all equi-joins in the tree span only a single site.

## H-Store Sketch
Is a design blueprint for H-Store, showing how to exploit OLTP characteristics (short, predictable transactions; tree schemas; single-sited execution) to get massive performance gains.
### System Architecture
* Grid of computers (shared-nothing cluster).
* Data is horizontally partitioned across grid nodes.
* User specifies K-safety (number of replicas).

#### At each site:
* **Rows stored in main memory** (no disk I/O in normal ops).
* **Conventional B-trees** for indexing (cache-tuned to L2 size).
* **Single-threaded execution per core**:
  * Each CPU core is treated as an independent "logical site".
  * No need for locks, latches, or multi-threaded data structures.

#### Transactions:
* Predefined **stored procedures** (registered in advance).
* Transactions run to completion, with no interruption.
* No ad-hoc SQL queries.
* Stored procedures currently written in C++, but language choice is flexible (later, they consider Ruby-on-Rails).

#### Logging:
* No redo log.
* Undo log written only when necessary, and only to memory (discarded after commit).

### Query Execution
* Optimizer is **simple** because OLTP queries are short (few joins, no complex aggregates).
* Plans are compiled at transaction-definition time.

#### Query plan types:
1. **Single-sited**: all commands execute on one site (ideal case).
2. **One-shot**: decomposable into independent subplans, executed in parallel across sites, no interdependencies.
3. **General**: some commands need intermediate results or cross-site communication → falls back to Gamma-style execution model.
4. **Transaction depth**: number of inter-site messages needed. Deeper transactions → higher latency.

### Database Designer
* Goal: make transactions single-sited wherever possible.
* Automatic physical design tool decides:
  * Partitioning strategy
  * Replication strategy
  * Indexes
* Uses schema + workload knowledge (transaction classes) to maximize single-site execution.

#### Strategy:
* Partition root table across sites.
* Assign dependent tuples (children in schema tree) to same site as their root.
* Replicate read-only tables everywhere.
* Apply vertical partitioning where useful.
* Manual partitioning/indexing also possible, but automation is the ultimate goal.

### Transaction Management, Replication, and Recovery
H-Store simplifies transaction processing using schema/workload properties.

#### Replication:
* Each table has two+ replicas.
* Reads go to any replica.
* Writes are broadcast to all replicas.

#### Timestamps:
* Every transaction gets a unique global timestamp (site_id + local clock).
* Clocks are kept loosely synchronized (e.g., via NTP).
* Timestamps provide a global order of transactions.

#### Execution cases:
1. **Single-sited / one-shot**
   * Transaction runs at one site (or independently across sites).
   * Small delay ensures replicas apply transactions in the same order.
   * No redo log, no 2PC, no concurrency control.
2. **Two-phase transactions**
   * No undo log needed.
   * Updates only happen after "abort decision" in phase 1.
3. **Sterile transactions**
   * Commute with everything → no concurrency control or global ordering needed.
4. **General case**
   * If transactions aren’t single-sited/one-shot/sterile → must use optimistic concurrency control:
     * Each site checks for conflicts based on timestamps.
     * If conflict → abort.

#### Optimistic strategies:
* **Basic strategy**: run transaction, abort if conflicting.
* **Intermediate strategy**: add a small delay to catch possible earlier conflicting transactions before execution.
* **Advanced strategy**: track read/write sets and use optimistic validation (abort only if true conflicts).
* H-Store escalates from basic → intermediate → advanced depending on abort rates.

### Key Takeaways from chapter 4
* **In-memory, shared-nothing, single-threaded per core** → removes most traditional DB overhead.
* **No redo logging, minimal undo logging** → big performance gain.
* **Stored procedures only, no ad-hoc queries** → predictable workloads.
* **Automatic partitioning + replication** → makes most transactions single-site or one-shot.
* **Optimistic concurrency control with fallback** → handles the few multi-site conflicts efficiently.

## A Performance comparison
This chapter tests H-Store vs. a popular commercial RDBMS using the TPC-C benchmark (the industry standard for OLTP).

![alt text](images/experiment.png)

### TPC-C Benchmark Recap
* Simulates a wholesale supplier managing orders, stock, payments, and deliveries.
* Schema includes **Warehouse, District, Customer, Order, Order-line, Stock**, Item tables.
* 5 transaction classes (must follow defined mixes):
  * **New-order** (50% of workload) – places customer orders.
  * **Payment** (≥43%) – updates balances.
  * **Order-status** (≥4%) – queries most recent order.
  * **Delivery** (≥4%) – delivers outstanding orders.
  * **Stock-level** (≥4%) – checks inventory thresholds.

### Partitioning & Schema Strategy
* TPC-C schema is **not strictly a tree** (Stock & Item cause cross-links).
* Authors transform it into a **single-site / one-shot schema**:
  * **Item** table = read-only → replicate at all sites.
  * **Order-line** partitioned with Orders (customer/warehouse-based).
  * **Stock** table vertically partitioned:
    * Updateable parts stay partitioned.
    * Read-only parts replicated everywhere.

* With these tweaks, all TPC-C transactions become:
  * **One-shot** (no interdependencies across sites).
  * **Strongly two-phase** (abort decision always same across replicas).
  * **Sterile** (commutative, no concurrency conflicts).
* This means **ACID properties are preserved with no traditional overheads** (no logging, no 2PC, no locks).

### Implementation Setup
* Both systems run on:
  * Dual-core 2.8GHz CPU
  * 4 GB RAM
  * Four 250 GB SATA disks
* H-Store:
  * Stored procedures in C++
  * Direct procedure calls (not JDBC)
  * No redo logging, minimal undo logging
* Commercial RDBMS:
  * Stored procedures in proprietary SQL language
  * Full ARIES logging, locks, multi-threading
  * Tuned by a professional DBA (several days effort)

### Results
* **H-Store**: 70,416 transactions per second (TPS)
* **Commercial RDBMS**: 850 TPS
* **Performance difference**: 82× faster on identical hardware

#### Why so different?
* Commercial RDBMS bottlenecks:
  * **Logging overhead** (2/3 of execution time spent writing redo logs).
  * Concurrency control & latching also add significant overhead.
* Even with logging **turned off**, commercial RDBMS could only reach ~2,500 TPS — still far below H-Store.

### Comparison to Industry TPC-C Records
* At the time, best official TPC-C results ≈ 133,000 TPS.
* But that required:
  * 128-core shared memory machines
  * Very high-end enterprise setups
* Per-core performance:
  * Record system: ~1,000 TPS per core
  * Commercial system in test: ~425 TPS per core
  * **H-Store: ~35,000 TPS per core** (!), on a cheap desktop machine
* H-Store achieves within a factor of 2 of the world’s best TPC-C result on commodity hardware.

### Key Takeaways from chapter 5
1. **All TPC-C transactions can be transformed into one-shot, sterile, strongly two-phase forms** with minor schema tweaks.
   * Eliminates need for logging, locking, and 2PC.

2. **H-Store achieves nearly two orders of magnitude speedup** over a commercial RDBMS.

3. **Traditional bottlenecks (logging, locking, latching, JDBC overhead)** dominate RDBMS performance.

4. **Per-core throughput of H-Store is unmatched** — shows that new architectures can extract far more value from hardware than legacy systems.

5. **Conclusion**: Legacy RDBMS are fundamentally obsolete for OLTP. H-Store proves specialized, rethought systems can shatter performance barriers.

## Summary
In the last quarter of a century, there has been a dramatic shift in:
1. DBMS markets: from business data processing to a collection of markets, with varying requirements
2. Necessary features: new requirements include shared nothing support and high availability
3. Technology: large main memories, the possibility of hot standbys, and the web change most everything

The result is:
1. The predicted demise of “one size fits all”
2. The inappropriateness of current relational implementations for any segment of the market
3. The necessity of rethinking both data models and query languages for the specialized engines, which we expect to be dominant in the various vertical markets

## Future work
* More work is needed to identify when it is possible to automatically identify single-sited, two-phase, and one-shot applications.
* The rise of multi-core machines suggests that there may be interesting optimizations related to sharing of work between logical sites physically co-located on the same machine.
* A careful study of the performance of the various transaction management strategies.
* A study of the overheads of the various components of a OLTP system – logging, transaction processing and two-phase commit, locking, JDBC/ODBC, etc -- would help identify which aspects of traditional DBMS design contribute most to the overheads we have observed.
* After stripping out all of these overheads, our H-Store implementation is now limited by the performance of in-memory data structures, suggesting that optimizing these structures will be important.
* Integration with data warehousing tools – for example, by using no-overwrite storage and occasionally dumping records into a warehouse – will be essential if H-Store-like systems are to seamlessly co-exist with data warehouses.
