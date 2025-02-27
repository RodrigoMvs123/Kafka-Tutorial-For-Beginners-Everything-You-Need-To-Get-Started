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

