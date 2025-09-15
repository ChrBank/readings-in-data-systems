# Paper 1 - Pruning in Snowflake: Working Smarter, Not Harder

# Paper 2 - AutoComp: Automated Data Compaction for Log-Structured Tables in Data Lakes
Since when having immutable files, when we append new rows it creates a new file.

> **Goal**: Many small files --> compact to one big file.

Having lots of small files --> This results in poor query performance, high storage costs, metadata bloat, and system bottlenecks.

We want all these small files to be in one file. By automatically compact these files into one file.

With how we organize datalakes better, we can apply different database engines on this data.

By compacting these small files into a one pax file, we can apply min/max, because pax files contains metadata.



