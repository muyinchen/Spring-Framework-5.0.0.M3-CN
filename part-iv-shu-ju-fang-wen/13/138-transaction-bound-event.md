## 13.8Transaction bound event

As of Spring 4.2, the listener of an event can be bound to a phase of the transaction. The typical example is to handle the event when the transaction has completed successfully: this allows events to be used with more flexibility when the outcome of the current transaction actually matters to the listener.

Registering a regular event listener is done via the`@EventListener`annotation. If you need to bind it to the transaction use`@TransactionalEventListener`. When you do so, the listener will be bound to the commit phase of the transaction by default.

Let’s take an example to illustrate this concept. Assume that a component publish an order created event and we want to define a listener that should only handle that event once the transaction in which it has been published as committed successfully:

```java
@Component
   public class MyComponent {

   	@TransactionalEventListener
   	public void handleOrderCreatedEvent(CreationEvent<Order> creationEvent) {
     		...
   	}
   }
```

The`TransactionalEventListener`annotation exposes a`phase`attribute that allows to customize to which phase of the transaction the listener should be bound to. The valid phases are`BEFORE_COMMIT`,`AFTER_COMMIT`\(default\),`AFTER_ROLLBACK`and`AFTER_COMPLETION`that aggregates the transaction completion \(be it a commit or a rollback\).

If no transaction is running, the listener is not invoked at all since we can’t honor the required semantics. It is however possible to override that behaviour by setting the`fallbackExecution`attribute of the annotation to`true`.

