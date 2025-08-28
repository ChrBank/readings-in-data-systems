# Discussion

Why 2007 when it was published?
* Marked changes and new emerging applications
* Hardware changed
* More memory (DRAM) available
* Analytics become more important
* 2005: multicores
* Problem at that time, we used this for processing information at warehouses. 

Why redesign?
* Assumptions for how database should be designed are changed
* New hardware are unfit for traditional databases.
* Duo to the emering of new specialised application the "one size fits all" does not fit different apps and optimizations needs.
* **Scalability**: With respect to #threads/cores and #users/data size. The single core limitations were different, hence we had little concurrency problems.
* Database in that time used a shared everything design, this does not scale well, because we did not have the correct concurrency control of shared data. Now we want to use grid based solutions.
* When your data fits in memory, many database components become a bottleneck.
  * Problems with logging, buffer manager (moving data).
* DBA were cheaper than hardware, now it is opposite.

New design?
* Remove redo log (helps with recovery) => replication, since data is already in memory
* Minimal undo log (helps with aborts)  
  * Without the need for redo, you can log less info
* Rather than data copy, redo the actions comoute is cheap
* Remove locking by strict partitioning, each note writes to their own partition.
  * => Shared nothing design
* Simplify concurrency control
  * Optimistic

Challenges?
* Not everything is partitionable 
  * Like cross partitional transactions.
* Load balance => even workloads.
* DB should fit in memory
* No knobs design
* Stored procedures, remove the need for optimization queries or ad-hoc transactions.

Assumptions
* Good partitioning relativley simple
* fast networks
* Easy knops
* Data fits in memory
* Memory is cheap => but still more expensive than SSD or harddisk.
* 