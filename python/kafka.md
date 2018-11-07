#  Kafka使用

## 生产者

生产者 有好三个模块可供选择`confluent_kafka`、`SimpleProducer`、`kafka-python`模块的KafkaProducer。

**confluent_kafka**：只能用于linux平台
**SimpleProducer**、**kafka-python**:  可与用windows平台，SimpleProducer速度更快一点，但timestamp字段 不正确。windows下使用kafka-python模块的KafkaProducer，速度虽然没有SimpleProducer快，但是产生消息的 timestamp 字段正常。此timestamp字段在 sync_slave 当加入新的消费者组时应该会使用到(按照timestamp查找offset)。

### confluent_kafka

```
from confluent_kafka import Producer
class KafkaP:
    """
    生产模块：根据不同的key，区分消息
    """
    def __init__(self, bootstrap_servers, compression_type='gzip'):
        self.bootstrap_servers = bootstrap_servers
        self.compression_type = compression_type
        self.message_max_bytes = 52428800   # 不要大于kafka服务设置
        if self.compression_type is None:
            self.producer = Producer(
                                    {
                                        'bootstrap.servers': self.bootstrap_servers,
                                        'message.max.bytes': self.message_max_bytes
                                    }
                                    )
        else:
            self.producer = Producer(
                                    {
                                        'bootstrap.servers': self.bootstrap_servers,
                                        'message.max.bytes': self.message_max_bytes,
                                        'compression.type': self.compression_type
                                    }
                                    )
    def send_data(self, message, topic):
        # self.producer.send(topic=topic, key=key, value=message)
        self.producer.produce(topic, message.encode('utf-8'))
        # self.producer.flush()
    def reconnection_producer(self):
        if self.compression_type is None:
            self.producer = Producer(
                                    {
                                        'bootstrap.servers': self.bootstrap_servers,
                                        'message.max.bytes': self.message_max_bytes
                                    }
                                    )
        else:
            self.producer = Producer(
                                    {
                                        'bootstrap.servers': self.bootstrap_servers,
                                        'message.max.bytes': self.message_max_bytes,
                                        'compression.type': self.compression_type
                                    }
                                    )
    def close_producer(self):
        self.producer.flush()
```

### SimpleProducer
```
from kafka import SimpleProducer, SimpleClient

class KafkaP:
    """
    生产模块：根据不同的key，区分消息
    """

    def __init__(self, bootstrap_servers):
        self.bootstrap_servers = bootstrap_servers
        self.retries = 3
        self.ack = 0
        self.async = True
        self.client = SimpleClient(hosts=self.bootstrap_servers)
        self.producer = SimpleProducer(client=self.client,
                                       async_send=self.async,
                                       req_acks=SimpleProducer.ACK_NOT_REQUIRED
                                       )

    def send_data(self, message, topic):
        # self.producer.send(topic=topic, key=key, value=message)
        self.producer.send_messages(topic, message.encode('utf-8'))
        # self.producer.send_messages(topic, bytes(message))
        # self.producer.flush()

    def reconnection_producer(self):
        self.client = SimpleClient(hosts=self.bootstrap_servers)
        self.producer = SimpleProducer(client=self.client,
                                       async_send=self.async,
                                       req_acks=SimpleProducer.ACK_NOT_REQUIRED
                                       )

    def close_producer(self):
        # self.producer.flush()
        self.producer.stop()
        self.client.close()
```

### python-kafka
```
from kafka import KafkaProducer

class KafkaP:
    """
    生产模块：根据不同的key，区分消息
    """

    def __init__(self, bootstrap_servers, compression_type='gzip'):
        self.bootstrap_servers = bootstrap_servers
        self.retries = 3
        self.ack = 0
        self.linger_ms = 0
        self.compression_type = compression_type
        if self.compression_type is None:
            self.producer = KafkaProducer(bootstrap_servers=self.bootstrap_servers,
                                          retries=self.retries,
                                          acks=self.ack,
                                          linger_ms=self.linger_ms,
                                          )
        else:
            self.producer = KafkaProducer(bootstrap_servers=self.bootstrap_servers,
                                          retries=self.retries,
                                          acks=self.ack,
                                          linger_ms=self.linger_ms,
                                          compression_type=self.compression_type
                                          )

    def send_data(self, message, topic, key=None):
        self.producer.send(topic=topic, key=key, value=message)
        # print message

    def reconnection_producer(self):
        if self.compression_type is None:
            self.producer = KafkaProducer(bootstrap_servers=self.bootstrap_servers,
                                          retries=self.retries,
                                          acks=self.ack,
                                          linger_ms=self.linger_ms
                                          )
        else:
            self.producer = KafkaProducer(bootstrap_servers=self.bootstrap_servers,
                                          retries=self.retries,
                                          acks=self.ack,
                                          linger_ms=self.linger_ms,
                                          compression_type=self.compression_type
                                          )

    def close_producer(self):
        self.producer.flush()
        self.producer.close()
```


