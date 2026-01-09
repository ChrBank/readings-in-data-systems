# Query Optimization 

## Life of SQL query
![alt text](images/image.png)

> Query optimization is in the **generate logical plan** and **generate physical plan**

### Example of query optimization
![alt text](images/image-1.png)

![alt text](images/image-2.png)
The math symbol are part of an algebra expression.

![alt text](images/image-3.png)
> When we talk about a **query plan** it is usually a tree structure.
* This is not effective, we want to do *selection* first and then *join*. Like these following alternative plans which we call the logical plans:

![alt text](images/image-4.png)
![alt text](images/image-5.png)

> Just like the physical plan we can also have multiple physical plan
![alt text](images/image-6.png)
![alt text](images/image-7.png)
![alt text](images/image-8.png)
> Index scan is used for range scan for multiple IDs.
> Index lookup is used on a *single* ID.

## Query optimizer
Query optimizer is responsible for finding a good query plan
1. **enumerates** alternative **logical** & **physical** plans
2. computes **estimated cost** of each plan
3. picks the plan with the **lowest cost**
> … also called *cost-based optimization*

### This is usually how we do query optimization
![alt text](images/image-9.png)
* When generating plans a lot of plans are generated. 
* Therefore, the cost is used to *prun plans* when we do query optimizaion.
* The **catalog** is where we store the best plan we find.

> It is **not** about finding the best plan! 
> It is about **avoiding** worst plans & **finding** a good plan!

### What do we have in a catalog?
![alt text](images/image-10.png)
* **Table metadata**
  * schema, indexes, location of the corresponding database files.
* **Statistics about the tables**
  * *per table*: #tuples & #pages
  * *per index*: #distinct keys (#keys), min/max key values, #index pages
  * *per tree index*: index height
  * statistics in catalogs are updated periodically -> may not be precise
  * query optimization isn’t precise either, so this is OK
* **Other information about tables**
  * e.g., histograms of the values in some field for knowing data distribution are sometimes stored

### Qptimize query - Before cost estimation…
![alt text](images/image-20.png)
#### Step 1 - decomposition of a query
![alt text](images/image-11.png)
![alt text](images/image-12.png)
![alt text](images/image-13.png)

break query into **blocks**
a block has **exactly one** SELECT, FROM
            **at most one** WHERE, GROUP BY, …
if you have multiple nested queries -> find nested and outer block for each

We can also transform a query to another form. Since we don’t have to decompose each nested-query, some can be flattened
![alt text](images/image-14.png)

#### Step 2 - convert each block to relational algebra
![alt text](images/image-15.png)
![alt text](images/image-16.png)
![alt text](images/image-17.png)

**Alternative plans**
We can push down *selection* queries
![alt text](images/image-18.png)

We can also push down *projection*
![alt text](images/image-19.png)

#### Core of a query plan
* SELECT-PROJECT-JOIN
  * not much room for other equivalent plans beyond these operations
  * others build on this core
  * group by (either sort or hash-based)
  * aggregation (done after grouping)
  * order by (based on sorting)
  * … 
> **logical plan optimization** focuses on optimizing SELECT-PROJECT-JOIN core
> ![alt text](images/image-23.png)

### Estimate plan cost
![alt text](images/image-21.png)

#### Step 1 - must estimate cost of each operation in the plan tree
Depends on input **cardinalities** for each operation
**Goal**: Determine the less costly access method & operator algorithm
* file scan vs. index lookup
* nested-loop vs. sort-merge vs. hash join 

> **Cardinalities**: the number of elements in a set or other grouping, as a property of that grouping.

Cost depends on
* IO (= disk accesses) + computation
* IO typically dominates

![alt text](images/image-22.png)

#### Step 2 - must estimate result size of each operation
D epends on **output** size for each operation
* use information about input cardinalities
* use information from the catalog
* estimate size of intermediate results

#### Result size estimation
query block template:
![alt text](images/image-24.png)

Maximum #tuples in the result
* product of the cardinalities of the relations in relation-list 

Reduction factor (RF) = selectivity
* associated with conditions & its impact on reducing the result size

> assuming uniform distribution & independent conditions
> result size = max#tuples x product of all RFs

#### Selections - Result size estimation 
Based on the condition the reduction factor is …
* $column_i$ = value 
  * if there is index -> $1 / \#keys$
