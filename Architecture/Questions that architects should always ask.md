# Questions that Architects / Backend Engineers should always ask

**1. What if there are multiple instances of my service/application running?**

_Example 1_:

- Two instances consuming from same queue [(Competing Consumer)](https://docs.microsoft.com/en-us/azure/architecture/patterns/competing-consumers)
- This might result in messages being processed out of order
- e.g. if a customer creates an order and immediately cancels it, two messages are added to a queue
- instance A picks up create order, instance B picks up cancel order at the same time
- instance B immediately dismisses cancel order because order does not exist yet?
- instance A still creates order after a few seconds - not what the custoemr intended
- **SOLUTION**: queue sharding ensures message grouped by an id (e.g. customer id) is always handled by the same instance

_Example 2_:

- Load Balancer balancing between instance A and B
- Applciation saves data on instance's physical memory
- Data might be saved when load balancer routes to instance A.
- That same data will not be available when load balancer routes to instance B.
- **SOLUTION**: Use sticky sessions or save data in cloud volume.

**2. What if this endpoint is called twice simultaneously?**

**3. What if X is down?**

- Circuit breaker pattern
- Redundancy

**4. What if messages are received out of order?**

**5. What if duplicated messages are received?**

**6. What are all the error scenarios?**