## 消费者

```
from kafka import KafkaConsumer
from kafka import TopicPartition
import sys
reload(sys)
sys.setdefaultencoding('utf-8')


class KafkaC:
    """
    消费模块: 通过不同groupid消费topic里面的消息
    """
    def __init__(self, bootstrap_servers, topic, group, action=None, offset=None, enable_auto_commit=True):
        self.bootstrap_servers = bootstrap_servers
        self.topic = topic
        self.group_id = group
        self.action = action
        self.offset = offset
        # self.enable_auto_commit = True
        self.enable_auto_commit = enable_auto_commit
        self.auto_commit_interval_ms = 1000
        self.consumer = KafkaConsumer(
                                        # self.kafka_topic,
                                        # auto_offset_reset默认latest，如果是第一次使用一个新的消费者组，不管前面数据是否消费
                                        # 都不会消费，使用earliest参数就可以消费前面没有消费的数据
                                        auto_offset_reset='earliest',
                                        group_id=self.group_id,
                                        bootstrap_servers=self.bootstrap_servers,
                                        enable_auto_commit=self.enable_auto_commit,
                                        auto_commit_interval_ms=self.auto_commit_interval_ms
                                      )

    # 获取所有topics
    def get_all_topics(self):
        return self.consumer.topics()

    # 获取最开始offset
    def get_beginning_offsets(self):
        _ps = [TopicPartition(self.topic, p) for p in self.consumer.partitions_for_topic(self.topic)]
        return _ps, self.consumer.beginning_offsets(_ps)

    # 获取最新的offset,比最新数据的offset大1,因为kafkaoffset从0开始
    def get_end_offsets(self):
        _ps = [TopicPartition(self.topic, p) for p in self.consumer.partitions_for_topic(self.topic)]
        return _ps, self.consumer.end_offsets(_ps)

    # 获取当前消费者开始消费的offset
    def get_last_position(self, partition=None):
        _ps = [TopicPartition(self.topic, p) for p in self.consumer.partitions_for_topic(self.topic)]
        self.consumer.assign(_ps)
        return self.consumer.position(partition=_ps[0])

    # 以时间戳(微秒)方式获取offset,消息必须带有timestamp字段，即kafka版本大于0.10.0才支持
    def get_offset_by_timestamp(self, timestamp):
        _ps = [TopicPartition(self.topic, p) for p in self.consumer.partitions_for_topic(self.topic)]
        return _ps, self.consumer.offsets_for_times({_ps[0]: timestamp})

    def consume_data(self, offset=None):
        """
        :param action: none 从 kafka 正常的 `CURRENT-OFFSET` 开始消费
                    custom 从指定offset开始
                    begin 从 kafka 从这个topic最开始消费
                    end 从 kafka 从这个topic从最新生成的数据库开始，会跳过未消费数据，慎用
        :param offset: 数字 int>=0 type为custom时有效从当前数字 offset 开始，包括当前数字，
                       如果数字大于当前topic总offset，从最新生成的数据库开始
        :return:
        """

        # 获取topic所有分区并分配给当前消费者, 需要使用 assign 的话， 在 KafkaConsumer 初始化时就不能指定 topic
        _ps = [TopicPartition(self.topic, p) for p in self.consumer.partitions_for_topic(self.topic)]
        if offset is None:
            offset = self.get_last_position()
        self.consumer.assign(_ps)
        for p in self.consumer.partitions_for_topic(self.topic):
            # 也可以只指定一个分区的 offset
            self.consumer.seek(TopicPartition(self.topic, p), offset)
        # try:
        #     for message in self.consumer:
        #         yield message
        # except KeyboardInterrupt as e:
        #     print(e)
        for message in self.consumer:
            yield message

    def consume_data_stop(self):
        """
        获取到最新的offset自动停止
        :param action: 
                    none 从 kafka 正常的 `CURRENT-OFFSET` 开始消费
                    custom 从指定offset开始
                    begin 从 kafka 从这个topic最开始消费
                    end 从 kafka 从这个topic从最新生成的数据库开始，会跳过未消费数据，慎用
        :param offset: 
                    数字 int>=0 type为custom时有效从当前数字 offset 开始，包括当前数字，
                    如果数字大于当前topic总offset，从最新生成的数据库开始
        :return:
        """

        # 获取topic所有分区并分配给当前消费者, 需要使用 assign 的话， 在 KafkaConsumer 初始化时就不能指定 topic
        _ps = [TopicPartition(self.topic, p) for p in self.consumer.partitions_for_topic(self.topic)]
        self.consumer.assign(_ps)
        if self.action is None:
            pass
        elif self.action == 'begin':
            self.consumer.seek_to_beginning()
        elif self.action == 'end':
            self.consumer.seek_to_end()
        elif self.action == 'custom':
            for p in self.consumer.partitions_for_topic(self.topic):
                # 也可以只指定一个分区的 offset
                self.consumer.seek(TopicPartition(self.topic, p), self.offset)
        else:
            print('action values is not support! Plesase input "begin|end|custom"')
            sys.exit(1)

        for message in self.consumer:
            yield message

    def commit_consumer(self):
        # self.consumer.commit()
        self.consumer.commit_async()    # 异步提交

    def close_consumer(self):
        self.consumer.close(autocommit=self.enable_auto_commit)

if __name__ == '__main__':
    BOOTSTRAP_SERVERS = "pycdhnode2:9092,pycdhnode3:9092,pycdhnode4:9092"
    KAFKA_TOPIC = "py_etl.py_etl_active_stock_fund_evaluate_2_1"
    GROUP = 'py_group'
    ACTION = 'custom'
    OFFSET = 887931

    # CONSUMER = KafkaC(BOOTSTRAP_SERVERS, KAFKA_TOPIC, GROUP, ACTION, OFFSET)
    CONSUMER = KafkaC(BOOTSTRAP_SERVERS, KAFKA_TOPIC, GROUP)

    ALL_TOPICS = CONSUMER.get_all_topics()
    if len(ALL_TOPICS) == 0:
        print('topic: {0} is not exist'.format(KAFKA_TOPIC))
        CONSUMER.close_consumer()
    elif len(ALL_TOPICS) != 0 and KAFKA_TOPIC not in ALL_TOPICS:
        print('topic: {0} is not exist'.format(KAFKA_TOPIC))
        CONSUMER.close_consumer()
    else:
        # MESSAGE = CONSUMER.consume_data()
        # # MESSAGE = CONSUMER.consume_data()
        # for MSG in MESSAGE:
        #     print(MSG)
        #     # time.sleep(1)
        #     # value = msg.value
        #     # print(value.decode('utf8'))

        PS, STARTOFFSETS = CONSUMER.get_beginning_offsets()
        PS, ENDOFFSETS = CONSUMER.get_end_offsets()
        STARTOFFSET = int(STARTOFFSETS[PS[0]])
        ENDOFFSET = int(ENDOFFSETS[PS[0]])   # 获取topic第一个0分区的最新的offset
        POSITION = CONSUMER.get_last_position(PS[0])    # 获取topic第一个0分区最新的开始消费的offset

        print "startoffset: {0}".format(STARTOFFSET)
        print "endoffset: {0}".format(ENDOFFSET)
        print "position: {0}".format(POSITION)
        print "total not consume log: {0}".format(ENDOFFSET - POSITION)

        print '---------------------------------------------'

        if CONSUMER.action is not None and CONSUMER.offset is not None:
            if CONSUMER.offset > ENDOFFSET - 1:
                CONSUMER.offset = ENDOFFSET
            POSITION = CONSUMER.offset

        MESSAGE = CONSUMER.consume_data_stop()

        print "startoffset: {0}".format(STARTOFFSET)
        print "endoffset: {0}".format(ENDOFFSET)
        print "position: {0}".format(POSITION)
        print "total not consume log: {0}".format(ENDOFFSET - POSITION)

        # dt = '2018-07-26 10:45:29'
        # ts = int(time.mktime(time.strptime(dt, "%Y-%m-%d %H:%M:%S")))
        # # print int(ts)   # 获取毫秒
        # # print int(round(ts*1000))   # 获取微秒
        # PS, TIMEOFFSETS = CONSUMER.get_offset_by_timestamp(int(round(ts*1000)))
        # TIMEOFFSET = TIMEOFFSETS[PS[0]]
        # if TIMEOFFSET is not None:
        #     print 'message @timestamp: {0} offset: {1}'.format(TIMEOFFSET[1], TIMEOFFSET[0])
        # else:
        #     print 'message @timestamp: {0} offset: None'.format(int(round(ts*1000)))

        sys.exit(0)

        start_time = datetime.datetime.now()

        i = 0
        if ENDOFFSET > POSITION:
            for MSG in MESSAGE:
                # print MSG
                # print(
                #         'topic:{0} partition:{1} offset:{2} key:{3} value:{4}'
                #         .format(MSG.topic, MSG.partition, MSG.offset, MSG.key, MSG.value.decode('utf-8'))
                #      )
                i += 1
                if MSG.offset == ENDOFFSET - 1:
                    CONSUMER.close_consumer()
                    break
        else:
            print('no message to consumption')
            CONSUMER.close_consumer()

        print('total consumption: {0}'.format(i))

        end_time = datetime.datetime.now()
        exec_time = 'exec_time:{0}'.format(end_time - start_time)
        print(exec_time)
```