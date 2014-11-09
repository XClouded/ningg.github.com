---
layout: post
title: Kafka 0.8.1��Consumer API and Consumer Configs
description: Kafka��һ����Ϣ���У��Ǿ�Ҫ��������ж�ȡ���ݣ���ȡ���ݵ����ʵ�壬����Kafka��Consumer
category: kafka
---




##Consumer API

��δ�Kafka�ж�ȡ���ݣ����ַ�ʽ��

* High Level Consumer API��
* Simple Consumer API��
* Kafka Hadoop Consumer API��

###High Level Consumer API

	class Consumer {
	  /**
	   *  Create a ConsumerConnector
	   *
	   *  @param config  at the minimum, need to specify the groupid of the consumer and the zookeeper
	   *                 connection string zookeeper.connect.
	   */
	  public static kafka.javaapi.consumer.ConsumerConnector createJavaConsumerConnector(ConsumerConfig config);
	}

	/**
	 *  V: type of the message
	 *  K: type of the optional key assciated with the message
	 */
	public interface kafka.javaapi.consumer.ConsumerConnector {
	  /**
	   *  Create a list of message streams of type T for each topic.
	   *
	   *  @param topicCountMap  a map of (topic, #streams) pair
	   *  @param decoder a decoder that converts from Message to T
	   *  @return a map of (topic, list of  KafkaStream) pairs.
	   *          The number of items in the list is #streams. Each stream supports
	   *          an iterator over message/metadata pairs.
	   */
	  public <K,V> Map<String, List<KafkaStream<K,V>>>
		createMessageStreams(Map<String, Integer> topicCountMap, Decoder<K> keyDecoder, Decoder<V> valueDecoder);

	  /**
	   *  Create a list of message streams of type T for each topic, using the default decoder.
	   */
	  public Map<String, List<KafkaStream<byte[], byte[]>>> createMessageStreams(Map<String, Integer> topicCountMap);

	  /**
	   *  Create a list of message streams for topics matching a wildcard.
	   *
	   *  @param topicFilter a TopicFilter that specifies which topics to
	   *                    subscribe to (encapsulates a whitelist or a blacklist).
	   *  @param numStreams the number of message streams to return.
	   *  @param keyDecoder a decoder that decodes the message key
	   *  @param valueDecoder a decoder that decodes the message itself
	   *  @return a list of KafkaStream. Each stream supports an
	   *          iterator over its MessageAndMetadata elements.
	   */
	  public <K,V> List<KafkaStream<K,V>>
		createMessageStreamsByFilter(TopicFilter topicFilter, int numStreams, Decoder<K> keyDecoder, Decoder<V> valueDecoder);

	  /**
	   *  Create a list of message streams for topics matching a wildcard, using the default decoder.
	   */
	  public List<KafkaStream<byte[], byte[]>> createMessageStreamsByFilter(TopicFilter topicFilter, int numStreams);

	  /**
	   *  Create a list of message streams for topics matching a wildcard, using the default decoder, with one stream.
	   */
	  public List<KafkaStream<byte[], byte[]>> createMessageStreamsByFilter(TopicFilter topicFilter);

	  /**
	   *  Commit the offsets of all topic/partitions connected by this connector.
	   */
	  public void commitOffsets();

	  /**
	   *  Shut down the connector
	   **/
	  public void shutdown();
	}

You can follow [this example][Consumer Group Example] to learn how to use the high level consumer api.


###Simple Consumer API

	class kafka.javaapi.consumer.SimpleConsumer {
	/**
	*  Fetch a set of messages from a topic.
	*
	*  @param request specifies the topic name, topic partition, starting byte offset, maximum bytes to be fetched.
	*  @return a set of fetched messages
	*/
	public FetchResponse fetch(kafka.javaapi.FetchRequest request);
	
	/**
	*  Fetch metadata for a sequence of topics.
	*
	*  @param request specifies the versionId, clientId, sequence of topics.
	*  @return metadata for each topic in the request.
	*/
	public kafka.javaapi.TopicMetadataResponse send(kafka.javaapi.TopicMetadataRequest request);
	
	/**
	*  Get a list of valid offsets (up to maxSize) before the given time.
	*
	*  @param request a [[kafka.javaapi.OffsetRequest]] object.
	*  @return a [[kafka.javaapi.OffsetResponse]] object.
	*/
	public kafak.javaapi.OffsetResponse getOffsetsBefore(OffsetRequest request);
	
	/**
	* Close the SimpleConsumer.
	*/
	public void close();
	}
	

