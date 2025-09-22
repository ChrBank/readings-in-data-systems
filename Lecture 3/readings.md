# How Good Are Modern Spatial Analytics Systems?

## Context of the Topic
Spatial data has exploded in recent years due to smartphones, cars, sensors, satellites, and social media platforms that constantly generate geotagged information. Companies like Uber, Lyft, Foursquare, and Twitter create massive volumes of location data daily. Analyzing such “big spatial data” enables better services, predictions, and insights—for example, optimizing city transportation, studying climate change impacts, or predicting disaster aid needs.

This surge has motivated the development of **modern spatial analytics systems capable** of handling complex spatial queries at scale. While big data frameworks like Hadoop and Spark existed, their spatial support was initially limited, leading to specialized systems like HadoopGIS, SpatialHadoop, GeoSpark, Simba, Magellan, and LocationSpark. The paper evaluates and compares these systems.

## Context of Motivation
Previous studies compared some spatial systems but were **incomplete** (limited queries, missing data types, lack of scalability studies, or outdated benchmarks). Moreover, systems like GeoSpark and Magellan have evolved significantly, introducing new features and optimizations not yet studied.

The authors aim to fill this gap by performing a **comprehensive comparison** of Spark-based spatial systems (SpatialSpark, GeoSpark, Simba, Magellan, LocationSpark) across multiple query types, datatypes, and datasets, focusing on performance, scalability, and memory costs.

Key open questions motivating the study:
* How do modern systems perform for all major features?
* How do they handle all combinations of spatial joins?
* What are the memory costs and their effect on performance?
* How do they scale with data size and cluster size?

## 1. Introduction
* Spatial data is generated massively from devices and apps (Uber, Twitter, Foursquare, etc.).

* This data has high analytical value (transport optimization, disaster response, business intelligence).

* Traditional big data tools (Hadoop, Spark) **lacked native spatial support**, leading to specialized systems.

* The paper contributes by:
  * Surveying modern spatial analytics systems.
  * Evaluating them using **real-world datasets**.

## 2. Motivation
* Focuses on **Spark-based systems** (SpatialSpark, GeoSpark, Simba, Magellan, LocationSpark).
  * Hadoop-based systems excluded (they perform poorly compared to Spark).

* *Prior comparisons were partial, outdated*, or missing scalability studies.

* Systems have since evolved (e.g., GeoSpark added datatypes + optimizer, Magellan adopted by FOSS4G).

* **Missing evaluations**:
  * Performance across all features and joins.
  * Effects of query selectivity and memory.
  * Scalability with more nodes.
* **Goal**: provide a comprehensive benchmark and reusable setup for future studies.

## 3. Queries
* Evaluated four datatypes: **points**, **linestrings**, **rectangles**, **polygons**.

* Query types considered:
  * **Range queries** – return all objects within a region.
  * **kNN queries** – find k nearest neighbors to a point.
  * **Spatial joins** – combine datasets based on spatial predicates (e.g., overlap, intersect, within).
  * **Distance joins** – special case of joins with distance predicate.
  * **kNN joins** – return k nearest neighbors across datasets.

* Other queries (computational geometry, raster operations) excluded as not supported by studied systems.

## 4. Spatial Analytics Systems
Survey of systems:
* **Hadoop-GIS**: First Hadoop spatial system, grid-based partitioning, supports range + spatial joins.
* **SpatialHadoop**: Native Hadoop extension, supports R-tree indexing, range/kNN/joins.
* **SpatialSpark**: Lightweight Spark system, supports multiple geometries, range, joins, distance joins (but no kNN join).
* **GeoSpark**: Spark-based, supports many datatypes + query optimizer, RDD-based, wide partitioning/indexing schemes, supports range, kNN, joins.
* **Magellan**: SparkSQL extension, supports many geometries, Z-order indexing, range + joins (but not kNN/distance joins).
* **Simba**: Spark SQL-based, supports range, kNN, distance joins, and kNN joins (points only).
* **LocationSpark**: Spark-based, advanced query scheduler, supports range, kNN, joins, kNN joins (points + rectangles).

## 5. Experimental Setup
* Clusters deployed on **Amazon AWS EMR** with varying nodes (1–16).
* Systems tested on compatible Spark versions.
* **Datasets**: OpenStreetMap (points, roads/linestrings, buildings/polygons, rectangles). Subsets used due to size.
* Preprocessing done with Java Topology Suite (to clean geometries).
* Additional experiments run with US Census TIGER dataset.
* Spark memory tuned (RDD caching strategies differ across systems).
* **Performance metrics**: throughput, join/prep time, shuffle costs, memory, scalability.

## 6. Evaluation
* **Memory Costs**: Systems consume ~3× disk size; partitioning overhead due to replication. Magellan’s LineString index very costly.
* **Range Queries**: GeoSpark and LocationSpark dominate; Magellan weakest (scans all partitions). LocationSpark excels at low selectivity due to bloom filter (sFilter).
* **kNN Queries**: GeoSpark and Simba stable; LocationSpark fluctuates (due to sFilter). Simba uses serialized index (slower).
* **Distance Joins**: Only Simba, GeoSpark, SpatialSpark support; GeoSpark best due to reusable partition/index.
* **Spatial Joins**: GeoSpark generally best; Magellan high shuffle costs; LocationSpark strong for point-rectangle joins.
* **kNN Joins**: Only Simba & LocationSpark; Simba fails on large datasets (due to duplicates); LocationSpark more efficient and scalable.

