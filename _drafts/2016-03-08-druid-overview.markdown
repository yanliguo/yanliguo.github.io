---
layout:	post
title:	"Druid Overview"
date:	2016-03-08 15:42:00 +0800
---

>*"A fast column-oriented distributed data store."*
>*- [Druid][druid-io]*

## Features
* OLAP
* Sub-second query
* Real-time
* Time-serial data: each line of data have to be taged with a timestamp
* Scalable to support PB-data
* Distributed & fault tolerance
* Compatible with Hadoop / Spark / Kafka / Storm ...

## Architecture

### Segment
Data in a small time interval (`a bucket`). For each pair of `<Dinmension_x, Value_y>`, a `bitmap` index is built, compressed and serialized, as an individual column,  into this segment. ([An article][druid-compression] about this compression algorithm). So only related dimension and values will be loaded in a query session.


### Realtime node
A small bucket (`small time interval`) of data will be kept in ***realtime node***, in row oriented format (like in mysql). Since the data size will not be that lardge, the querying is fast on realtime node. When the time window is closed, data comming will be kept in a new bucket. The old bucket is built into a segment and `hand off` to deep storage.

### Coordinator
Coordinator is the  command center of Druid. It monitors changes of meta data, segment regestry activity in `ZooKeeper` and notifies hitorical nodes to load new segments or drop old segments from their memory. It also keeps the loading balanced between historical nodes. 

### Broker node
Broker node keeps a map of bucket location in historical nodes and changes this map the moment historical node loads or drops a segment by listening nodes in Zookeeper. Broker node brokes a query into some sub queries, and sends each sub query to a historical node where the time-interval segment is kept.  

### Historical node
Segments in the past time intervals are kept in historical nodes. Historical loads segment into its off-heap memory as possible as it can, then cold data will be swapt into disk if the total size is beyond its memory capacity. Historical recieves queries from broker and do each sub query on a segment and the sends the segment result back to broker. It loads segment from deepstorage, following the orders of coordinator by listening load-queue node in Zookeeper.  

### Metadata
Metadata of segment, mysql.

### Deep Storage
The storage of serialized segment, historical data. Druid supports local disk, S2, hdfs ... .



## Data Ingestion


### Ingesting realtime data
As talked above, raw data is fed into realtime and kept in the right format of each line.  



#### Using hadoop job to ingest batch data

***[Akka implementation of druid-indexer job dispatcher][akka-druid-indexer]***
***[Working with Docker][akka-druid-indexer-with-docker]***

#### Indexing Service to ingest batch data


## A Query Session


## Merits
* Real-time & historical support, users don't have to write more job to handle real-time data to hand off to historical
* Fast & convenient compressed bitmap indexing on dimensions  ------- Core of Druid
* In-memory
## Demerits
* Historical is read only, writing overwrites historical segments
* Segment should & only should be divided by time, and any line of data have to be with a timestamp
* If there is not enough memory, cold data will be flashed into disk. Querying on cold data will be slower.

[druid-io]:http://druid.io/
[akka-druid-indexer]:https://yanliguo.github.org
[akka-druid-indexer-with-docker]:https://yanliguo.github.org
[druid-compression]:https://yanliguo.github.org
