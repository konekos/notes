# Apache Kafka

### 一、basic introduce
server
client-producer

### 二、环境搭建
#### 1.single

```bash
tar -zxvf kafka_2.10-0.10.2.1.tgz -C demo/
cd demo/
ln -svnf kafka_2.10-0.10.2.1 kafka
mkdir data/zk -p
mkdir data/kafka -p
cd kafka
vi config/zookeeper.properties //修改日志路径
vim config/server.properties //日志路径  listeners  zookeeper.connect

启动zk
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
netstat -na | grep 2181
启动kafka
bin/kafka-server-start.sh -daemon config/server.properties

连接到kafka 
bin/zookeeper-shell.sh localhost:2181

ls /

创建topics
bin/kafka-topics.sh --create --topic test --zookeeper localhost:2181  --partitions 1 --replication-factor 1
启动producer
bin/kafka-console-producer.sh --topic test --broker-list localhost:9092
启动consumer
bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092
从头收集
bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092 --from-beginning

```

#### 2.伪分布式



![1562162242659](E:\studydyup\notes\src\pic\Snipaste_2018-02-20_22-53-18.png)

复制3个server.properties文件，修改broker_id 端口号，日志目录。复制log4j.properties文件。

#### 3.分布式和broker的基本配置

zk 版本3.4.10

```
vim config/zoo.cfg

server.1=192.168.119.129:2888:3888
server.2=192.168.119.130:2888:3888
server.3=192.168.119.131:2888:3888 //通信端口和选举端口

启动3个zk  bin/zkServer.sh start conf/zoo.cfg
```

kafka修改配置文件,启动

修改host,添加

```
192.168.119.129 s1
192.168.119.130 s2
192.168.119.131 s3
```



```
//创建topic
bin/kafka-topics.sh --create --topic test3 --zookeeper s1:2181,s2:2181,s3:2181 --partitions 1 --replication-factor 3
//list topic
bin/kafka-topics.sh --list --zookeeper s1:2181,s2:2181,s3:2181
//topic详情
bin/kafka-topics.sh --describe --topic test3  --zookeeper s1:2181,s2:2181,s3:2181

Topic:test2	PartitionCount:1	ReplicationFactor:3	Configs:
	Topic: test2	Partition: 0	Leader: 1	Replicas: 1,0,2	Isr: 1,0,2
```

Isr：和leader同步状态的机器，leader挂了，有资格成为leader，保证高可用。



简单测试，启动producer

```
bin/kafka-console-producer.sh --topic test3 --broker-list s1:9092,s2:9092,s3:9092
```

启动consumer

```
bin/kafka-console-consumer.sh --topic test3 --bootstrap-server s1:9092,s2:9092,s3:9092
```

##### broker基本配置

- broker.id  唯一id，可以使用ip后三位。

- port 9092

- zookeeper.connect  zk集群，s1:2181,s2:2181,s3:2181/kafka  可以指定目录，不指定代表根目录

- log.dirs 存放commit log。可以多个目录

- num.recovery.threads.per.data.dir

![1562170465912](E:\studydyup\notes\src\pic\1562170465912.png)

- auto.create.topics.enable producer写不存在的topic consumer读不存在的topic client读不存在topic的metadata。默认允许。不建议允许。
- num.partitions  创建topic时候的partition数量。只能增加，不能减少。
- log.retension.ms/hours/  保留多久的数据   针对broker的。
- log.retension.bytes 保留的最大容量  针对partition的。如果一个topic有8个partition，这个topic最多保留8G。
- log.segment.bytes 。一个partition有多个segment，默认1G，达到后segment会closed，然后创建新的。closed的segment才可以expire。 这个值过小，必须经常关闭和分配，减少磁盘写效率。根据业务特点指定参数。

![1562171802919](E:\studydyup\notes\src\pic\1562171802919.png)

- log.segment.ms  segment到达closed的需要的时间。
- message.max.bytes  producer数据的时候，最大消息的大小。默认1M。无法恢复的错误，不能重传。

##### 多少个broker？

考虑磁盘容量，系统要求存放20T。一台最多4T，可以设置5个broker。

网络带宽。单broker的上下行带宽达到80%~90%时候，加多一个broker，到50%，再加一个到30%。

##### 多少个partition？