## 7. Conclusion
* GeoSpark generally best overall, with strong performance across most queries.
* LocationSpark outperforms in specialized cases (point-rectangle joins, kNN joins).
* Simba good but limited (crashes with large datasets).
* Magellan weakest, with high shuffle and limited query support.
* SpatialSpark less feature-rich, high memory use.
* Overall, modern systems perform well but trade-offs exist in query support, memory efficiency, and scalability.

# Comparison of Approximations of Complex Objects Used for Approximation-based Query Processing in Spatial Database Systems

## Context of the Topic

Spatial database systems (SDBS) must efficiently manage and query huge collections of geometric objects (points, lines, polygons). Since these objects are often irregular and complex, directly storing and querying them is inefficient.

To address this, approximations are used: simplified geometric shapes (like bounding boxes, circles, ellipses, convex hulls, etc.) that capture the essential position and extent of objects. Approximations make queries faster by filtering candidate results before applying costly exact geometry checks.

Traditionally, the **minimum bounding box (MBB)** is the most popular approximation. However, it is often too coarse and produces many false hits. This paper investigates alternative approximations to improve accuracy and efficiency in query processing.

## Context of Motivation

The **main bottleneck** in spatial query processing is efficiency. Complex geometric operations are CPU-intensive and unsuitable for very large datasets.

Approximation-based query processing (filter + refinement) depends heavily on the **quality of the approximation**: better approximations reduce false hits and lower refinement costs.

The MBB is simple and efficient but often too inaccurate. Other shapes like convex hulls, rotated bounding boxes, ellipses, and n-corners may provide much higher approximation quality while still being manageable.

The challenge: Can non-rectangular approximations be effectively integrated into access methods (like the R*-tree) that were originally designed for bounding boxes?

The paper aims to answer:
* Which types of approximations are most suitable for query processing?
* How well can non-rectangular approximations be managed in existing spatial access methods?

## 1. Introduction

GIS handle millions of spatial objects stored on secondary storage.

Objects are complex polygons with holes → too irregular for direct indexing.

Approximations (geometric keys) make indexing possible.

The **MBB** is widely used, but rough and inaccurate.

Goal: explore better approximations that balance **simplicity** (fast filtering) and **quality** (few false hits).

## 2. Approximation-based Query Processing

### 2.1 Two-step query processing
* Queries are decomposed into basic types (point, window, region, enclosure, containment, nearest-neighbor, spatial join).
* Processing = **filter step** (use approximations to find candidates) + **refinement step** (check exact geometry).
* Criteria:
  * *Simplicity*: approximations must allow fast filtering.
  * *Quality*: approximations must reduce false hits.
  * *Construction cost*: acceptable since only needed on insert/update.

### 2.2 Quality of approximations
* Defined via false area (difference between approximation and original).
* Introduces **approximation quality metric (GAppr)** = ratio of true object area vs. error area.

### 2.3 Classification
* **Conservative**: contains the object (e.g., MBB, convex hull).
* **Progressive**: is contained inside the object.
* **Generalizing**: simplifies shape (no containment guarantee).
* Only conservative and progressive are useful for queries.

### 2.4 Six selected convex conservative approximations
![alt](images/image-1.png)
* MBB (aligned rectangle).
* RMBB (rotated rectangle).
* MBC (circle).
* MBE (ellipse).
* Convex hull (CH).
* Minimum bounding n-corner (n-C).

## 3. Empirical Comparison of Approximations
Used real-world maps (Europe, Baden-Württemberg, Lakes & Islands, Africa).

Evaluated approximation quality and storage requirements.

* Findings:
  * Quality ranking: Convex Hull > 5-Corner > 4-Corner > RMBB > Ellipse > MBB > Circle.
  * Convex hull best, but variable/high storage cost.
  * RMBB improves MBB significantly with only 1 extra parameter.
  * 5-Corner nearly matches convex hull with fixed, modest storage.
  * Higher quality directly reduces false hits, lowering refinement costs.
  * Gains especially strong for expensive queries like spatial joins (quadratic benefit).

## 4. Approximations Stored in Spatial Access Methods (SAMs)

### 4.1 Spatial Access Methods
* SAMs (e.g., R-tree, R*-tree) usually use MBBs.
* Alternatives (sphere tree, polyhedra-tree, etc.) are too complex and costly.
* Strategy: store complex approximations inside SAMs designed for MBBs.

### 4.2 Organization in R-tree*
* Tested R*-tree with 33,204 polygons (European counties).
* Queries: 10,000 point queries distributed realistically.
* Considered storage requirements, area extension, block accesses.

* Findings:
  * More complex approximations → higher storage + larger bounding regions → more block accesses.
  * BUT: higher approximation quality more than compensates the overhead.
  * Circle worst (poor quality, higher costs).
  * 5-Corner, RMBB, and Ellipse best trade-offs (better accuracy, acceptable cost).
  * Convex hull great quality but too costly in storage.

## 5. Conclusions
* Effective approximations must be simple and high-quality.
* Ranking: Convex Hull > 5-Corner > RMBB ≈ Ellipse > MBB > Circle.
* Best practical choices:
  * 5-Corner → highest quality/cost ratio.
  * RMBB & Ellipse → balanced between storage and accuracy.
* Using approximations in R*-trees works well: directory still uses MBBs, approximations only in data blocks.
* Fewer false hits → significant query time reduction, especially for joins.
* Future directions: compressed/multi-container approximations, spatial joins, extension to 3D (ellipsoids, rotated boxes).