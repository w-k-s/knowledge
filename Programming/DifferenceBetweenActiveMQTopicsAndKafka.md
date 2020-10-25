# The differences between ActiveMQ Topics & Kafka

1. ActiveMQ Broker had to maintain the delivery state of every message resulting into lower throughput. Kafka producer doesn’t wait for acknowledgements from the broker. 
	
	Overall throughput will be high if broker can handle the messages as fast as producer.

2. Kafka has a more efficient storage format. On average, each message had an overhead of 9 bytes in Kafka, versus 144 bytes in ActiveMQ.

3. ActiveMQ is push based messaging system and Kafka is pull based messaging system . In AcitveMQ, Producer push messages to all consumers. Producer has responsibility to ensure that message has been delivered. In Kafka, Consumer will pull messages from broker at its own time. It’s the responsibility of consumer to consume the messages it has supposed to consume.

4. Slow Consumers in AMQ can cause problems on non-durable topics since they can force the broker to keep old messages in RAM which once it fills up, forces the broker to slow down producers, causing the fast consumers to be slowed down. A slow consumer in Kakfa does not impact other consumers.

5. In Kafka - A consumer can rewind back to an old offset and re-consume data. It is useful when you fix some issue and decide to re-play the old messages post issue resolution.

6. Performance of Queue and Topics degrades with addition of more consumers in ActiveMQ. But Kafka does not have that dis-advantage with addition of more consumers.

7. Kafka is highly scalable due to replication of partitions. It can ensure that messages are delivered in a sequence with in a partition.

8. ActiveMQ is traditional messaging system where as Kakfa is meant for distributed processing system with huge amount of data and effective for stream processing

Due to above efficiencies, Kafka throughput is 2x - 4x times more than normal messaging systems like ActiveMQ and RabbitMQ.