partition是kafka扩展的重要参数。partition数量可以分散brokers的负载。partition一般等于brokers，或者是brokers的倍数，partition均匀分布到brokers去。读写是根据partition的。

如果非常要求顺序，那么只用一个partition，可以保证顺序。

希望多大的吞吐量。写可以多线程写。读只能一个线程读。

当需要分析message来源，做partition的路由。

不需要过多估算partition，性能提升是抛物线。资源占用和增加选举时间。

###  三、Producer

producer是线程安全的。往buffer里放record，batch统一发送给broker。

#### 1. Producer message to kafka

![1562246207838](E:\studydyup\notes\src\pic\1562246207838.png)

可恢复异常，不可恢复异常。重试几次可配置。



kafka producer 强制属性

- bootstrap.servers 
- key.serializer
- value.serializer

##### Fire-and-forget Send

发了不管成功失败。

先创建topic，不要让kafka自己创建。

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Properties;
import java.util.concurrent.Future;
import java.util.stream.IntStream;

/*****************************************
 * @author: Jasu Wong
 * @Date: 2019-07-04 21:13
 *****************************************/
public class FireAndForgetSender {

    private final static Logger logger = LoggerFactory.getLogger(FireAndForgetSender.class);

    public static void main(String[] args) {
        Properties props = initProps();
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
        IntStream.range(0,10).forEach(i->{
            ProducerRecord<String, String> record = new ProducerRecord<>("fafsender", String.valueOf(i), i + "ss");
            producer.send(record);
            logger.info("success");
        });
    }

