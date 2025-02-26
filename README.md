## Kafka Tutorial For Beginners Everything You Need To Get Started

- https://www.youtube.com/watch?v=QkdkLdMBuL0 

**Without** Event Streaming

```
Order
        Payment 
                    Notification 
                                    Inventory 
                                                Analytics   
```

Visual Studio Code
Explorer
Open Editor
StartUp.js

StartUp.js
```javascript
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

