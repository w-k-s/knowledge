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

----

## Selecting a message broker

Copying an answer I gave to another engineer to help them decide when a particular kind of Message Broker ought to be used:

**Apache Kafka** 

- Kafka is used when you have a stream to which data is added very quickly from one end, and the data needs to be processed very quickly by a number of downstream systems on the other end. 
- Think of Kafka like checking-in for a flight at an airport. The check-in area is divided into a  number of check-in zones (similar to partitions in Kafka), and each zone has a number of counters to cater to a single airline (similar to topics in Kafka). This arrangement allows for a number of passengers for a number of downstream flights to be processed very quickly.

**RabbitMQ/ActiveMQ**

- RabbitMQ and ActiveMQ are when you need to distribute tasks between different downstreams: it's about receiving a message, figuring out where it needs to go and trying to make sure it gets there. Performance isn't that much of a concern.
- Think of RabbitMQ/ActiveMQ like a mailroom. Mail is received, sorted into bags for delivery (like Delivery Exchanges) and then delivered to the recipient. It's pretty reliable but not always fast.

**ZeroMQ, Redis**

- Zero MQ, the way I understand it, is a to-do list. It manages tasks that need to be done. The key thing to remember is that you're okay with losing track of the tasks.
- Think of Zero MQ like an inbox tray at a disk. You put work in that you need to do at the bottom of the to-do pile. Once you've worked through one file, you move it to the done pile and then pick up the next file.
- At any point, a cat can flip over the inbox tray. When that happens, there's a mess of paper on the floor and it's difficult to figure out what belongs where. 