    private static Properties initProps() {
        final Properties props = new Properties();
        props.put("bootstrap.servers", "s1:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        return props;
    }
}
```

##### SyncSend

要同步等应答。get wait住。

```java
ProducerRecord<String, String> record = new ProducerRecord<>("fsender", String.valueOf(i), i + "ss");
            Future<RecordMetadata> send = producer.send(record);
            try {
                RecordMetadata metadata = send.get();
                System.out.println(metadata.topic());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
```

##### AsyncSend

发到kafka client的消息收集器。

成功有回调。

```java
ProducerRecord<String, String> record = new ProducerRecord<>("fsender", String.valueOf(i), i + "ss");
            producer.send(record, (r, e) -> {
                if (e == null) {
                    System.out.println(r.offset());
                }
            });
```



#### 2. Producer configuration in deep

- ACK  代表多少个partition副本必须接收到record，在producer认为写成功之前。非常影响消息会不会丢失。
  - acks=0，不关心，始终认为成功。
  - acks=1，leader副本（partition的leader）收到消息的时候，producer认为成功。如果不能写入leader（leader挂了或者还没选举好新leader），producer收到错误响应并会重试防止数据丢失。还是会丢数据，leader收到数据，在同步给其他follower时挂了。
  - acks=all。所有的在同步的副本（Isr里的broker）接收到消息，producer才会认为成功。保证数据不会丢。但效率低。

- buffer.memory   kafka发的消息会放到batch，达到一定数量和时间，才会发送。in-fight 发送中在等ack的消息。
- compression.type 发送的batch消息的压缩类型。uncompressed，snappy，lz4，gzip(效率大于snappy，cpu内存消耗高)，lz4等。整体压缩batch。
- retries 可重试的消息进行retries的次数。不可恢复的异常不会重试。
- batch.size sender里有若干batch，batch达到大小发送batch。
- linger.ms 和上个参数一起控制，时间到了大小没大也会发。
- client.id  用于做日志，统计，报表。
- max.in.fight.requests.per.connection 控制多少个消息不需要等应答。越大使用内存越高，设置太大会让batch无效。设置成1，一个一个地发，失败重试也不会发第二个。
- timeout.ms request.timeout.ms, metadata.fetct.timeout.ms 超时时间。
- max.block.ms 
- max.request.size  broker的

##### 担保顺序

保证在一个partition的顺序。 retries可能打乱顺序。设置一个partition，设置in-flight = 1。

#### 3. Serializers

放到consumer

#### 4. Partitions

kafka重要的概念，核心。实现高可用。

kafka message，key（可无）+value 和topic组成record。key为null，使用默认的partitioner

key做partition的路由。

3个partition 3个broker 3个副本。

```
bin/kafka-topics.sh --create --topic testp --zookeeper s1:2181,s2:2181,s3:2181 --partitions 3 --replication-factor 3
```

同一个key放到一个partition。没有key随机发。

当使用所有partition计算映射，当某partition不可用，会报错。

多个partition，可以多个线程并发读。



##### producer send源码

```java
/**
     * Implementation of asynchronously send a record to a topic.
     */
    private Future<RecordMetadata> doSend(ProducerRecord<K, V> record, Callback callback) {
        TopicPartition tp = null;
        try {
            // first make sure the metadata for the topic is available
            ClusterAndWaitTime clusterAndWaitTime = waitOnMetadata(record.topic(), record.partition(), maxBlockTimeMs);
            long remainingWaitMs = Math.max(0, maxBlockTimeMs - clusterAndWaitTime.waitedOnMetadataMs);
            Cluster cluster = clusterAndWaitTime.cluster;
            byte[] serializedKey;
            try {
                serializedKey = keySerializer.serialize(record.topic(), record.headers(), record.key());
            } catch (ClassCastException cce) {
                throw new SerializationException("Can't convert key of class " + record.key().getClass().getName() +
                        " to class " + producerConfig.getClass(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG).getName() +
                        " specified in key.serializer");
            }
            byte[] serializedValue;
            try {
                serializedValue = valueSerializer.serialize(record.topic(), record.headers(), record.value());
            } catch (ClassCastException cce) {
                throw new SerializationException("Can't convert value of class " + record.value().getClass().getName() +
                        " to class " + producerConfig.getClass(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG).getName() +
                        " specified in value.serializer");
            }
            int partition = partition(record, serializedKey, serializedValue, cluster);
            tp = new TopicPartition(record.topic(), partition);

            setReadOnly(record.headers());
            Header[] headers = record.headers().toArray();

            int serializedSize = AbstractRecords.estimateSizeInBytesUpperBound(apiVersions.maxUsableProduceMagic(),
                    compressionType, serializedKey, serializedValue, headers);
            ensureValidRecordSize(serializedSize);
            long timestamp = record.timestamp() == null ? time.milliseconds() : record.timestamp();
            log.trace("Sending record {} with callback {} to topic {} partition {}", record, callback, record.topic(), partition);
            // producer callback will make sure to call both 'callback' and interceptor callback
            Callback interceptCallback = this.interceptors == null ? callback : new InterceptorCallback<>(callback, this.interceptors, tp);

            if (transactionManager != null && transactionManager.isTransactional())
                transactionManager.maybeAddPartitionToTransaction(tp);

            RecordAccumulator.RecordAppendResult result = accumulator.append(tp, timestamp, serializedKey,
                    serializedValue, headers, interceptCallback, remainingWaitMs);
            if (result.batchIsFull || result.newBatchCreated) {
                log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
                this.sender.wakeup();
            }
            return result.future;
            // handling exceptions and record the errors;
            // for API exceptions return them in the future,
            // for other exceptions throw directly
        } catch (ApiException e) {
            log.debug("Exception occurred during message send:", e);
            if (callback != null)
                callback.onCompletion(null, e);
            this.errors.record();
            if (this.interceptors != null)
                this.interceptors.onSendError(record, tp, e);
            return new FutureFailure(e);
        } catch (InterruptedException e) {
            this.errors.record();
            if (this.interceptors != null)
                this.interceptors.onSendError(record, tp, e);
            throw new InterruptException(e);
        } catch (BufferExhaustedException e) {
            this.errors.record();
            this.metrics.sensor("buffer-exhausted-records").record();
            if (this.interceptors != null)
                this.interceptors.onSendError(record, tp, e);
            throw e;
        } catch (KafkaException e) {
            this.errors.record();
            if (this.interceptors != null)
                this.interceptors.onSendError(record, tp, e);
            throw e;
        } catch (Exception e) {
            // we notify interceptor about all exceptions, since onSend is called before anything else in this method
            if (this.interceptors != null)
                this.interceptors.onSendError(record, tp, e);
            throw e;
        }
    }
```

##### 自定义partitioner

```java
public class MyPartitioner implements Partitioner {

    private final static String[] BIZ_TYPE = new String[]{"LOGIN", "LOGOFF", "ORDER"};
    private final static String LOGIN = "LOGIN";
    private final static String LOGOFF = "LOGOFF";
    private final static String ORDER = "ORDER";


    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {

        if (keyBytes == null || keyBytes.length == 0) {
            throw new IllegalArgumentException("key is required biz type");
        }
        switch (key.toString().toUpperCase()) {
            case LOGIN:
                return 0;
            case LOGOFF:
                return 1;
            case ORDER:
                return 2;
                default:
                    throw new IllegalArgumentException("key is invalid");
        }

    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```

使用自定义partitioner

```java
private static Properties initProps() {
        final Properties props = new Properties();
        props.put("bootstrap.servers", "192.168.119.129:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        return props;
    }
```



#### 5. Summary



### 四、Consumer

#### 1、Consumer and Consumer group

使用kafka Consumer消费数据。先订阅topic，receive（不是被动接受的，自己去poll） message from the topic。

一个Consumer group里有多个consumer。

多个producer可以写数据到topic，也需要多个consumer去消费数据，一个consumer读取一个topic下多个partition的一个。

可以多consumer group读topic。

#### 2、 Consumer groups and partition Rebalance

consumer group里 有多个consumer，多个partition和consumer的平衡。增加partition和consumer都会触发rebalance。

p1->c1

p2->c2

p3->c3

​        c4(Idl)



rebalance使consumer group高可用，高可扩展性（简单安全增加移除consumer），但触发rebalance时 consumer不能消费message，当partitions从一个consumer转换到另一个，consumer会丢失当前状态（丢失offset），如果缓存了数据，要refresh数据，减慢了应用。



consumers 通过发送心跳到kafka broker的group coordinator维护consumer group和partition的关系。

#### 3、 Kafka Consumer Programming

```java
public class SimpleConsumer {

    private final static Logger logger = LoggerFactory.getLogger(SimpleConsumer.class);

    public static void main(String[] args) {

        final AtomicInteger counter = new AtomicInteger();
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(loadProps());
        consumer.subscribe(Collections.singletonList("test_c"));
        for (; ; ) {
            //这里只是接收，不要阻塞住。提交异步处理。
            ConsumerRecords<String, String> consumerRecords = consumer.poll(100);
            //timestamp record发送的时间
            consumerRecords.forEach(r -> {
                counter.getAndIncrement();
                System.out.println(r.timestamp());
                System.out.println(r.key());
                System.out.println(r.offset());
                if (counter.get() > 3) {
                    Runtime.getRuntime().halt(-1);
                }
            });

        }


    }

    private static Properties loadProps() {
        Properties properties = new Properties();
        properties.put("bootstrap.servers", "192.168.119.129:9092,192.168.119.130:9092,192.168.119.131:9092");
        properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("group.id", "test_group");
        //自动提交时间10s,测试自动commit
        properties.put("auto.commit.interval.ms", "10000");
        properties.put("client.id", "demo-consumer-client");
        return properties;
    }
}
```



```
Runtime.getRunTime().halt(-1); //直接杀死，不给任何机会执行其他程序（hook）finally等。
```



#### 4、 Consumer Configuration in deep



Kafka的Consumer是不能多线程操作的，producer可以。

- fetch.min.bytes	consumer要数据的时候至少要准备多少数据
- fetch.max.wait.ms     和上个参数共同控制，减少consumer和broker的通讯次数
- max.partition.fetch.bytes     partition最大拉取的大小，分摊到每个consumer。
- session.timeout.ms     broker和consumer的会话超时时间。超时认为客户端死了，consumer死了会触发rebalance。
- heartbeat.interval.ms     控制consumer和group coordinator的心跳频率。心跳时间必须比session timeout小，1/3时间比较科学。
- auto.offset.reset      earliest 从头开始读数据。  consumer读了多少数据，会提交给broker。达到最大的offset，就会读不到（换个group）。
- enable.auto.commit   自动提交group读了多少数据到broker。默认true。读的时候程序意外kill，未提交会导致读到重复数据。
- auto.commit.interval.ms  多少时间提交一次。
- partition.assignment.strategy   分配给Consumer partition的策略。Range（会因topic为奇数造成不均）或RoundBin。RangeAssigner接口，可以实现自定义。
- client.id      好处在broker可以有标识。
- max.poll.records     如果配置了前面的最小大小，可能会拉几次，达到一次返回的数据。
- send.buffer.bytes    
- receieve.buffer.bytes  在不同数据中心，调大可以减少交互的次数。



#### 5、 Commit and Offset in deep

kafka里出现重复数据是100%出现的，尽量不要丢失。

offset代表Consumer读到了哪个位置。kafka 0.9以后（之前在zk），有默认的topic记录（__consumer_offsets）。

auto commit  不想管理offset，kafka默认的配置。（poll时或者Consumer close时去异步提交offset）

5s做一次auto commit，花了3s读了100条，还未触发auto commit，如果此时rebalance或者consumer死掉了，没有commit上去，会读重复。



自己管理提交offset的方式，同步、异步、或者组合。

```
//自己写逻辑控制commit  同步 阻塞住。会重试
            consumer.commitSync();
```

```
//自己写逻辑控制commit  异步 不阻塞  出了问题不会重试
            consumer.commitAsync();
```

```java
//同步异步结合
try {
            for (; ; ) {
                //这里只是接收，不要阻塞住。提交异步处理。
                ConsumerRecords<String, String> consumerRecords = consumer.poll(100);
                //timestamp record发送的时间
                consumerRecords.forEach(r -> {
                    counter.getAndIncrement();
                    System.out.println(r.timestamp());
                    System.out.println(r.key());
                    System.out.println(r.offset());
                    if (counter.get() > 3) {
                        Runtime.getRuntime().halt(-1);
                    }
                });
                //
                consumer.commitAsync();
            }
        }finally {
            //会重试
            consumer.commitSync();
        }
```



```
//提交指定offset的原因，处理record中间出错时，后面的也不能继续处理，那么指定offset提交，防止丢数据。
        Map<TopicPartition, OffsetAndMetadata> map = new HashMap<>();
        TopicPartition tp = new TopicPartition("test_c", 1);
        OffsetAndMetadata om = new OffsetAndMetadata(14, "no metadata");
        map.put(tp, om);

        for (; ; ) {
            //这里只是接收，不要阻塞住。提交异步处理。
            ConsumerRecords<String, String> consumerRecords = consumer.poll(100);
            //timestamp record发送的时间
            consumerRecords.forEach(r -> {
                counter.getAndIncrement();
                System.out.println(r.timestamp());
                System.out.println(r.key());
                System.out.println(r.offset());
            });
            //只提交到offset=14
            consumer.commitSync(map);
        }
```

##### 提交特定的offset

#### 6、 Consumer programming in deep

#### 7、 Serializers&Deserializers

自定义Serializer 和 Deserializer

```java
/**
 * 4 bytes
 * [4],[4].[10],[4],[20]
 * [4],[10],[20]
 *
 * String可变长的。
 */
public class User {
    private int age;
    private String name;
    private String address;

    public User() {
    }

    public User(int age, String name, String address) {
        this.age = age;
        this.name = name;
        this.address = address;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

```java
public class UserSerializer implements Serializer<User> {
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {

    }

    @Override
    public byte[] serialize(String topic, User data) {
        if (data == null) {
            return null;
        }
        int age = data.getAge();
        String name = data.getName();
        String address = data.getAddress();

        byte[] naBytes;
        int nameLength = 0;
        if (name != null) {
            naBytes = name.getBytes();
        } else {
            naBytes = new byte[0];
        }
        nameLength = naBytes.length;

        int addrLength = 0;
        byte[] addBytes;

        if (address != null) {
            addBytes = address.getBytes();
        } else {
            addBytes = new byte[0];
        }
        addrLength = addBytes.length;

        ByteBuffer byteBuffer = ByteBuffer.allocate(4 + 4 + nameLength + 4 + addrLength);
        byteBuffer.putInt(age);
        byteBuffer.putInt(nameLength);
        byteBuffer.put(naBytes);
        byteBuffer.putInt(addrLength);
        byteBuffer.put(addBytes);

        return byteBuffer.array();
    }

    @Override
    public void close() {

    }
}
```

```java
public class UserDeserializer implements Deserializer<User> {
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        //do nothing
    }

    @Override
    public User deserialize(String topic, byte[] data) {

        if (data == null) {
            return null;
        }
        if (data.length < 12) {
            throw new SerializationException("error");
        }

        ByteBuffer byteBuffer = ByteBuffer.wrap(data);
        int age = byteBuffer.getInt();

        int nameLength = byteBuffer.getInt();
        byte[] nameBytes = new byte[nameLength];
        byteBuffer.get(nameBytes);
        String name = new String(nameBytes);


        int addrLength = byteBuffer.getInt();
        byte[] addrBytes = new byte[addrLength];
        byteBuffer.get(addrBytes);
        String address = new String(addrBytes);

        return new User(age,name,address);
    }

    @Override
    public void close() {

    }
}
```

#### 8、producer和consumer的拦截器

在produce的过程中，对message进行加工。

cosumer在poll到message之前，决定需不需要这个record，被过滤掉的offset也会提交，不会重复读。

producer拦截器不允许返回为null，否则序列化空指针。

```java
public class UserDeserializer implements Deserializer<User> {
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        //do nothing
    }

    @Override
    public User deserialize(String topic, byte[] data) {

        if (data == null) {
            return null;
        }
        if (data.length < 12) {
            throw new SerializationException("error");
        }

        ByteBuffer byteBuffer = ByteBuffer.wrap(data);
        int age = byteBuffer.getInt();

        int nameLength = byteBuffer.getInt();
        byte[] nameBytes = new byte[nameLength];
        byteBuffer.get(nameBytes);
        String name = new String(nameBytes);


        int addrLength = byteBuffer.getInt();
        byte[] addrBytes = new byte[addrLength];
        byteBuffer.get(addrBytes);
        String address = new String(addrBytes);

        return new User(age,name,address);
    }

    @Override
    public void close() {

    }
}
```



#### 9. Rebalance listeners

一个partition，只能一个Consumer读。

触发rebalance。

```

```



consumer.assignment(); //分配了几个partition。

consumer.assign(Collection partition);//手动分配。不会受到consumer group的管理。

### 五、Kafka内部原理

1. kafka replication 如何工作
2. kafka如何处理来自producer和consumer的requests
3. kafka如何处理储存比如文件格式和索引

#### 1、Cluster Membership

Kafka使用ZK维护broker，broker有唯一ID。在ZK brokers/ids/创建节点。

当broker断开了和zk的连接（网络问题或者垃圾回收过长），通知其他节点。

broker断开后，它还会存在于其他数据结构里（replicas）。如果重启了老的broker（相同id），会立即加入cluster。



#### 2、Controller

集群的老大，是其中一个broker，最先启动的节点。负责partition的选举。

如果Controller挂了，各节点收到通知，重新选举。



controller_epoch：解决脑裂，出现两个master的问题。



当Controller意识到一个broker离开了，要重新选举partition的leader。



当Controller意识到一个broker加入，检查是不是副本，通知去同步数据。



#### 3、 Replication

kafka核心概念。保证kafka的高可用性。

一个topic，被partition，一个partition有多个副本。broker存储不止这个partition。



leader replication：每个partition有一个leader，所有生产和消费都经过leader做，确保数据一致性。

follower replication：即时同步leader的数据。

leader的另一个任务是知道 哪个follower同步up-to-date。follower尝试与leader一致，如果同步落后太多会被移除（Isr）。

leader 1-10   

follower1  1-9      未来的的及同步10，不影响在Isr里面。

follower  1-8         同步超时时间超过配置移出Isr。



OffSet最小的值，为High Water Maker(木桶原理)。



为了与leader同步，follower发起fetch request来同步。fetch请求包含了offset，是有顺序的。

持续同步leader的数据才会在 Isr——In-sync replicas，才有资格选举成为leader。



#### 4、Request Processing

Kafka内部通信协议。

请求消息头包括：

- 请求类型（Api key）
- 请求版本（Version）
- Correlation Id：唯一标识请求的数字，会出现在Response和错误日志。
- Client Id：consumer、producer、broker给其他broker的partition发请求。



使用的reactor的改进模型。没有用netty和mina。



两种requests

- produce requests
- fetch requests ： consumer或replica拉取同步

Kafka是最终一致性。

请求非leader的partition，返回“Not a leader for Partition”。

Producer发送时，会先返回partitions的信息包含leader副本等，如果指定往非leader的broker发数据，该broker会缓存其他节点的metadata，不会异常。客户端也会缓存这些信息。信息过期后，会报异常，但是是可恢复异常，会重试。data.max.age.ms ，元数据的保存时间。



![1563286166816](E:\studydyup\notes\src\pic\1563286166816.png)



High Water Mark

Isr里的所有副本，Consumer只能消费到的线。是所有副本里最小的，其他副本里多出的对consumer不可见。

#### 5、Physical Storage

kafka实际是分布式的commit log的storage。Kafka最基本储存单元为partition，一个partition只能处于一个磁盘（顺序读写）。

partition分隔为segments，当前正写入的是active的，不能delete。写满了会关闭，再分配新的。

文件格式：

Compaction：



![20190716232432](E:\studydyup\notes\src\pic\20190716232432.png)





Delete Event



### 六、构建可靠的kafka传输机制

#### 1、Reliable Gurarantees

kafka写到内存里，就算成功了，没有到真正落盘。

#### 2、 Broker prevent data loss

3个重要的参数控制broker的可靠性。

- replication factor N，允许你丢失N-1个副本。推荐3个副本，越多IO开销越大。

- Unclean Leader Election  由unclean.leader.election.enable控制。 一个partition 3个副本，两个follower挂了（out of sync）。然后leader也挂了，Producer写不了，Consumer也读不了，读写是一致的。此时一个follower恢复了，会变成leader，但会看不到leader的数据。虽然有可用性，但是牺牲了一致性。

  总之如果允许了out-of-sync的副本成为leader节点，有数据丢失和不一致的风险。如果不允许，那么降低了可用性。

  kafka 0.11版本，unclean.leader.election.enable默认是false，只有Isr的节点才能成为leader，服务才可用。牺牲可用性换取一致性。

- Reliable-Mininum In-Sync Replica  min.insync.replicas  (可在broker配置，是全局的；topic level配置，只控制topic)至少几个副本在Isr里，才能写。0.10版本acks=all（确保leader同步到了所有Isr队列，副本挂掉就忽略了），min.insync.replicas=2，不能写，但可读。0.11版本ack=1；可以往leader写，但是不可读。

  

  **最小同步中副本数可以设置为 总副本数 N/2 + 1;  unclean为false，acks=all。防止数据不丢失。**

  

#### 3、Producer deliver Reliable

如果broker设置了3个副本，不允许unclean副本竞选leader。如果Producer的acks=1; 会丢数据。

如果3个副本，不允许unclean。Producer的acks=all; 如果发消息到kafka，但是leader挂了还在选取leader，Producer会收到leader不可用异常。如果Producer异常处理不当，则丢数据。

- 根据需求设置acks。0，发出去不管了。1，只确认leader。all，isr都同步才可以。
- 在配置和代码正确处理异常。可恢复异常重置次数retries，



#### 4、 Consumer deliver Reliable

数据commit到broker，才对consumer可见。

两个重要数据，Isr，HW（high water mark）。consumer看到的是HW（ISR里的最低offset）线。



在一个consumer group，一个consumer挂掉了，另一个consumer接管时从停掉的位置（提交的offset）开始。



如果consumer消费数据的时候，拿到数据并提交了offset，没处理完就挂了，那么数据就丢了。



Consumer侧需要配置的参数：

- group.id
- auto.offset.reset
- enable.auto.commit
- auto.commit.interval.ms



##### 在Consumer明确提交offset

- 处理一条提交一条

- 提交频率在性能和重复数据做权衡

- 确保知道提交的是哪个offset

- Rebalance

- Consumer需要重试

  - cache  存在一个地方做处理
  - submit  发往kafka另一个topic

- 消息处理时间过长 提交给threadpool，threadpoll满了，pause掉consumer。

  - 多线程提交offset
  - 问题：每一个offset结束时间不一定，提交哪个offset？

  要做控制，提交最大值。不能提交小的offset，造成duplicate。

- Exactly-once delivery && at-least once

  - Producer阶段，提交数据的时候，写成功了，ack出错了。造成重复数据。没满足幂等。
  - consumer阶段  重复读取offset

  解决方案：

  - messgae里有unique key。->存到数据库/redis里，或者请求api等做处理。业务要满足幂等。
  - 

#### 5、 Vliadating System Reliable



























