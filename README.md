## Kafka Tutorial For Beginners Everything You Need To Get Started

> Kafka is a highly scalable system for managing **event** logs

- https://www.youtube.com/watch?v=QkdkLdMBuL0 

**Without** Event Streaming

```
Order
        Payment 
                    Notification 
                                    Inventory 
                                                Analytics   
```

```
Visual Studio Code
Explorer
Open Editor
StartUp.js
```

StartUp.js
```Javascript
// Order Service ( Tightly Coupled, Synchronous )
async function processOrder(order) {
    try {
        // Sarah clicks "Buy Now" and waits...
        await inventoryService.checkStock(order.items);
        // 2 seconds later: Stock confirmed

        // Still waiting...
        await paymentService.processPayment(order.payment);
        // 3 seconds later: Payment processed

        // Sarah's gatting anxious...
        await notificationService.sendConfirmation(order);
        // 1 second later: Email sent

        // By now, Sara's wonderling if her order went through
        await analyticsService.trackOrder(order);

    } catch {
        (error) {
            // After all that waiting, if any service failed, Sarah sees "Something went wrong, please try again"
            throw new Error('Order processing failed');
        }
    }
}
```

#### Architecture can not handdle the volume  

> StartUp getting scaling in orders 

> Getting slow with services

**Tight** Coupling 

> Each service directly calls the other ( Synchronous Execution - Services operates in a linear, step-by-step manner )

If **Payments** goes down
```
Order
        /Payment/ 
                    Notification 
                                    Inventory 
                                                Analytics   

- API in the background is not responding 
- Service crashes over load
```
Or **Single** Points of Failure 

> A 10 minute **inventory** service outage means 2 hours of order backlogs

If **Inventory** goes down
```
Order
         Payment 
                    Notification 
                                    /Inventory/ 
                                                 Analytics
```

> Losing Analytics Data ( Analytics service can not record data )

If **Analytics** goes down
```
Order
         Payment 
                    Notification 
                                    Inventory 
                                               /Analytics/
```

#### Kafka Event Streaming Solution

> **Generates Event record** on purchase and hands over to kafka

```
Order                           Payment
                Broker
```

What is an **Event**

> An event **records the fact that somenthing happened** in your bussiness 

> Also callled "record" or "message"

> When you read or write data to kafka, you do this in the form of events

Event 
```
Event Key
Event Value 
    Timestamp
Opitional: Metadata
```
#### Producers

> Producers are those services that **publish ( write ) events to Kafka**

> Use **Producer API** to produce and send evets

```Javascript
const Kafka = new KafkaProducer({
    'bootstrap.servers': 'localhost:9092'
});

async function processOrder(order) {
    await kafka.produce({
        topic: 'new-orders',
        value: JSON.stringify(order)
    });

    return 'Order recivied ! Check your email for update.';
}
```

What is a **Topic** ?

> Events are **organized and durably stored in topics**

> Topics are categories or feed names to which records are published

> Order service will write Events to order Topics

> **You define your topics**, just like schema for database

> You decide **based on your application's architecture** and data flow requirements

```Java
// Define the topics
NewTopic ordersTopic = new NewTopic("orders", 3, (short) 1);
NewTopic inventoryTopic = new NewTopic("Inventory", 2, (short) 1);
NewTopic notificationsTopic = NewTopic("notifications", 4, (short) 1);

// Add topics to a list
List<NewTopic> topics = Arrays.asList(ordersTopic, inventoryTopic, notificationTopic);

// Create the topics
adminClient.createTopics(topics)
.all() // Ensures the operation completes
.get();
```

```
    Order Service 
                    Add Event to Order Topic 

                    Other actions are triggered by Event
                    > Updating stock in DB
                    > Sending notification to customer
                    > Updating sales status  
```

##### Consumers

> Consumers are those that **subcribe to ( read and proess ) the events** sent by producers

```
    Microservices

                Notifications
                Inventory
                Payment 

                             Subscribed to Order Topic  

                             Notification
                             Send confirmation email to customer
                             Send notification to department head

                             Inventory
                             Update inventory database
                                                    Generate event and add to inventory topic  

                             Payment
                             Generate invoice and send to user   
```

