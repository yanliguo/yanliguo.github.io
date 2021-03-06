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
* Scalable to support PB-data
* Distributed & fault tolerance
* Compatible with Hadoop / Spark / Kafka / Storm ...
* Time-serial data: each line of data have to be taged with a timestamp
![Druid Data Type][data-type]


## Architecture

![Druid Architecture][img-arch]

Druid cluster is highly available, highly scaleble horizonly. Even when the zookeeper is down, broker can provide query service based on the current state, instead of service unavailable.

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
![Druid data ingest flow][ingest-flow]

### Ingesting realtime data
As talked above, raw data is fed into realtime and aggregated (if needed) and kept in row oriented.  


#### Using hadoop job to ingest batch data

***[Akka implementation of druid-indexer job dispatcher][akka-druid-indexer]***
***[Working with Docker][akka-druid-indexer-with-docker]***

#### Indexing Service to ingest batch data
Providing data set to indexing service, Overload creates & dispatches & monitors new sub task to a Middle Manager. Middle Manager keeps local peons loading balance, that is to say, Middle Manager will find a not so busy peon to do the new task.  
![Indexing Service][indexing-service]

## A Query Session

![This is how druid processing query request][query-session]


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

[img-arch]: http://128.199.173.102/images/druid/architechture.png "Druid Architecture"
[data-type]: http://128.199.173.102/images/druid/column-types.png "Data types"
[query-session]: http://128.199.173.102/images/druid/query-flow.png "Query Session"
[indexing-service]: http://128.199.173.102/images/druid/indexing-service.png "Druid Indexing Service"
[ingest-flow]: http://128.199.173.102/images/druid/realtime-batch-data-flow.png "Data Ingest Flow"
