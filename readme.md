# Домашнее задание Kafka

Запуск элементов Kafka в docker compose: <br/>
docker compose -f 'docker-compose.yaml' up -d --build

## 1. Создание топика:
```
docker exec -ti broker /usr/bin/kafka-topics --create  --bootstrap-server broker:9092 --replication-factor 1 --partitions 4 --topic my_topic"

--property "parse.key=true" --property "key.separator=,"

```

## 2. Информация по топику:
```
docker exec -ti broker /usr/bin/kafka-topics --bootstrap-server broker:9092 --describe --topic my_topic
Topic: my_topic TopicId: XG9w4rF8QGa4YKbFtPg9eg PartitionCount: 4       ReplicationFactor: 1    Configs: 
        Topic: my_topic Partition: 0    Leader: 1       Replicas: 1     Isr: 1
        Topic: my_topic Partition: 1    Leader: 1       Replicas: 1     Isr: 1
        Topic: my_topic Partition: 2    Leader: 1       Replicas: 1     Isr: 1
        Topic: my_topic Partition: 3    Leader: 1       Replicas: 1     Isr: 1
```
## 3. Запись сообщений в топик (в формате key-value)
```
docker exec -ti broker /usr/bin/kafka-console-producer --bootstrap-server broker:9092 --topic my_topic  --property "parse.key=true" --property "key.separator=,"
>k1,v1
>k2,v2
>k3,v3
>k4,v4
>k5,v5
>^C 
```

## 4. Чтение сообщений

docker exec -ti broker /usr/bin/kafka-console-consumer --bootstrap-server broker:9092 --topic my_topic --from-beginning
```
v1
v2
v3
v4
v5
^C
Processed a total of 5 messages
```

Сообщения в топике "my_topic"

![my_topic](/img/msg_in_topic.jpg)


## 5. Список всех топиков

```
docker exec -ti broker /usr/bin/kafka-topics --list  --bootstrap-server broker:9092
__connect-config
__connect-offsets
__connect-status
__consumer_offsets
__schemas
__transaction_state
_confluent-ksql-default__command_topic
default_ksql_processing_log
my_topic
```

В kafkaUI:

![all topics](/img/all_topics.jpg)


# Доступ к Kafka из приложения на Python

Пример приложения помещения сообщения в топик

```python
from confluent_kafka import Producer

conf = { 'bootstrap.servers': 'localhost:9092'}

producer = Producer(conf)

def delivery_report(err, msg):
    if err is not None:
        print(f'Ошибка доставки сообщения: {err}')
    else:
        print(f'Сообщение доставлено в {msg.topic()} [{msg.partition()}]')

topic = 'my_topic'

for i in range(10):
    producer.produce(topic, key=str(i), value=f'Сообщение {i}', callback=delivery_report)
    producer.poll(0)

producer.flush()
```

Пример приложения чтения сообщений из топика
```python
from confluent_kafka import Consumer, KafkaException

conf = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'my_group',
    'auto.offset.reset': 'earliest'
}

consumer = Consumer(conf)

topic = 'my_topic'
consumer.subscribe([topic])

try:
    while True:
        msg = consumer.poll(timeout=1.0)
        if msg is None:
            continue
        if msg.error():
            if msg.error().code() == KafkaError._PARTITION_EOF:
                print(f'Конец раздела {msg.topic()} [{msg.partition()}]')
            elif msg.error():
                raise KafkaException(msg.error())
        else:
            print(f'Получено сообщение: {msg.value().decode("utf-8")}')
finally:
    consumer.close()
```

Пример вывода приложений:  <br/>

```
python client.py

Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]
Сообщение доставлено в my_topic [0]

python reader.py
Получено сообщение: Сообщение 0
Получено сообщение: Сообщение 1
Получено сообщение: Сообщение 2
Получено сообщение: Сообщение 3
Получено сообщение: Сообщение 4
Получено сообщение: Сообщение 5
Получено сообщение: Сообщение 6
Получено сообщение: Сообщение 7
Получено сообщение: Сообщение 8
Получено сообщение: Сообщение 9
```