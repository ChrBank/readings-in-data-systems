## Disaggregated storage
* Split computation and storage into two layers

* Computation layer makes computation
* Storage layer has multiple storage devices. 
  * Multiple machines shares storage, which results into more data shared accross the network and more data movement
  * More network latency, however you have much more storage.

**On demand instance**: 
* Fixed price and guaranteed service
**Spot instance**: 
* Cheaper price, but not guaranteed service, you get one if there is one available.
* Trade off, if the workload istn't latency sensitive and predicatability isn't as important.

## What changes for the cloud? Storage hierarchy
Conventional storage hierarchy
* Memory - data not safe
* NVMe SSD - data is safe
* Hard disk data is safe

Cloud:
* Compute and Storage disaggregation
* Caching layer
  * Remote memory
  * SSD
  * Block storage (EBS, managed disks)
* Object store
  * Has the ground truth for data (S3, blob store ...)
  * No durability for data that isn't in object store.

> No cloud provider ensures that your data is persistence, only if the data is in object store. 
> Because they don't replicate the data or make backups for the other layers.

## Cloud storage hierarchy across vendors
AWS:
* Block storage: Elastic block storage (EBS)
* Object storage: Simple storage service (S3)
* File storage: Elastic file system (EFS), FSx

Mircosoft Azure cloud
* Block storage: Disk storage (Managed disk)
* Object storage: Blob storage 
* File storage: Azure files, NetApp files

Googlle cloud platform
* Block storage: Persistent disk, Local disk
* Object storage: 
* File storage: 

## What changes for the cloud? Cost
Conventional:
* Paying for storage devices

Cloud:
* Paying for the compute instance, or data storage solution.
* Not paying for specific devices.
* Paying for every single data access.

## What changes for the cloud? Network
Cloud:
* Some compute instances have higher bandwidth than local storage, but this doesn't guarantee stable or low latency! 
* This is because of how the network works. Maybe there might be many request going through the network.

## Cloud cost analysis? - What should I cache?
Modeling caching - latency insensitive case
* Cost of not caching $=$ cost per object store request $\times$ (number of requests $-$ 1)
* Cost of caching = (hourly storage cost per GB $+$ hourly instance cost per GB) $\times$ cache size in GB $\times$ lifetime of cache in hours $+$ cache miss rate $\times$ number of requests 


Modeling cahcing - latency sensitive case
* Cost of not cahcing = object store request $\times$ (number of request $-$ 1) $\times$ concurrent reads to guarantee latency 
* Racing reads to reach the latency target:
  * Concurrent requests for the object
  * Proceed with the first response, ignore the rest (do many same request mutliple times).
  * If you have a target latency, you will have to send multiple request to reach the target. Notice you need real measurements to know the values.
* Cost of caching = same as previous 

Evaluation:
* Latency insensitive case
  * With cheapest option -> 7 requests per sec for an object
* Latency sensitive case
  * 2 requests per hour for an object is enough to justify caching.