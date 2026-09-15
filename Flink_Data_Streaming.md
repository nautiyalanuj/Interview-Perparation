# Interview Rule of Thumb
 - Need event transport? 
   - Kafka / Event Hub / Kinesis
- Need real-time computation?
   - Flink
- Need only simple transformations?
  - Kafka Streams
- On AWS?
  - Kinesis + Flink 
- On Azure
  - Event Hub + Flink
- Large enterprise cross-cloud?
  - Kafka + Flink
  - because Kafka remains the de facto standard event backbone, while Flink is currently the most powerful general-purpose streaming engine for complex stateful real-time processing.
    
# What is data processing
- Data processing pipeline as name suggest is used to process a stream of huge amount of data.
- Example
 Interview Rule of Thumb - We have add aggregator metric, where we want to process real-time adds and create a metric/monitor over same.
  - Imagine you need to build a system that processes real-time messages from an IoT device deployed in moving trucks.
- Data processing can be divided into two stage **ingestion** where events/streams are ingested and **processing** stage where same events/streams are processed.
  - Kafka/Kinesis/Event Hub move and store events. Kafka Streams and Flink process events. Use whatever you like in different cloud systems.
  - Kafka Streams is lightweight embedded processing. Flink is a full-fledged distributed stream processing platform.
- References
  - https://www.redpanda.com/guides/event-stream-processing-flink-vs-kafka

# Which technology to use for data processing?

## Kafka Streams
- Kafka Streams is a Java library that processes data directly from Kafka.
- Kafka => Kafka Streams App => Kafka
- No separate cluster. Just deploy Java applications.
- Good at simple stream processing: filtering, mapping, aggregations
- Example: Count orders per user
  - Order Events =>  Kafka Streams => User Order Count Topic
- Use Kafka Streams when
  - Already use Kafka
  - Java ecosystem
  - Moderate scale
  - Simple-to-medium processing
  - Small operations team
- Don't use when (Then Flink is better)
  - Very large state
  - Very complex pipelines
  - Hundreds of TB/day
  - Multiple teams need shared processing platform

## Flink
 - Apache Flink is a distributed stream processing engine. Think of it as: Spark for streaming, but true real-time.
 - It excels at
   - Stateful processing
   - Complex event processing
   - Windowing
   - Joins
   - Event-time processing
- Example: Fraud detection
  - Card Swipe Events => Kafka => Flink => Fraud Alerts
  - Rule => same card used in 3 countries in 10 minutes; transaction > $5000; unusual spending pattern
  - Flink maintains state efficiently.
- Use Flink when
  - Large scale
  - Real-time analytics
  - Stateful processing
  - Event-time correctness
  - CEP (Complex Event Processing)
  - Streaming ML pipelines
- Why companies choose Flink? Need is 
  - millions of events/sec
  - low latency
  - exactly-once guarantees
  - Kafka Streams often becomes difficult at that scale.