* $column_i = column_j$
  * if there are indexes for both -> $1 / MAX(\#keys for I, \#keys for J)$
  * if there is index only for I -> $1 / MAX(\#keys for I)$
* $column_i < value$
  * if there is index -> $max - value / max - min$
* $column_i IN [list of values]$
  * if there is index -> $(1 / \#keys) x \# items in the list$

#### Join - Result size estimation
Based on the join condition between R and S relations.
Typically the join condition is on a common column.
Let’s assume there is an index on this column 
* #keys for S > #keys for R 
  * assume S has the superset of values -> each R finds a matching S 
  * $result size estimation = \#tuples(R) x \#tuples(S) / \#keys(S)$
* #keys for R > #keys for S
  * symmetric result
  * $result size estimation = \#tuples(S) x \#tuples(R) / \#keys(R)$


#### So far considering uniform data
This can lead to inaccurate estimations
![alt text](images/image-25.png)

For more robust reduction factors
![alt text](images/image-26.png)

### Generate plan - Enumeration of query plans
![alt text](images/image-27.png)

#### Single-relation plans
> Plans with SELECTIONS & PROJECTIONS 
> * No JOINS

Consider possible access paths:
1. file scan (heap or sorted)
2. index lookup/scan
> (based on available indexes)
> Pick the access path with the least estimated cost

![alt text](images/image-29.png)

#### Multi-relation plans – brute-force approach
(1) **enumerate** all possible **join orders** for relations
(2) **enumerate** all **join algorithms**, select the one with minimal cost
(3) **enumerate** all **access methods** for each input relation, select the one with minimal cost

> query optimization must not take longer than query execution!
> must prune search space!

#### Multi-relation plans – pruning join orders
Consider only left-deep plans (originally System R method)
> At times when they found this, this was the best plan, and today this is almost also the case, even with better hardware.
![alt text](images/image-30.png)

![alt text](images/image-31.png)

#### Multi-relation plans – join algorithm choices
![alt text](images/image-32.png)
![alt text](images/image-33.png)

> observe common sub-plans! cost estimation calculations are done with **dynamic programming**

#### Multi-relation plans – access paths
![alt text](images/image-34.png)

### Multi-relation query planning
Choice 1: **Bottom-up** plan enumeration (Generate approach)
* Start with nothing and then build up the plan to get to the outcome that you want 

Choice 2: **Top-down** plan enumeration (Transform approach)
* Start with the outcome that you want, and then work down the tree to find the optimal plan that gets you to that goal

#### Bottom-up plan enumeration
Use static rules to perform initial optimization. Then use **dynamic programming** to determine the best join order for tables using a divide-and-conquer search method
> Examples: IBM System R, DB2, MySQL, DuckDB, PostgreSQ

**Example**
Break query up into blocks and generate the logical operators for each block.

For each logical operator, generate a set of physical operators that implement it.
* All combinations of join algorithms and access paths

Then iteratively construct a "left-deep" join tree that minimizes the estimated amount of work to execute the plan.

We want to join three tables

![alt text](images/image-35.png)

We do to joins for two pairs while one is leftout. This results in an intermediary result:

![alt text](images/image-37.png)

Then we have to join the last leftout table with the intermediary result:

![alt text](images/image-36.png)

Finally we can estimate which was the fastest:
![alt text](images/image-38.png)

> Result: **We find the best order of how to do the join**.

#### Top-down plan enumeration
Start with a logical plan of what we want the query to be. 
Perform a branch-and-bound search to traverse the plan tree by converting logical operators into physical operators.
* Keep track of global best plan during search.
* Treat physical properties of data as first-class entities during planning.

> Examples: SQL Server, Greenplum, CockroachDB, Apache Calcite

##### Example
![alt text](images/image-39.png)

If we se an ORDER BY, we know we have to do some sorting, hence using sort merge join (SM_join) is better than hash join.
![alt text](images/image-40.png)

Further considerations
* Use **declarative rules** to generate transformations.
* Use **Memo table** (hash table) to keep already computed costs. 

Search **termination**:
* **Transformation Exhaustion**: Stop when there are no more ways to transform the plan.
* **Wall-clock Time**: Stop after the optimizer runs for some length of time
* **Transformation Count**: Stop after a certain number of transformations have been considered
*** Cost Threshold**: Stop when the optimizer finds a plan that has a lower cost than some threshold

## Query optimizer architecture
![alt text](images/image-41.png)

* It will take an **input** in the form of an logical plan (tree structure)
* It will **output** a physical/execution plan (tree structure)

**Size distribution estimator** is statistics based on the cardinaility which is used for the **cost model**.

## Alternatives to cost-based optimization
**Rule-based optimization**
* no calculations, quicker
* e.g., if there is an index, always use it
* Spark’s optimizer (before Spark 2.2)
**Randomized plan generation**
* randomly picks a plan as the name suggests
**Multi-query optimization**
* considers multiple queries together
* allows work-sharing
**Learning-based** query optimization (NEW)

> **Cost-based optimization** is the most common (so far..)

## Summary
* Queries are first **rewritten** where possible.
* Then, queries are first broken into **blocks**.
* Blocks are converted to **relational algebra** expressions.
* Equivalence transformations are used to **push down** selections and projections.
* **Cost estimations** are done based on metadata, statistics, and histograms kept in catalog.
* **Bottom-up** vs **top-down** plan enumeration
* Search space is **pruning** is important (eg., left-deep trees only, no cartesian products)