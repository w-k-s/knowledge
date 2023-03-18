# Data Transformation Day (AWS)

I recently added a seminar hosted by AWS called "Data Transformation Day". These are my notes on the event.

**NOTE**: 
- If you're reading this and you're not me (or you _are_ me from, but from the future), please note that I've only just cleaned up what I understood; I haven't fact-checked any of the points yet. Let the buyer beware.

---

## NoSQL
- The biggest impact of NoSQL in the tech industry is that whereas before you had to figure out how to map your application's data to a SQL database, now you have to look at your use-case and figure out which storage engine best fits your needs.
- **Review, Repeat, Review**: Modelling your data on any NoSQL db is tricky, and likely needs to evolve with time. It's important to monitor the how your data is read and written by your applications and review if your data model can be improved to further optimise the process.

## DynamoDB
- [**Single table design**](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) 1 application has one table (or you could say, 1 microservice = 1 table)
- DynamoDB removes the burden figuring out how many servers and partitions you need on the DB admin and architect; you just need to know your anticipated read/write workload and DynamoDB will figure out how to handle it.

--

**Raw Notes**

- AWS Kendra: natural language queries on enterprise data eg how much will we earn in next quarter based on previous month cash flows
- I spoke to a number of people who said they'd moved away from k8s to serverless which drastically reduced their infra cost. One person described how each developer deployed the application composed of serverless instances on the developers own Aws account (using AWS Cloud Dev Kit). This was cheaper than setting up a dedicated Dev ataging environmwnt. 
- AWS is also investing in serverless services to help customers manage costs. Aurora, opensearch, quicksight, I forgot the name of the nosql one.
- Hadoop complexity unpopular. Spark for data warehousing.
- data democritation: data silos -> Data public for maximum benefits and opportunities 
- Redshift ml can be asked to build a model from a dB, it analyses the data to find which model fits the data best. 
- underlying, serverless pipeline is the next big thing 
- for sharding, recommended to use vitesse. Used by careem. When dB size is > 100tb and metrics indicate increased latency in queries, that's when you may consider sharding. 
- discussed statements solution using Kafka, Cassandra. Agreed with it - you can have partition per customer per month and it can scale. For dynamodb the size of the cluster can autoacalle. Suggested integrating with an olap and then a reporting tool to generate statements. 
- I think in general data flow is online oltp (SQL/nosql)-> stream (Kafka, etl) > olap (Redshift) / warehouse < data crawler < data catalog < data query < data insight tool .
- one thing that was repeated over and over again was serverless architecture on the cloud in order to manage costs. 