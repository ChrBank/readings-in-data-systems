# Data stalls in DNN training

Epoch: whole run of a network
Batch: run of a small part of the dataset

What kind of data is used in DNN:
* Audio
* Images
* Texts

Training pipeline:
1. A minibatch of data items is feteched from stroage
2. The data items are pre-processed, for e.g.,, for image classification, data items are decompressed, and then randomly cropped,resized, and flipped.
3. The minibatch is then processed at the GPU to obtain the model’s prediction
4. A loss function is used to determine how much the prediction deviates from the right answer
5. Model weights are updated using computed gradients


Paper focus on preprocessing:
* From IO device -> preprocessing queue -> preprocessing.

DS (data stall)-analyzer
* PyTorch, DALI anallysis is naive
* Better measurements needed
* Measure ingestion rate
  * A measure GPU speed (maximum GPU capacity)
* Measure prep stall

What issues exist with this pipeline?
Is remte storage a bottleneck fro training:
* Solution: Download all dataset at once is better vs download it part by part.

Data too big too fit inside a cache:
* Fetch rate slower than GPU compute -> fetch stalls occur.

When does data prep at the CPU becomes a bottleneck for DNN training?
* Entire dataset fits in cache
* Operations on the CPU is much slower than GPU

Mitigating data stalls:
* MinIO cache
  * OS cahing does not work for DNNs 
  * You go through many epochs not just one.
  * **Never remove items from cache**
    * There is a lot of redundant reads, hence we can reuse the data for next epochs.
  * Implemnted in user space (no sudo needed)

* Partitoned MinIO caching
  * Cross ndoe bandwidth > SSD to node bandwidth

* Coordinated prep
  * HP (hyper parameter) search
  * Redundant data accesses by different HP-search runs
  * Compute batch once
  * Reuse the batch

Changes on top of DALI
* DALI + (MinIO cahce + Partitionned MinIO cache) + coordinated prep

MinIO cache:
* Impact: DNN-aware caching to minimize IO by reducing cache misses per epoch
* Benefits: Single server training

Partitioned MinIO cache
* Impact: Eliminate redundant fetch by coordinating remote MinIO caches
* Benfits: Distributed training

Coordinated Prep
* Impact: Eliminates redundant fetch and prep across jobs
* Benefits: Single server training

Results:
* Speed ups (12 hours vs 50 hours)
* Less disk usage


> They also bypass the OS by implementing their own cahing.


# tf.data service: A Case for Disaggregating ML Input Data Processing
A service that helps with preprocessing tasks. It helps to utilize better CPU and RAM resources to reduce data stalls.

Motivation for disaggregation: 
* More resource (CPU) efficient utilization of acceleratoors.
* Helps with horizontal scale-out of pre-processing

* Disaggregating the tasks for pre-processing. 


In some cases we see that the CPU and memory utilization go up and down. 

Sometimes having 60 cores, some cores might not be used. Hence we can use them to do some pre-proccessing instead. 

**Architecture and workflow for disaggregation**
Goals:
1. Eliminatet input data stalls
2. Avoid redundant data preprocessing
3. Simplify the design by relaxing constraints.
   1. Some datasets are large enough that training on all data may not be practical. Hence we do not need to read all data.

Implementation:
* Orchestrator
  * Resource management (CPU)
* Client (GPUs) 
* Dispatcher + Workers (CPUs)
  * tf.data service

**Emphemeral Data sharing**
* Each client has an arrow to the next batch for each worker (CPU preparring the data), so no data stalls occur.

**Coordinated reads**
* Instead of havving workers randomly making batches, workers now will no create batches of same sizes. 
* This helps with different consumers to not get out of line with each other -> helps reduce stalls.