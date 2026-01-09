# HadoopDB: An Architectural Hybrid of MapReduce and DBMS Technologies for Analytical Workloads

## Abstract

Data analysis is moving to **commodity shared-nothing clusters** and cloud deployments.

Debate: Parallel DBMS (performance) vs. MapReduce (scalability, fault tolerance, flexibility).

HadoopDB is proposed as a **hybrid system**: combines MapReduce (coordination layer) with single-node DBMS (execution layer).

**Goal**: performance of parallel DBMS + scalability/fault tolerance of MapReduce.

Built entirely on open source (Hadoop, PostgreSQL, Hive).

## Introduction

Analytical DBMS market is rapidly growing, with petabyte-scale data common.

Parallel DBMS = high performance but limited scalability (>100 nodes rare, fragile to failures).

MapReduce = scales to thousands of nodes but lacks structured query optimization, slower by order of magnitude.

HadoopDB vision: MapReduce as job coordinator + DBMS nodes as query executors.

Contributions:

1. Extend prior DB-vs-Hadoop comparisons to include fault tolerance/heterogeneity.

2. Hybrid design enabling shared-nothing DBMS with Hadoop.

3. Benchmarks comparing HadoopDB, Hadoop, and parallel DBMSs.

## Related Work

Pig (Yahoo), SCOPE (Microsoft), Hive (Facebook) = higher-level query interfaces for MapReduce.

Greenplum & Aster Data = allow MapReduce UDFs inside DBMS.

Prior work focused on languages, HadoopDB focuses on system-level hybridization.

## Desired Properties of Large-Scale Analytical Systems

1. **Performance**: high efficiency to save cost.
2. **Fault tolerance**: avoid restarting entire queries when a node fails.
3. **Heterogeneity support**: handle stragglers in large, non-homogeneous clusters.
4. **Flexible query interface**: SQL for BI tools + UDF support.

## Background and Shortfalls of Available Approaches

**Parallel DBMS**:

* Pros: decades of optimization, indexing, compression, SQL/ODBC support.

* Cons: assume homogeneity, fragile to failures, engineered for tens not thousands of nodes.

**MapReduce**:

* Pros: fault-tolerant, adaptive to heterogeneity, general-purpose programming.

* Cons: poor performance (no indexes, compression, cost-based planning).

**Hybrid needed**: performance of DBMS + robustness of MapReduce.

## HadoopDB Design

Core idea: **Hadoop coordinates queries**, DBMS nodes execute queries.

Components:
1. **Database Connector**: bridges Hadoop tasks with JDBC DBMS queries.
2. **Catalog**: metadata about nodes, partitions, schemas.
3. **Data Loader**: partitions/repartitions datasets across nodes.
4. **SMS Planner (SQL → MapReduce → SQL)**: extends Hive to push computation into DBMS when possible.

Example: Hive normally does Reduce-heavy plans; SMS planner pushes filters, joins, and aggregates into PostgreSQL.

Current support: selection, projection, aggregation. Future: joins and richer operators.

## Benchmarks

Setup: EC2 clusters (10, 50, 100 nodes). Compare Hadoop, HadoopDB (Postgres), Vertica, DBMS-X.

Tasks:
1. **Grep** (unstructured scan): HadoopDB ≈ Hadoop, both worse than DBMSs (compression advantage).
2. **Selection** (structured): HadoopDB + DBMSs faster (use indexes). Hadoop lags.
3. **Aggregation** (group-by): DBMSs best (column-store wins), HadoopDB better than Hadoop. Hive can outperform hand-coded Hadoop with hash aggregation.
4. **Join**: HadoopDB, Vertica, DBMS-X leverage indexes; Hadoop lags.
5. **UDF Aggregation** (text link counts): Hadoop best (native text handling); HadoopDB competitive; DBMSs weakest (poor UDF/text handling).

Summary: HadoopDB consistently outperforms Hadoop, approaches DBMS performance.

## Fault Tolerance & Heterogeneous Environments

Experiment: simulated node failures/stragglers.

Hadoop & HadoopDB: resilient (redistribute/re-execute tasks).

Vertica: large slowdowns (query restart, straggler bottlenecks).

HadoopDB slightly outperforms Hadoop in failure scenarios (pushdown reduces data transfer).

Slightly worse than Hadoop in heterogeneity (scheduler sometimes queries straggler DB instead of replica).

## Discussion

HadoopDB’s fault tolerance benefits stem from Hadoop layer.

Its performance gap vs. DBMS mainly due to PostgreSQL (row-store, no compression) and integration overhead.

Strong potential to support shared-nothing DBMS on commodity clusters.

Could generalize: HadoopDB approach might turn any single-node DBMS into a parallel DBMS.

## Conclusion

HadoopDB unifies the strengths of MapReduce and DBMS:

Scalability + fault tolerance of Hadoop.

Performance + SQL interface of DBMS.

Experiments show it clearly beats Hadoop, and comes close to parallel DBMS performance.

Open-source design ensures low cost and accessibility.

Paves way for petabyte-scale analytical systems in the cloud.

## Strengths

Innovative hybrid design: first true system-level combination of Hadoop + DBMS.

Practical implementation: PostgreSQL + Hadoop + Hive integration.

**Strong benchmarking**: across structured, unstructured, and mixed workloads.

Fault tolerance & heterogeneity resilience: inherits Hadoop’s strengths.

Open-source & cloud-friendly: democratized large-scale analytics.

## Weaknesses

Performance still behind commercial DBMSs (Postgres row-store bottleneck, no compression).

Limited SQL pushdown support (joins not fully supported at time of paper).

Data loading overhead: 10× slower than Hadoop due to repartitioning and indexing.

