# 📨 Spring Boot + Kafka — Complete In-Depth Guide

> **"Apache Kafka is a distributed event streaming platform. It handles millions of events per second for real-time data pipelines and event-driven microservices."**

---

## 📑 Table of Contents

1. [What is Kafka?](#1-what-is-kafka)
2. [Kafka Architecture](#2-kafka-architecture)
3. [Core Concepts](#3-core-concepts)
4. [Setup & Configuration](#4-setup--configuration)
5. [Producer (Sending Messages)](#5-producer-sending-messages)
6. [Consumer (Receiving Messages)](#6-consumer-receiving-messages)
7. [Topics & Partitions](#7-topics--partitions)
8. [Consumer Groups](#8-consumer-groups)
9. [Error Handling & Dead Letter Queue](#9-error-handling--dead-letter-queue)
10. [Serialization (JSON, Avro)](#10-serialization-json-avro)
11. [Kafka with Microservices](#11-kafka-with-microservices)
12. [Kafka Streams](#12-kafka-streams)
13. [Testing Kafka](#13-testing-kafka)
14. [Monitoring & Operations](#14-monitoring--operations)
15. [Best Practices](#15-best-practices)
16. [Interview Questions & Answers (50+)](#16-interview-questions--answers-50)

---

## 1. What is Kafka?

```
Kafka = Distributed Message Broker + Event Log

Traditional Queue:                 Kafka:
Message consumed → GONE!           Message consumed → STILL THERE!
One consumer reads                 Multiple consumers can read
                                   Messages stored for days/weeks

Use cases:
  ✅ Event-driven microservices (order → payment → notification)
  ✅ Real-time data pipelines (logs, metrics, clickstream)
  ✅ Event sourcing (store all state changes)
  ✅ Stream processing (Kafka Streams)
  ✅ Log aggregation (centralize logs from 100+ services)
```

---

## 2. Kafka Architecture

```
┌────────────────────────────────────────────────────────┐
│                    Kafka Cluster                        │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ Broker 1 │  │ Broker 2 │  │ Broker 3 │             │
│  │          │  │          │  │          │             │
│  │ Topic A  │  │ Topic A  │  │ Topic A  │             │
│  │ Part 0   │  │ Part 1   │  │ Part 2   │             │
│  │ (Leader) │  │ (Leader) │  │ (Leader) │             │
│  │          │  │          │  │          │             │
│  │ Topic B  │  │ Topic B  │  │          │             │
│  │ Part 0   │  │ Part 1   │  │          │             │
│  └──────────┘  └──────────┘  └──────────┘             │
│                                                         │
│  Producers ──messages──► Brokers ──messages──► Consumers │
└────────────────────────────────────────────────────────┘
```

---

## 3. Core Concepts

| Concept | Description |
|---------|-------------|
| **Broker** | Kafka server. Stores messages. Cluster = multiple brokers. |
| **Topic** | Named channel for messages. Like a database table. |
| **Partition** | Topic is split into partitions for parallelism. |
| **Producer** | Sends messages to topics. |
| **Consumer** | Reads messages from topics. |
| **Consumer Group** | Group of consumers sharing work. Each partition → one consumer in group. |
| **Offset** | Position of a message in a partition. Consumer tracks its offset. |
| **Replication** | Partitions are replicated across brokers for fault tolerance. |
| **Zookeeper/KRaft** | Cluster coordination. KRaft (new) replaces Zookeeper. |

```
Topic: "orders" with 3 partitions:

Partition 0: [msg0, msg3, msg6, msg9 ...]  → Consumer A
Partition 1: [msg1, msg4, msg7, msg10 ...]  → Consumer B
Partition 2: [msg2, msg5, msg8, msg11 ...]  → Consumer C

Messages within a partition are ORDERED.
Messages across partitions are NOT ordered.
```

---

## 4. Setup & Configuration

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```properties
# application.properties
spring.kafka.bootstrap-servers=localhost:9092

# Producer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# Consumer
spring.kafka.consumer.group-id=my-service
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

```yaml
# docker-compose.yml — Kafka with KRaft (no Zookeeper!)
version: '3.8'
services:
  kafka:
    image: confluentinc/cp-kafka:7.6.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:29093
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:29093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk
```

---

## 5. Producer (Sending Messages)

```java
// Simple producer
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderProducer {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    // Fire and forget
    public void sendOrderEvent(OrderEvent event) {
        kafkaTemplate.send("order-events", event);
        log.info("Sent order event: {}", event);
    }
    
    // With key (ensures same key → same partition → ordering!)
    public void sendWithKey(OrderEvent event) {
        kafkaTemplate.send("order-events", event.getOrderId(), event);
        // All events for orderId="123" go to SAME partition → ordered!
    }
    
    // With callback (async confirmation)
    public void sendWithCallback(OrderEvent event) {
        CompletableFuture<SendResult<String, OrderEvent>> future = 
            kafkaTemplate.send("order-events", event.getOrderId(), event);
        
        future.thenAccept(result -> {
            log.info("Sent to partition={}, offset={}",
                result.getRecordMetadata().partition(),
                result.getRecordMetadata().offset());
        }).exceptionally(ex -> {
            log.error("Failed to send: {}", ex.getMessage());
            return null;
        });
    }
    
    // Send to specific partition
    public void sendToPartition(OrderEvent event, int partition) {
        kafkaTemplate.send("order-events", partition, event.getOrderId(), event);
    }
}

// Event class
@Data @AllArgsConstructor @NoArgsConstructor
public class OrderEvent {
    private String type;        // ORDER_CREATED, ORDER_PAID, ORDER_SHIPPED
    private String orderId;
    private String userId;
    private BigDecimal amount;
    private LocalDateTime timestamp;
}
```

---

## 6. Consumer (Receiving Messages)

```java
@Service
@Slf4j
public class OrderConsumer {
    
    // Simple consumer
    @KafkaListener(topics = "order-events", groupId = "payment-service")
    public void handleOrderEvent(OrderEvent event) {
        log.info("Received order event: type={}, orderId={}", event.getType(), event.getOrderId());
        
        switch (event.getType()) {
            case "ORDER_CREATED" -> processPayment(event);
            case "ORDER_CANCELLED" -> refundPayment(event);
        }
    }
    
    // With metadata
    @KafkaListener(topics = "order-events", groupId = "audit-service")
    public void handleWithMetadata(
            @Payload OrderEvent event,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            @Header(KafkaHeaders.RECEIVED_TIMESTAMP) long timestamp) {
        
        log.info("Event: {} from partition={}, offset={}", event, partition, offset);
    }
    
    // Manual acknowledgment
    @KafkaListener(topics = "order-events", groupId = "inventory-service",
                   containerFactory = "manualAckFactory")
    public void handleWithManualAck(OrderEvent event, Acknowledgment ack) {
        try {
            inventoryService.reserveStock(event);
            ack.acknowledge();  // Commit offset only after successful processing
        } catch (Exception e) {
            log.error("Failed to process: {}", e.getMessage());
            // Don't acknowledge → message will be redelivered!
        }
    }
    
    // Batch consumer
    @KafkaListener(topics = "order-events", groupId = "analytics-service",
                   containerFactory = "batchFactory")
    public void handleBatch(List<OrderEvent> events) {
        log.info("Received batch of {} events", events.size());
        analyticsService.processBatch(events);
    }
}
```

---

## 7. Topics & Partitions

```java
// Create topics programmatically
@Configuration
public class KafkaTopicConfig {
    
    @Bean
    public NewTopic orderEventsTopic() {
        return TopicBuilder.name("order-events")
            .partitions(6)           // 6 partitions for parallelism
            .replicas(3)             // 3 copies for fault tolerance
            .config(TopicConfig.RETENTION_MS_CONFIG, "604800000")  // 7 days
            .build();
    }
    
    @Bean
    public NewTopic userEventsTopic() {
        return TopicBuilder.name("user-events")
            .partitions(3)
            .replicas(3)
            .compact()               // Keep only latest value per key
            .build();
    }
}
```

```
Partition count guidelines:
  Low throughput:   3-6 partitions
  Medium:           6-12 partitions
  High throughput:  12-50+ partitions
  
  Rule: partitions >= max number of consumers in a group
  More partitions = more parallelism = higher throughput
```

---

## 8. Consumer Groups

```
Topic "orders" (3 partitions):

Consumer Group: "payment-service"
  Consumer A → Partition 0
  Consumer B → Partition 1
  Consumer C → Partition 2
  Each message processed by ONE consumer in the group ✅

Consumer Group: "notification-service"
  Consumer D → Partition 0, 1
  Consumer E → Partition 2
  Same messages, processed INDEPENDENTLY by this group ✅

Multiple groups = Multiple independent consumers of same data!
```

---

## 9. Error Handling & Dead Letter Queue

```java
@Configuration
public class KafkaErrorConfig {
    
    @Bean
    public DefaultErrorHandler errorHandler(KafkaTemplate<String, Object> kafkaTemplate) {
        // Send failed messages to dead letter topic
        DeadLetterPublishingRecoverer recoverer = 
            new DeadLetterPublishingRecoverer(kafkaTemplate);
        
        // Retry 3 times with 1 second delay, then send to DLT
        return new DefaultErrorHandler(recoverer, new FixedBackOff(1000L, 3));
    }
}

// Process dead letter messages
@KafkaListener(topics = "order-events.DLT", groupId = "dlt-processor")
public void handleDeadLetter(ConsumerRecord<String, OrderEvent> record) {
    log.error("Dead letter: topic={}, key={}, value={}",
        record.topic(), record.key(), record.value());
    // Alert, manual retry, or store for investigation
}
```

---

## 10. Serialization (JSON, Avro)

```java
// JSON Serialization (default in Spring Boot)
// Producer: JsonSerializer → sends JSON bytes
// Consumer: JsonDeserializer → reads JSON bytes

// Custom config for type mapping
@Bean
public ProducerFactory<String, Object> producerFactory() {
    Map<String, Object> config = new HashMap<>();
    config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
    config.put(JsonSerializer.TYPE_MAPPINGS, 
        "order:com.example.OrderEvent,user:com.example.UserEvent");
    return new DefaultKafkaProducerFactory<>(config);
}
```

---

## 11. Kafka with Microservices

```
Order Service → "order-events" topic → Payment Service
                                     → Inventory Service
                                     → Notification Service
                                     → Analytics Service

Each service has its own consumer group.
Each processes events independently.
Services are decoupled! ✅
```

---

## 12. Kafka Streams

```java
// Real-time stream processing

@Configuration
@EnableKafkaStreams
public class KafkaStreamsConfig {
    
    @Bean
    public KStream<String, OrderEvent> orderStream(StreamsBuilder builder) {
        KStream<String, OrderEvent> stream = builder.stream("order-events");
        
        // Count orders per user in real-time
        stream
            .filter((key, event) -> "ORDER_CREATED".equals(event.getType()))
            .groupBy((key, event) -> event.getUserId())
            .count(Materialized.as("order-count-store"))
            .toStream()
            .to("user-order-counts");
        
        return stream;
    }
}
```

---

## 13. Testing Kafka

```java
@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = {"order-events"})
class OrderProducerTest {
    
    @Autowired
    private OrderProducer producer;
    
    @Autowired
    private EmbeddedKafkaBroker kafkaBroker;
    
    @Test
    void shouldSendAndReceiveEvent() throws Exception {
        OrderEvent event = new OrderEvent("ORDER_CREATED", "order-1", "user-1", 
                                           BigDecimal.TEN, LocalDateTime.now());
        
        producer.sendOrderEvent(event);
        
        // Verify with consumer
        Consumer<String, String> consumer = configureConsumer();
        ConsumerRecords<String, String> records = 
            KafkaTestUtils.getRecords(consumer, Duration.ofSeconds(5));
        
        assertThat(records.count()).isEqualTo(1);
    }
}
```

---

## 14. Monitoring & Operations

```
Key Kafka Metrics:
  - Consumer lag: How far behind consumers are
  - Throughput: Messages/second
  - Partition count: Even distribution
  - Replication: All replicas in sync

Tools:
  - Kafka UI (Conduktor, AKHQ)
  - Prometheus + Grafana
  - Confluent Control Center
```

---

## 15. Best Practices

1. **Use keys for ordering** — Same key → same partition → ordered
2. **Idempotent consumers** — Handle duplicate messages gracefully
3. **Dead letter queues** — Don't lose failed messages
4. **Set proper retention** — Balance storage vs replay capability
5. **Right-size partitions** — More = more parallelism, but more overhead
6. **Use consumer groups** — Parallel processing within a service
7. **Monitor consumer lag** — Alert when consumers fall behind
8. **Use Avro/Protobuf** for production — Schema registry, versioning
9. **Manual offset commit** for critical processing — Don't lose messages
10. **Test with EmbeddedKafka** — Fast, reliable tests

---

## 16. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Kafka?** Distributed event streaming platform. Publish-subscribe messaging with persistent, ordered log.

**Q2: What is a topic?** Named channel for messages. Producers send to topics, consumers read from topics.

**Q3: What is a partition?** Topic subdivision for parallelism. Messages within partition are ordered.

**Q4: What is a broker?** Kafka server that stores messages. Cluster = multiple brokers.

**Q5: What is a consumer group?** Group of consumers sharing work. Each partition assigned to one consumer in group.

**Q6: What is an offset?** Position of message in a partition. Consumer tracks which offset it has processed.

**Q7: Producer vs Consumer?** Producer sends messages. Consumer reads messages.

**Q8: What is KafkaTemplate?** Spring abstraction for sending messages. Like JdbcTemplate for Kafka.

---

### Intermediate

**Q9: What is consumer lag?** Difference between latest offset and consumer's current offset. High lag = consumer falling behind.

**Q10: What is replication factor?** Number of copies of each partition. RF=3 means data on 3 brokers.

**Q11: What is a dead letter queue?** Topic for messages that failed processing after retries. For investigation/reprocessing.

**Q12: Kafka vs RabbitMQ?** Kafka: high throughput, persistent log, replay. RabbitMQ: flexible routing, lower latency, traditional queue.

**Q13: What is idempotent producer?** Ensures exactly-once delivery to broker. Prevents duplicates on retry.

**Q14: What is auto.offset.reset?** What to do when no offset exists. `earliest`: read from beginning. `latest`: read new messages only.

**Q15: What is KRaft?** Kafka Raft — replaces Zookeeper for cluster coordination. Built into Kafka.

---

### Rapid-Fire (Q16–Q50)

**Q16: Default Kafka port?** 9092.

**Q17: `@KafkaListener`?** Annotation that makes a method a Kafka consumer.

**Q18: `groupId` purpose?** Identifies consumer group. Same group = share work. Different group = independent copies.

**Q19: Message ordering guarantee?** Within a partition: ordered. Across partitions: no ordering.

**Q20: How to ensure ordering?** Use same key for related messages → same partition → ordered.

**Q21: What is acknowledgment?** Consumer confirming it processed a message. Offset committed.

**Q22: What is `enable.auto.commit`?** Auto-commit offsets periodically. Set false for manual control.

**Q23: What is `EmbeddedKafka`?** In-memory Kafka for testing. No external broker needed.

**Q24: What is retention period?** How long Kafka keeps messages. Default: 7 days.

**Q25: What is compaction?** Keep only latest value per key. Like a KV store.

**Q26: What is a leader partition?** Primary copy that handles reads/writes. Other copies are followers.

**Q27: What is ISR?** In-Sync Replicas. Followers caught up with leader.

**Q28: What is `acks=all`?** Wait for all replicas to acknowledge. Most durable.

**Q29: What is `acks=1`?** Wait for leader only. Faster but less durable.

**Q30: What is `acks=0`?** Don't wait. Fastest but may lose messages.

**Q31: How many partitions for a topic?** At least = max consumers in group. Typically 3-12 for most use cases.

**Q32: What is Kafka Connect?** Framework for connecting Kafka to external systems (databases, S3, etc.).

**Q33: What is Kafka Streams?** Library for real-time stream processing within Kafka.

**Q34: What is Schema Registry?** Central store for message schemas (Avro, Protobuf). Schema evolution.

**Q35: What is exactly-once semantics?** Each message processed exactly once. Achieved with idempotent producer + transactional consumer.

**Q36: What is at-least-once?** Message processed one or more times. Possible duplicates.

**Q37: What is at-most-once?** Message processed zero or one time. Possible data loss.

**Q38: Kafka message max size?** Default: 1MB. Configurable. Don't send large messages.

**Q39: What is ProducerRecord?** Object representing a message: topic, key, value, partition, headers.

**Q40: What is ConsumerRecord?** Object representing received message with metadata.

**Q41: What is `@SendTo`?** Spring annotation to forward consumed message to another topic.

**Q42: What is backpressure?** Consumer slower than producer. Handle with consumer scaling or rate limiting.

**Q43: Kafka vs Redis Pub/Sub?** Kafka: persistent, replay, consumer groups. Redis: ephemeral, faster, simpler.

**Q44: What is partition rebalancing?** Reassigning partitions when consumers join/leave group.

**Q45: What is sticky assignor?** Minimizes partition reassignment during rebalance.

**Q46: What is cooperative rebalancing?** Incremental rebalance without stop-the-world. Better than eager.

**Q47: Transaction in Kafka?** Atomic writes across multiple partitions/topics. `kafkaTemplate.executeInTransaction()`.

**Q48: How to replay messages?** Reset consumer offset to earlier position. Messages are still in Kafka.

**Q49: What is KSQL?** SQL interface for Kafka Streams. Query streams with SQL-like syntax.

**Q50: Kafka monitoring tools?** Confluent Control Center, AKHQ, Conduktor, Prometheus + Grafana.

---

## 📚 References

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Spring for Apache Kafka](https://docs.spring.io/spring-kafka/reference/)
- [Confluent Kafka Tutorials](https://developer.confluent.io/)
- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide-v2/)

---

> **Previous Topic:** [← 26 - Microservices](../26-microservices/README.md)  
> **Next Topic:** [28 - Linux →](../28-linux/README.md)