##### Real-Time Processing with Stream API

> Analytics 

When sells happen in the appplocation, you may have a sell dashboard where the servers are updating real time sells numbers

- Real-Time Metrics

- Personalized Recommendations 

- Website Activity Tracking 

- Fraud Detection

Kafka Streams API 

> The Streaming API allows you to **create real-time apps** by **continously** transforming and analysing income data streams

```
Kafka Producer
                Topic           Kafka Consumer ( Will process one event at a time  )
                                    - Notification Service ( Will read a order event ) / Will send an email or notification to the customer
```

Stream

```
Stream
        Key and Value -> Key and Value -> Key and Value...
```

> Think of a stream as an **continous real-time flow of records ( key-value pairs )

> You do not need to explicitly request new records, you just receive them

> Provides higher-level functions to process event streams, like **transformation and statefull operations** like aggregations ( counts, averages, sums ) and joins

**Kafka Streams API** is a library you embed in your app to perform stream processing

```Javascript
const builder = new StreamsBuilder();

// Read from our existing orders topic
const orders = builder.stream('new-orders');

// Calculate real-time revenue by category
const revenueByCategory = orders
    .groupBy((key, order) => order.category)
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
    .aggregate(
        ( => 0,
        (key, order, total) => total + order.amount
        )
    );

// Detect trending products
const trendingProducts = orders
    .groupBy((key, order) => order.productId)
    .windowedBy(TimeWindows.of(Duration.ofHours(5)))
    .count()
    .filter((key, count) => count > 100);

// Output to different topics for dashboards
revenueByCategory.to('revenue-metrics');
trendingProducts.to(trending-products);
```

#### Partitions for Scalability and Performance 

> This are streams of constant data saved as events in Kafka 

Partitions 

- A lot of data processed, easy to handdle and process without compromise the performance

> **Scalability**: Distribute data across multiple Kafka brokers

> **Paralellism**: Allows Concurrent message processing

> **Ordering**: Guarantees order within a Partition

> **Fault Tolerance**: Replication and leader failover

> **Logical Grouping**: Groups related data for efficint processing.

To add sections and **Partions** will add more "workers" per sections to help out in designed sections. 

Partitions 
```

                   EU
                   US
                   Asia

                   Orders
```

Consumer Groups for Scalable Message Consumption

> Additional instances of microservices, like replicas in K8s, they can all consume from Kafka partitions and process events faster in parallel 

```
    Consumer Group 1
    Consumer Group 2
    Consumer Group 3
```

Who does Kafka know which consumers form a group, and how to be bid, and each one belongs together ?

Grouped by the group ID Attribute 
```Javascript
const inventoryConsumer = new KafkaConsumer({
    groupId: 'inventory-team',
});
```

When start the replicas

> Kafka **distributes load automatically 

Partitions 
```

                   EU             Consumer 1
                   US             Consumer 2 ( Replica )
                   Asia           Consumer 3 ( Replica )

                   Orders
```

#### Kafka Brokers

What is a Kafka Broker

> Server that **store data** ( Message ) in topics, **manages message** distribution to consumers

```
Kafka Broker 1
  Topic A
Partition 0

Kafka Broker 2
  Topic A   
Partition 1

Kafka Broker 3
  Topic A
Partition 2

Kafka Cluster
```

> Topic's partitions are **distributed across multiple brokers**

> Each partition has a **leader broker** and multiple replicas

> Stores messages on disk for a configurable **retention period** ( Log )

> This enable **real time data processing** 

> Consumers can **read multiple times** and **whenever they want**

```
Kafka Broker 1
  Topic A
Partition 0

Kafka Broker 2 
  Topic A          Topic A      Topic A
Partition 1      Partition 0   Partition 1

Kafka Broker 3
  Topic A
Partition 2

Kafka Cluster
```

#### Zookeeper and KRaft

> Zookeeper is a centralized service for managing metadata and **coordination tasks for distributed systems**

> External dependency of Kafka

Cluster Management 

> Maintaining **registry** off all active brokers in the cluster

Leader Election

> Zookeeper facilitates the election of the leader 