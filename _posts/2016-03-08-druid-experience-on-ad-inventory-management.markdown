---
layout:	post
title:	"Druid Experience on Ad Inventory Management"
date:	2016-03-08 16:00:00 +0800
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

## Architecture

### Ingesting realtime data

### Ingesting batch data

#### Using hadoop job

***[Akka implementation of druid-indexer job dispatcher][akka-druid-indexer]***
***[Working with Docker][akka-druid-indexer-with-docker]***

#### Indexing Service

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
[druid-overview-by-myself]:http://yanliguo.github.io/blog/druid-overview.html
[akka-druid-indexer]:https://yanliguo.github.org
[akka-druid-indexer-with-docker]:https://yanliguo.github.org