Complex architecture: coordination overhead between Hadoop and DB nodes.

Benchmarks limited: no real-world enterprise workloads beyond synthetic/log-based tasks.

# New Query Optimization Techniques in the Spark Engine of Azure Synapse

## Abstract

Big data queries are dominated by **stateful operators**: *exchange* (data shuffling), *hash aggregate*, and *sort*.

Paper introduces three **sets of optimizations** in Azure Synapse’s Spark engine:

* Exchange placement algorithm – reduces number and cost of exchanges while maximizing reuse.
* Partial push-down optimizations – push partial computations below expensive operators (aggregate, semi-join, intersection).
* Peephole optimizations – specialize sort and other operators for specific input characteristics.

Results: 1.8× speedup over Apache Spark 3.0.1 on TPC-DS.

## Introduction

Modern query compilers (like Spark) use multiple stages for parallelism, separated by exchanges (shuffles).

Exchanges and stateful operators dominate runtime costs.

Focus: new techniques to reduce costs of exchange, aggregate, and sort.

Main contributions:

* New **exchange placement** algorithm.
* Extensions of **partial push-down** techniques for scale-out big data.
* **Peephole optimizations** for operator specialization.

## Partial Push-Downs
Idea: push partial computation (pre-aggregation, filtering, deduplication) closer to data sources.

Three optimizations:

* **Partial aggregate push-down** – compute local aggregates before shuffle.
* **Partial semi-join push-down** – deduplicate smaller join inputs, convert inner joins to semi-joins when possible.
* **Partial intersection push-down** – prune inputs early in set operations.

Key novelty: more aggressive rules than prior work, and specialized cost model for distributed settings.

Findings: some push-downs only help in scale-out systems (savings from reduced shuffles).

## Exchange Placement

Exchanges shuffle data between stages to satisfy partitioning.

Problem: Spark uses local greedy placement → suboptimal, too many shuffles.

Existing system Scope (Microsoft) uses cost-based exploration, but too slow.

Synapse Spark’s approach:
* Cost-based exploration under a strict time budget.
* Explore alternatives only when overlap or reuse is possible.
* Introduce multi-consumer exchanges (reuse shuffle outputs across consumers).
* Use plan-marking to detect and enforce reuse early.

Achieves fewer exchanges, better overlap, and more reuse.

## Partial Aggregation & Semi-Join Push-Downs

**Partial aggregation**: push aggregation below joins and intersections where safe.

Semi-join push-down:
* Deduplicate small input keys early.
* Convert unnecessary inner joins into semi-joins.

Both reduce data exchanged between stages.

Specialized costing mechanism ensures only beneficial push-downs are applied.

## Bit-Vector Filtering

Implements distributed Bloom filter construction for semi-joins.

Constructed incrementally at task, executor, and orchestrator levels.

Uses plan marking to avoid redundant filter construction.

Shared across multiple consumers to save cost.

## Peephole Optimizations

Focus: optimize sort, one of Spark’s most expensive operators.

Spark uses TimSort with prefix-based comparison.

Two optimizations:
1. **Sort key reordering** – place high-cardinality keys first to reduce collisions and comparisons.
2. **Two-level sort** – when first key has low cardinality, sort first by prefix, then refine within partitions.

Saves up to 10× comparisons in TPC-DS queries.

## Evaluation

Benchmarked Synapse Spark vs Apache Spark 3.0.1 on TPC-DS (1TB).

Results:
* Overall 1.8× speedup.
* Exchange placement → 27% faster.
* Partial push-downs → 40% faster.
* Peephole optimizations → rest of gains.

Per-query breakdown: most queries benefit, especially shuffle-heavy and sort-heavy queries.

Semi-join push-down effective only in scale-out settings.

## Related Work

Exchange placement: Scope, Volcano, and others – but too costly for big data.

Partial aggregation: studied in relational DBMSs, but less aggressive and not tuned for scale-out.

Bloom filters: prior systems use them, but this paper innovates in distributed construction + reuse.

Sort optimizations: DB literature has sort-key optimizations, adapted here for Spark runtime.

## Conclusion

Introduces three families of optimizations in Synapse Spark:
1. Exchange placement (cost-based, multi-consumer aware).
2. Partial push-downs (aggregate, semi-join, intersection).
3. Peephole optimizations (sort).

Together yield 1.8× faster performance on TPC-DS.

Many techniques are generalizable to other big-data engines, not just Spark.

## Strengths

Holistic optimization: tackles all three costly operators (exchange, aggregate, sort).

Novel exchange placement algorithm: balances cost and optimizer time budget.

Aggressive partial push-downs beyond traditional DBMS approaches.

Practical impact: implemented in production (Azure Synapse).

Strong evaluation: TPC-DS benchmark with detailed per-query results.

Generality: some optimizations applicable beyond Spark.

## Weaknesses

Limited scope: only three operator types targeted; other bottlenecks (joins, UDFs, I/O) less addressed.

Partial push-down applicability: only beneficial in distributed settings, not in scale-up DBMS.

Optimizer complexity: cost-based exploration with plan marking adds optimizer overhead.

Evaluation limited: mostly TPC-DS synthetic workload; few real-world workloads shown.

## Relevance & Context

Context: Spark dominates big-data analytics, but shuffle, sort, and aggregation remain bottlenecks.

Relevance: Synapse Spark improves over open-source Spark, delivering production-level optimizations.

Industry impact: Available to all Azure Synapse users; highlights cloud vendors’ push to optimize open-source engines.

Research contribution: Extends classical DBMS optimization techniques (partial aggregation, Bloom filters, sort-key ordering) into modern scale-out distributed systems.

Future importance: These techniques pave the way for integrating cost-based, fine-grained optimization into cloud-native data platforms.