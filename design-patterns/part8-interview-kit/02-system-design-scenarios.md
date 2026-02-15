# System Design Scenarios

> **Part 8: Interview Kit**  
> **Status:** Applying Patterns to Reality

---

## Scenario 1: Universal Notification Service
**Requirement**: Build a service that sends notifications via Email, SMS, Push, and Slack. It must support user preferences and be easily extensible.

### Pattern Selection
1.  **Factory Pattern**: `NotificationFactory.create("EMAIL")` returns `EmailNotifier`.
2.  **Strategy Pattern**: `Notifier` interface. Implementations: `EmailStrategy`, `SmsStrategy`. Allows swapping logic.
3.  **Observer Pattern**: When `OrderPlaced` event occurs, the `NotificationListener` triggers the Strategy.
4.  **Adapter Pattern**: Connect to `Twilio` (SMS) and `SendGrid` (Email) utilizing 3rd party SDKs wrapped in our Interface.

---

## Scenario 2: Payment Gateway Integration
**Requirement**: Integrate with Stripe, PayPal, and Square. If one fails, try another. Minimize complexity for the core app.

### Pattern Selection
1.  **Facade Pattern**: Create a `PaymentFacade`. Client just calls `facade.pay(100)`. Facade handles the complexity.
2.  **Adapter Pattern**: Standardize the APIs. `StripeAdapter` converts our `PayRequest` to Stripe's format.
3.  **Circuit Breaker**: If PayPal is down (timeout), trip the breaker and route traffic to Stripe automatically.
4.  **Retry Pattern**: If a network blip occurs, retry once before failing.

---

## Scenario 3: Real-Time Multiplayer Game
**Requirement**: 100 players in a customized map. Shooting, movement, inventory. Low latency.

### Pattern Selection
1.  **Flyweight Pattern**: The Map has 1,000 trees. Store the "Tree Texture" once. Share it across 1,000 tree instances to save RAM.
2.  **State Pattern**: Player state (`Running`, `Jumping`, `Shooting`). Avoid `if(state == RUNNING)`.
3.  **Command Pattern**: Every key press is a `Command`. Send `MoveCommand` to server. Allows "Replay" feature by re-executing commands.
4.  **Prototype Pattern**: Spawning enemies. Clone a "Grunt" prototype instead of initializing a new one from scratch.

---

## Scenario 4: Legacy Banking Migration
**Requirement**: Move from Mainframe to Microservices without downtime.

### Pattern Selection
1.  **Strangler Fig Pattern**: Put a Gateway in front. Route "New Accounts" to Microservice. Route "Old Accounts" to Mainframe.
2.  **Anti-Corruption Layer (ACL)**: The Microservice should NOT use the Mainframe's COBOL data structure. Translate it at the boundary.
3.  **Facade**: Hide the Mainframe's complexity behind a clean Java Interface.
