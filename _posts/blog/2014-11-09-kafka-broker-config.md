---
layout: post
title: Kafka 0.8.1：broker config（doing）
description: Kafka构建的消息队列中，真正进行message存储的是broker
categories: kafka big-data
---


##Kafka整体架构的回顾

（todo）

要点：broker、producer、consumer的定位。




##Broker Configs

The essential configurations are the following:（必要的配置如下）

* broker.id
* log.dirs
* zookeeper.connect

Topic-level configurations and defaults are discussed in [more detail below](http://kafka.apache.org/documentation.html#topic-config).

**notes(ningg)**：什么叫作**Topic-level configuration**？是指与`topic`相关的配置。

下文针对各个属性的介绍将以如下形式进行：

* Property
	* Default
	* Description

###broker.id
* null(non-negative integer id)
* Each broker is uniquely identified by a non-negative integer id. This id serves as the broker's "name" and allows the broker to be moved to a different host/port without confusing consumers. You can choose any number you like so long as it is unique.（唯一标识broker，目的：当broker移动到另一个`host:port`后，不会对consumer造成影响）

###log.dirs
* /tmp/kafka-logs
* A comma-separated list of one or more directories in which Kafka data is stored. Each new partition that is created will be placed in the directory which currently has the fewest partitions.（以逗号`,`分割，Kafka data的存储位置；新建的partition将会被放置在当前partition数最小的目录下）

###port
* 6667
* The port on which the server accepts client connections.

###zookeeper.connect
* null
* Specifies the ZooKeeper connection string in the form `hostname:port`, where hostname and port are the host and port for a node in your ZooKeeper cluster. To allow connecting through other ZooKeeper nodes when that host is down you can also specify multiple hosts in the form `hostname1:port1,hostname2:port2,hostname3:port3`.
* ZooKeeper also allows you to add a "chroot" path which will make all kafka data for this cluster appear under a particular path. This is a way to setup multiple Kafka clusters or other applications on the same ZooKeeper cluster. To do this give a connection string in the form `hostname1:port1,hostname2:port2,hostname3:port3/chroot/path` which would put all this cluster's data under the path `/chroot/path`. Note that you must create this path yourself prior to starting the broker and consumers must use the same connection string.

**notes(ningg)**：关于`zookeeper`参数，几个问题：

* Kafka集群需要借助Zookeeper来进行管理，因此，需要设定Zookeeper集群的位置，可以只设置一个Zookeeper，也可以设置一个列表，疑问：设置一个zookeeper与一个zookeeper列表有差异吗？当只设置一个zookeeper服务器时，是否会自动获取zookeeper列表？
* 可以设置`chroot`目录，用于存储kafka集群相关数据，这么做的原因：方便同一个zookeeper集群，管理多个应用（例如，kafka集群）；但需要在启动broker之前，提前创建`chroot`目录，并且consumer需要使用相同的`zookeeper.connect`作为connection string。

###message.max.bytes
* 1000000
* The maximum size of a message that the server can receive. It is important that this property be in sync with the maximum fetch size your consumers use or else an unruly producer will be able to publish messages too large for consumers to consume.

###num.network.threads
* 3
* The number of network threads that the server uses for handling network requests. You probably don't need to change this.（处理网络请求所设定的线程数，通常不用调整这个参数）

###num.io.threads
* 8
* The number of I/O threads that the server uses for executing requests. You should have at least as many threads as you have disks.（server执行request时，启动的I/O线程数目，建议与磁盘个数相同）

###background.threads
* 4
* The number of threads to use for various background processing tasks such as file deletion. You should not need to change this.

###queued.max.requests
* 500
* The number of requests that can be queued up for processing by the I/O threads before the network threads stop reading in new requests.

###host.name
* null	
* Hostname of broker. If this is set, it will only bind to this address. If this is not set, it will bind to all interfaces, and publish one to ZK.

###advertised.host.name
* null	
* If this is set this is the hostname that will be given out to producers, consumers, and other brokers to connect to.

###advertised.port
* null	
* The port to give out to producers, consumers, and other brokers to use in establishing connections. This only needs to be set if this port is different from the port the server should bind to.

* socket.send.buffer.bytes	100 * 1024	The SO_SNDBUFF buffer the server prefers for socket connections.
* socket.receive.buffer.bytes	100 * 1024	The SO_RCVBUFF buffer the server prefers for socket connections.
* socket.request.max.bytes	100 * 1024 * 1024	The maximum request size the server will allow. This prevents the server from running out of memory and should be smaller than the Java heap size.
* num.partitions	1	The default number of partitions per topic if a partition count isn't given at topic creation time.
* log.segment.bytes	1024 * 1024 * 1024	The log for a topic partition is stored as a directory of segment files. This setting controls the size to which a segment file will grow before a new segment is rolled over in the log. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.roll.hours	24 * 7	This setting will force Kafka to roll a new log segment even if the log.segment.bytes size has not been reached. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.cleanup.policy	delete	This can take either the value delete or compact. If delete is set, log segments will be deleted when they reach the size or time limits set. If compact is set log compaction will be used to clean out obsolete records. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.retention.{minutes,hours}	7 days	The amount of time to keep a log segment before it is deleted, i.e. the default data retention window for all topics. Note that if both log.retention.minutes and log.retention.bytes are both set we delete a segment when either limit is exceeded. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.retention.bytes	-1	The amount of data to retain in the log for each topic-partitions. Note that this is the limit per-partition so multiply by the number of partitions to get the total data retained for the topic. Also note that if both log.retention.hours and log.retention.bytes are both set we delete a segment when either limit is exceeded. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.retention.check.interval.ms	5 minutes	The period with which we check whether any log segment is eligible for deletion to meet the retention policies.
* log.cleaner.enable	false	This configuration must be set to true for log compaction to run.
* log.cleaner.threads	1	The number of threads to use for cleaning logs in log compaction.
* log.cleaner.io.max.bytes.per.second	None	The maximum amount of I/O the log cleaner can do while performing log compaction. This setting allows setting a limit for the cleaner to avoid impacting live request serving.
* log.cleaner.dedupe.buffer.size	500*1024*1024	The size of the buffer the log cleaner uses for indexing and deduplicating logs during cleaning. Larger is better provided you have sufficient memory.
* log.cleaner.io.buffer.size	512*1024	The size of the I/O chunk used during log cleaning. You probably don't need to change this.
* log.cleaner.io.buffer.load.factor	0.9	The load factor of the hash table used in log cleaning. You probably don't need to change this.
* log.cleaner.backoff.ms	15000	The interval between checks to see if any logs need cleaning.
* log.cleaner.min.cleanable.ratio	0.5	This configuration controls how frequently the log compactor will attempt to clean the log (assuming log compaction is enabled). By default we will avoid cleaning a log where more than 50% of the log has been compacted. This ratio bounds the maximum space wasted in the log by duplicates (at 50% at most 50% of the log could be duplicates). A higher ratio will mean fewer, more efficient cleanings but will mean more wasted space in the log. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.cleaner.delete.retention.ms	1 day	The amount of time to retain delete tombstone markers for log compacted topics. This setting also gives a bound on the time in which a consumer must complete a read if they begin from offset 0 to ensure that they get a valid snapshot of the final stage (otherwise delete tombstones may be collected before they complete their scan). This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.index.size.max.bytes	10 * 1024 * 1024	The maximum size in bytes we allow for the offset index for each log segment. Note that we will always pre-allocate a sparse file with this much space and shrink it down when the log rolls. If the index fills up we will roll a new log segment even if we haven't reached the log.segment.bytes limit. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
* log.index.interval.bytes	4096	The byte interval at which we add an entry to the offset index. When executing a fetch request the server must do a linear scan for up to this many bytes to find the correct position in the log to begin and end the fetch. So setting this value to be larger will mean larger index files (and a bit more memory usage) but less scanning. However the server will never add more than one index entry per log append (even if more than log.index.interval worth of messages are appended). In general you probably don't need to mess with this value.
* log.flush.interval.messages	None	The number of messages written to a log partition before we force an fsync on the log. Setting this lower will sync data to disk more often but will have a major impact on performance. We generally recommend that people make use of replication for durability rather than depending on single-server fsync, however this setting can be used to be extra certain.
* log.flush.scheduler.interval.ms	3000	The frequency in ms that the log flusher checks whether any log is eligible to be flushed to disk.
* log.flush.interval.ms	None	The maximum time between fsync calls on the log. If used in conjuction with log.flush.interval.messages the log will be flushed when either criteria is met.
* log.delete.delay.ms	60000	The period of time we hold log files around after they are removed from the in-memory segment index. This period of time allows any in-progress reads to complete uninterrupted without locking. You generally don't need to change this.
* log.flush.offset.checkpoint.interval.ms	60000	The frequency with which we checkpoint the last flush point for logs for recovery. You should not need to change this.
* auto.create.topics.enable	true	Enable auto creation of topic on the server. If this is set to true then attempts to produce, consume, or fetch metadata for a non-existent topic will automatically create it with the default replication factor and number of partitions.
* controller.socket.timeout.ms	30000	The socket timeout for commands from the partition management controller to the replicas.
* controller.message.queue.size	10	The buffer size for controller-to-broker-channels
* default.replication.factor	1	The default replication factor for automatically created topics.
* replica.lag.time.max.ms	10000	If a follower hasn't sent any fetch requests for this window of time, the leader will remove the follower from ISR (in-sync replicas) and treat it as dead.
* replica.lag.max.messages	4000	If a replica falls more than this many messages behind the leader, the leader will remove the follower from ISR and treat it as dead.
* replica.socket.timeout.ms	30 * 1000	The socket timeout for network requests to the leader for replicating data.
* replica.socket.receive.buffer.bytes	64 * 1024	The socket receive buffer for network requests to the leader for replicating data.
* replica.fetch.max.bytes	1024 * 1024	The number of byes of messages to attempt to fetch for each partition in the fetch requests the replicas send to the leader.
* replica.fetch.wait.max.ms	500	The maximum amount of time to wait time for data to arrive on the leader in the fetch requests sent by the replicas to the leader.
* replica.fetch.min.bytes	1	Minimum bytes expected for each fetch response for the fetch requests from the replica to the leader. If not enough bytes, wait up to replica.fetch.wait.max.ms for this many bytes to arrive.
* num.replica.fetchers	1	
* Number of threads used to replicate messages from leaders. Increasing this value can increase the degree of I/O parallelism in the follower broker.
* 
* replica.high.watermark.checkpoint.interval.ms	5000	The frequency with which each replica saves its high watermark to disk to handle recovery.
* fetch.purgatory.purge.interval.requests	10000	The purge interval (in number of requests) of the fetch request purgatory.
* producer.purgatory.purge.interval.requests	10000	The purge interval (in number of requests) of the producer request purgatory.
* zookeeper.session.timeout.ms	6000	ZooKeeper session timeout. If the server fails to heartbeat to ZooKeeper within this period of time it is considered dead. If you set this too low the server may be falsely considered dead; if you set it too high it may take too long to recognize a truly dead server.
* zookeeper.connection.timeout.ms	6000	The maximum amount of time that the client waits to establish a connection to zookeeper.
* zookeeper.sync.time.ms	2000	How far a ZK follower can be behind a ZK leader.
* controlled.shutdown.enable	false	Enable controlled shutdown of the broker. If enabled, the broker will move all leaders on it to some other brokers before shutting itself down. This reduces the unavailability window during shutdown.
* controlled.shutdown.max.retries	3	Number of retries to complete the controlled shutdown successfully before executing an unclean shutdown.
* controlled.shutdown.retry.backoff.ms	5000	Backoff time between shutdown retries.
* auto.leader.rebalance.enable	false	If this is enabled the controller will automatically try to balance leadership for partitions among the brokers by periodically returning leadership to the "preferred" replica for each partition if it is available.
* leader.imbalance.per.broker.percentage	10	The percentage of leader imbalance allowed per broker. The controller will rebalance leadership if this ratio goes above the configured value per broker.
* leader.imbalance.check.interval.seconds	300	The frequency with which to check for leader imbalance.
* offset.metadata.max.bytes	1024	The maximum amount of metadata to allow clients to save with their offsets.

More details about broker configuration can be found in the scala class `kafka.server.KafkaConfig`.

##Topic-level configuration

Configurations pertinent to topics have both a global default as well an optional per-topic override. If no per-topic configuration is given the global default is used. The override can be set at topic creation time by giving one or more --config options. This example creates a topic named my-topic with a custom max message size and flush rate:
 > bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic my-topic --partitions 1 
        --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1
Overrides can also be changed or set later using the alter topic command. This example updates the max message size for my-topic:
 > bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic my-topic 
    --config max.message.bytes=128000
To remove an override you can do
 > bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic my-topic 
    --deleteConfig max.message.bytes
The following are the topic-level configurations. The server's default configuration for this property is given under the Server Default Property heading, setting this default in the server config allows you to change the default given to topics that have no override specified.
Property	Default	Server Default Property	Description
cleanup.policy	delete	log.cleanup.policy	A string that is either "delete" or "compact". This string designates the retention policy to use on old log segments. The default policy ("delete") will discard old segments when their retention time or size limit has been reached. The "compact" setting will enable log compaction on the topic.
delete.retention.ms	86400000 (24 hours)	log.cleaner.delete.retention.ms	The amount of time to retain delete tombstone markers for log compacted topics. This setting also gives a bound on the time in which a consumer must complete a read if they begin from offset 0 to ensure that they get a valid snapshot of the final stage (otherwise delete tombstones may be collected before they complete their scan).
flush.messages	None	log.flush.interval.messages	This setting allows specifying an interval at which we will force an fsync of data written to the log. For example if this was set to 1 we would fsync after every message; if it were 5 we would fsync after every five messages. In general we recommend you not set this and use replication for durability and allow the operating system's background flush capabilities as it is more efficient. This setting can be overridden on a per-topic basis (see the per-topic configuration section).
flush.ms	None	log.flush.interval.ms	This setting allows specifying a time interval at which we will force an fsync of data written to the log. For example if this was set to 1000 we would fsync after 1000 ms had passed. In general we recommend you not set this and use replication for durability and allow the operating system's background flush capabilities as it is more efficient.
index.interval.bytes	4096	log.index.interval.bytes	This setting controls how frequently Kafka adds an index entry to it's offset index. The default setting ensures that we index a message roughly every 4096 bytes. More indexing allows reads to jump closer to the exact position in the log but makes the index larger. You probably don't need to change this.
max.message.bytes	1,000,000	message.max.bytes	This is largest message size Kafka will allow to be appended to this topic. Note that if you increase this size you must also increase your consumer's fetch size so they can fetch messages this large.
min.cleanable.dirty.ratio	0.5	log.cleaner.min.cleanable.ratio	This configuration controls how frequently the log compactor will attempt to clean the log (assuming log compaction is enabled). By default we will avoid cleaning a log where more than 50% of the log has been compacted. This ratio bounds the maximum space wasted in the log by duplicates (at 50% at most 50% of the log could be duplicates). A higher ratio will mean fewer, more efficient cleanings but will mean more wasted space in the log.
retention.bytes	None	log.retention.bytes	This configuration controls the maximum size a log can grow to before we will discard old log segments to free up space if we are using the "delete" retention policy. By default there is no size limit only a time limit.
retention.ms	7 days	log.retention.minutes	This configuration controls the maximum time we will retain a log before we will discard old log segments to free up space if we are using the "delete" retention policy. This represents an SLA on how soon consumers must read their data.
segment.bytes	1 GB	log.segment.bytes	This configuration controls the segment file size for the log. Retention and cleaning is always done a file at a time so a larger segment size means fewer files but less granular control over retention.
segment.index.bytes	10 MB	log.index.size.max.bytes	This configuration controls the size of the index that maps offsets to file positions. We preallocate this index file and shrink it only after log rolls. You generally should not need to change this setting.
segment.ms	7 days	log.roll.hours	This configuration controls the period of time after which Kafka will force the log to roll even if the segment file isn't full to ensure that retention can delete or compact old data.





















##参考来源

* [Kafka Documentation](http://kafka.apache.org/documentation.html)






[NingG]:    http://ningg.github.com  "NingG"
