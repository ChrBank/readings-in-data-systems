# Online Analytical Processing (OLAP) Systems

Last lecture:
* OTLP build great interactive applicaitons
* "Small" data

Today lecture:
* OLAP processing
* **Big data**
* Keep track of the history, interactions, low-level, media, user input, and logs.
* How do we analyse large amount of data this?

## Motivation
Big data characteristics and the five V's:
* **Volume**
  * Amount of data is stored and processed
  * Lot of data is generated every minute
* **Velocity**
  * The speed at which data is generated, collected, and processed.
* **Variety**
  * Different types of data that need to be stored
* **Veraticy**
  * The accuracu and trustworthiness of data.
* **Value**
  * The ability to turn data into insights for decision making and creating businees value.
  * Big data is only beneficial if it delivers insights.

With this lecture we focus on data volume.

## OLAP
Online analytical processing (OLAP)

* Scanning over large datasets.
* Complex queries that read a lot of data.
* Focus on querying and reporting
* Typically involves data aggregations and multi dimensional analysus

> **Online** = system gives answer immediately


Example of OLAP:
* How many products did each customer buy
* Cumulative sum of sales per user over time


### OLTP:
* Focus on real-time data processing
* High volumne of short transactions
* Typically involeves frquent insert, update, and delete operations.

Example of OLTP in Analog APP:
* Price of a tea
* Price of fancy product
* Update user DONNA.

### OLAP vs OLTP
* DB for OLTP: running applications and "online" workloads
* DB for OLAP: dooing "offline" analysis, reporting, ML

Because of different storage models, no system that fits all.

**State of the art:**
Online application --> OTLP database (like oracle) --> ETL (Extract transform load) --> OLAP database (like snowflake) --> offline analysis reporting.

**Database workloads**:
* OLTP: Focuses on writes and simple operations
* OLAP: Focuses on reads and complex operations
* HTAP: Hybrid of both OLTP and OLAP.

### Data storage
DBMS's storage model defines how it organizes tuples on disk and in memory.

Data in files by tuples, one tuple after the next
* Header contains metadata like number of tuples, start byte of first tuple, checksums, and more.
* Tuples can be of different lengths.

**N-ary storage model (NSM)**:
* Store all attiubutes of tuple in a single file, tuple by tuple.
* Known as "row store".
NSM with OLTP is usefull, however NSM with OLAP is not good since you can endup reading lots of useless data.

Summary;
* Advantage:
  * Good for OLTP, fast updates and deletes
  * Good for memory
* Disadvantage:
  * Bad for OLAP, large useless scans
  * Bad for memory.

**DSM**
Data Server Manager (DSM)
* Store data in files organized differently by sections,
  * Also known as "column store". 
* This works well with OLAP systems since we only reads certain columns/sections in a file --> read much less data.
* Hence we have better cache locality when working on a single attribute.
* Better data compressions.

However:
* For OLTP this is bad, since you know need to read from multiple pages or files + tuple reconstrcution costs.
* If you insert, you have to update multiple columns hence you will access a lot of different sections. 

### Systems in the wild:
Row stroe database:
* MariaDB
* PostgreSQL
* SQLite

Column store databse:
* SAP Hana

Newer OLAP systems:
* DuckDB
* Snowflake

### Partition attributes across (PAX)
A hybird of NSM and DSM

* Better I/O than NSM
* Better tuple reconstruction than pure DSM, because attributes are closer toghetset, e.g. on the same memory page.

PAX works by storing data in "Row groups" with NSM.
In DSM we store "tuple storage per row group".
* Everything is organized in one file
  * You could also split them up in multiple files.

DuckDB only has 1 database file.
Snowflase has multiple files and has one row group per file.

## Snowflake data management
Snowflake is a big OLAP database system.

### Example

Users.csv have a table of users IDs and usernames.

Example: we store only userIDs and usernames.
* How is this data ingested in Snowflake.
* We copy into users from @stage.


Snowflake query queiry engine reads CSV and prdouces data files in PAX format and metadata.

Snowflake will store the data in two files:
* Half are stored in one pax file, and the other half in another pax file.
* What snowflake will do is collecting metadata
  * What was added
  * What was deleted
  * Min/max
    * ID, 1-3, 4-6
    * NAME: D-V, E-Z 
  * This is usefull for pruning

Now we insert a new user (7, "Kasper")
* Snowflake query engine produces data file will in PAX format and metadata.
* Files are immutable, this helps cache always being up to date.
* In the metadata file, the new user4.data file will be added under the added section.

Now we delete a user (name = "Zoi")
* Snowflake query engine ccopies remaining data form user2.data into a new file and produces metadata.
* This will be marked in the metadata file as user2.data inside the deleted section. 

Now what we can do is use query optimization via Pruning.
* Because Snowflake creates a lot of metadata files we can use pruning if we want to do a scan if ID=5 for example. Without this predicate we cannot do pruning.

### Efficient query processing
SOrting 
* Archieves best query performance (best pruning) for given sort key
* Sorting a tible is expensive
* May block concurrent workloads or starve trying to commit

Clustering:
* Maintains an approximante of sorting (90% sorted is fine)

Snowflake will look at different clusters (how much files overlap)