For most applications, the high level consumer Api is good enough. Some applications want features not exposed to the high level consumer yet (e.g., set initial offset when restarting the consumer). They can instead use our low level SimpleConsumer Api. The logic will be a bit more complicated and you can follow the example in [here][Simple Consumer Example].




###Kafka Hadoop Consumer API

Providing a horizontally scalable solution for aggregating and loading data into Hadoop was one of our basic use cases. To support this use case, we provide a Hadoop-based consumer which spawns off many map tasks to pull data from the Kafka cluster in parallel. This provides extremely fast pull-based Hadoop data load capabilities (we were able to fully saturate the network with only a handful of Kafka servers).

Usage information on the hadoop consumer can be found [here][Kafka Hadoop Consumer Example].




##Consumer Configs

The essential consumer configurations are the following:

* group.id
* zookeeper.connect


���Ľ���ϸ������Щ������

* Property
	* Default
	* Description

* group.id
	* null
	* A string that uniquely identifies the group of consumer processes to which this consumer belongs. By setting the same group id multiple processes indicate that they are all part of the same consumer group.

**notes(ningg)**��consumer group����ϰһ�£�Ϊʲô����������ʣ�Kafka��һ��message�����͵���Щ�ط��أ�һ����Ⱥ����Consumer��һ����ֻ���͸�ĳһ������������Consumer��ͬʱmessageҪ����ͬһ��Consumer�б�֤message�Ĵ���˳����������һ�������������£�ͬʱΪ�˸������ܣ�������һ�����consumer group��ͬһ��group�¿��԰������consumer��ÿ��group���յ�message����ʵ�������ڲ���һ��consumer�����һ��partition�е�message�ͷ��͸�һ��group����˳����������ǲ����������ʣ�һ��consumer group��ֻ����һ��consumer���ܹ�ʵ�ִ���˳�����ˣ�Ϊʲô��Ҫ���ö��consumer��

* zookeeper.connect
	* null
	* Specifies the ZooKeeper connection string in the form `hostname:port` where host and port are the host and port of a ZooKeeper server. To allow connecting through other ZooKeeper nodes when that ZooKeeper machine is down you can also specify multiple hosts in the form `hostname1:port1,hostname2:port2,hostname3:port3`.
	* The server may also have a ZooKeeper `chroot` path as part of it's ZooKeeper connection string which puts its data under some path in the global ZooKeeper namespace. If so the consumer should use the same chroot path in its connection string. For example to give a chroot path of `/chroot/path` you would give the connection string as `hostname1:port1,hostname2:port2,hostname3:port3/chroot/path`.

**notes(ningg)**��������`zookeeper.connect`ʱ����������zookeeper��`chroot`��`chroot`�ĺ��壺�ı�Ԫ������global Zookeeper namespace�еĴ洢λ�ã�һ���޸���`chroot`������Ҫ������Zookeeperʱ��Ҳ����`chroot`��������ʽ��`hostname1:port1,hostname2:port2,hostname3:port3/chroot/path`������ǰ��⣬ǰ���`/chroot/path`��`hostname1:port1`Ҳ����Ч�ģ�

* consumer.id
	* null	
	* Generated automatically if not set.

* socket.timeout.ms
	* 30 * 1000
	* The socket timeout for network requests. The actual timeout set will be `max.fetch.wait` + `socket.timeout.ms`.���ȴ�message��ʱ�䣩

* socket.receive.buffer.bytes
	* 64 * 1024
	* The socket receive buffer for network requests

* fetch.message.max.bytes
	* 1024 * 1024
	* The number of byes of messages to attempt to fetch for each topic-partition in each fetch request. These bytes will be read into memory for each partition, so this helps control the memory used by the consumer. The fetch request size must be at least as large as the maximum message size the server allows or else it is possible for the producer to send messages larger than the consumer can fetch.��consumer��������messagesʱ������ֽ�����ͨ��Ҫ��`fetch.message.max.bytes`����Ϊmaximum message size��

* auto.commit.enable
	* true
	* If true, periodically commit to ZooKeeper the offset of messages already fetched by the consumer. This committed offset will be used when the process fails as the position from which the new consumer will begin.��Ĭ��`true`����ʾ��Consumer�ɹ���ȡmessage����zookeeper����message��offset��ʾcommit��committed offset�����ã���consumer processʧ�ܺ��µ�consumer����һoffset�����¿�ʼ����

* auto.commit.interval.ms
	* 60 * 1000
	* The frequency in ms that the consumer offsets are committed to zookeeper.��Consumer�೤ʱ���ύһ��offset��

**notes(ningg)**���ѵ�����consumerÿ�ɹ�fetchһ��message����commitһ��offset��

* queued.max.message.chunks
	* 10
	* Max number of message chunks buffered for consumption. Each chunk can be up to `fetch.message.max.bytes`.���������message chunk�ĸ�����

**notes(ningg)**��message chunkʲô��˼��������

* rebalance.max.retries
	* 4
	* When a new consumer joins a consumer group the set of consumers attempt to "rebalance" the load to assign partitions to each consumer. If the set of consumers changes while this assignment is taking place the rebalance will fail and retry. This setting controls the maximum number of attempts before giving up.���µ�consumer���뵽consumer group�����consumer group�е�������partition������ٷ��䣬�����������У���Щconsumer set�ҷ����仯����᳢������ִ�У��˲�������ʾ���ԵĴ�������

* fetch.min.bytes
	* 1
	* The minimum amount of data the server should return for a fetch request. If insufficient data is available the request will wait for that much data to accumulate before answering the request.��server��fetch request���ص���С�ֽ��������data���㣬���ȴ��ۻ��㹻������֮���ٽ�����Ӧ����

* fetch.wait.max.ms
	* 100
	* The maximum amount of time the server will block before answering the fetch request if there isn't sufficient data to immediately satisfy `fetch.min.bytes`���趨��`fetch.min.bytes`�����û���㹻���ݣ�����ȴ�ʱ�䣩

* rebalance.backoff.ms
	* 2000
	* Backoff time between retries during rebalance.��reblalanceʱ����ͬ��retry֮����˱�ʱ������������retry֮��ļ��ʱ�䣩

* refresh.leader.backoff.ms
	* 200
	* Backoff time to wait before trying to determine the leader of a partition that has just lost its leader.��ʧȥleader���ٴ�����leader���˱�ʱ�䣩

* auto.offset.reset
	* largest
	* What to do when there is no initial offset in ZooKeeper or if an offset is out of range:����Zookeeper��û��initial offset����offset������Χʱ������Զ�����offset����
		* smallest : automatically reset the offset to the smallest offset
		* largest : automatically reset the offset to the largest offset
		* anything else: throw exception to the consumer

* consumer.timeout.ms
	* -1
	* Throw a timeout exception to the consumer if no message is available for consumption after the specified interval�����û��consumer���õ�message���ȴ��೤ʱ���ϵͳ�׳��쳣��

* client.id
	* group id value
	* The client id is a user-specified string sent in each request to help trace calls. It should logically identify the application making the request.������׷�ٵ��ù��̣�

* zookeeper.session.timeout.ms
 	* 6000
	* ZooKeeper session timeout. If the consumer fails to heartbeat to ZooKeeper for this period of time it is considered dead and a rebalance will occur.��һ��ʱ����consumer���ʧȥ��Zookeeper֮������������϶�consumer�Ѿ���ʧ������consumer group�ڽ���rebalance��

**notes(ningg)**������`zookeeper.session.timeout.ms`�����`auto.commit.interval.ms`֮��Ĺ�ϵ��ǰ�ߺ�������heartbeat�������߸������offset commit��

* zookeeper.connection.timeout.ms
	* 6000
	* The max time that the client waits while establishing a connection to zookeeper.��client��zookeeper�������ӵ�ʱ�䣬������һʱ�䣬�Զ��ͷţ�

* zookeeper.sync.time.ms
 	* 2000
	* How far a ZK follower can be behind a ZK leader��**ʲô��˼**����

More details about consumer configuration can be found in the scala class `kafka.consumer.ConsumerConfig`.



##�ο���Դ

* [Consumer Group Example][Consumer Group Example]
* [Simple Consumer Example][Simple Consumer Example]
* [Kafka Hadoop Consumer Example][Kafka Hadoop Consumer Example]




[Consumer Group Example]:		https://cwiki.apache.org/confluence/display/KAFKA/Consumer+Group+Example
[Simple Consumer Example]:		https://cwiki.apache.org/confluence/display/KAFKA/0.8.0+SimpleConsumer+Example
[Kafka Hadoop Consumer Example]:		https://github.com/linkedin/camus/








