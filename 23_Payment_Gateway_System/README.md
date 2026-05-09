### Summary

This lecture focuses on the **Low-Level Design (LLD) of a Payment Gateway System**, which acts as an intermediary between client applications and various payment providers like Paytm, Razorpay, Google Pay, etc. The session explains how to design a scalable, extensible, and robust payment gateway that can be integrated into any application (e.g., Amazon, Zomato) to handle payments efficiently.

---

### Key Concepts and Design Requirements

- **Payment Gateway System**: A plug-and-play intermediary module integrated into client applications to process payments through multiple third-party providers.
- **Difference from Banking System**: The payment gateway handles request flow and communication, whereas the banking system manages actual fund transfer and internal architecture.
- **Multiple Providers Support**: The system must support multiple payment providers (Paytm, Razorpay, etc.) and allow easy addition of new providers in the future.
- **Standardized Payment Flow**: Every payment should follow a strict sequence of three steps:
  1. **Validate** the payment request (check funds, currency, request validity).
  2. **Initiate** the payment (send request to provider).
  3. **Confirm** the payment (get confirmation if payment succeeded).
- **Error Handling and Retry Mechanism**: The gateway should handle errors gracefully and retry failed payments a configurable number of times before declaring failure.
- **Design Patterns Used**:
  - **Template Method Pattern**: To standardize the payment flow across different providers while allowing provider-specific implementations of each step.
  - **Proxy Pattern**: To introduce a proxy layer (gateway proxy) that wraps real provider gateways adding retry logic and error handling without altering provider classes.
- **Plug-and-Play Architecture**: The gateway offers a single entry point (controller) that client applications interact with, hiding complexity internally.
- **SOLID Principles**: The design follows SOLID principles for maintainability and extensibility.

---

### System Architecture and Classes

| Component          | Description                                                                                         |
|--------------------|-------------------------------------------------------------------------------------------------|
| **PaymentRequest** | Model class containing payment details: sender, receiver, amount (double), currency (string).     |
| **IBankingSystem** | Interface with method `processPayment(amount)` returning boolean success/failure.                 |
| **PaytmBankingSystem, RazorpayBankingSystem** | Concrete implementations of `IBankingSystem` simulating external banking APIs with different success rates. |
| **PaymentGateway (Abstract)** | Defines the template method `processPayment(request)` with abstract `validate()`, `initiate()`, and `confirm()` methods to be implemented by subclasses. |
| **PaytmGateway, RazorpayGateway** | Concrete gateway classes implementing provider-specific validation, initiation, and confirmation. Each holds a reference to its respective banking system. |
| **PaymentGatewayProxy** | Implements `PaymentGateway`, holds a real gateway object, adds retry mechanism to `processPayment()`, delegates other methods to the real gateway. |
| **GatewayFactory** | Singleton factory class that creates and returns proxies wrapping specific provider gateways based on an enum `GatewayType` (Paytm, Razorpay). |
| **PaymentService** | Singleton service class holding a reference to a `PaymentGateway` and exposing methods to set gateway and process payments. Handles business logic. |
| **PaymentController** | Singleton controller class exposing a single entry point `handlePayment(gatewayType, paymentRequest)`. It uses `GatewayFactory` and `PaymentService` to execute payments. |

---

### Payment Flow (Happy Path)

| Step       | Component                  | Action                                                                                     |
|------------|----------------------------|--------------------------------------------------------------------------------------------|
| 1. Client | Client application          | Creates a `PaymentRequest` with sender, receiver, amount, currency.                        |
| 2. Controller | `PaymentController`       | Receives request and gateway type, requests gateway proxy from `GatewayFactory`.            |
| 3. Service | `PaymentService`            | Sets the gateway obtained from factory and calls `processPayment()` on it.                 |
| 4. Proxy  | `PaymentGatewayProxy`        | Applies retry logic while delegating calls to real gateway.                                |
| 5. Real Gateway | `PaytmGateway` or `RazorpayGateway` | Executes `validate()`, `initiate()`, and `confirm()` in order, delegating payment to banking system. |
| 6. Banking System | `PaytmBankingSystem` or `RazorpayBankingSystem` | Processes payment and returns success/failure (simulated with random success rates).        |
| 7. Result | Proxy -> Service -> Controller -> Client | Returns final success/failure to client after retries if necessary.                       |

---

### Retry Mechanism Details

- The **proxy class** wraps the real gateway and implements the retry logic in `processPayment()`.
- It attempts payment processing multiple times (configurable retries) until success or retry limit reached.
- Retries are internal to the proxy, abstracted away from clients and gateways.
- This avoids duplicating retry logic in each provider gateway or burdening clients with retry policies.
- The retry count differs per provider (e.g., Paytm retries 3 times, Razorpay 1 time).

---

### UML and Relationships

| Relationship Type     | Between                        | Explanation                                                                                     |
|-----------------------|-------------------------------|------------------------------------------------------------------------------------------------|
| **Inheritance (Is-A)** | `PaytmGateway`, `RazorpayGateway` inherit from `PaymentGateway` | Implement provider-specific payment steps.                                                     |
| **Composition (Has-A)** | `PaymentGateway` has a banking system reference (proxy of `IBankingSystem`) | Strong relationship; banking system accessed via proxy over internet (external service).        |
| **Proxy Pattern**      | `PaymentGatewayProxy` wraps `PaymentGateway` | Adds retry and error handling transparently.                                                    |
| **Singleton**          | `GatewayFactory`, `PaymentService`, `PaymentController` | Single instances throughout the app lifecycle for centralized management.                      |
| **Factory Pattern**    | `GatewayFactory` creates gateways | Uses enum `GatewayType` to instantiate appropriate gateway proxies.                             |
| **Controller-Service Pattern** | `PaymentController` delegates to `PaymentService` | Separation of concerns: controller handles request routing, service handles business logic.    |

---

### Quantitative Data: Provider Success Rates and Retry Counts

| Provider          | Success Rate | Retry Count |
|-------------------|--------------|-------------|
| **Paytm**         | 20% (randomly simulated) | 3           |
| **Razorpay**      | 90% (randomly simulated) | 1           |

---

### Code and Implementation Highlights

- **PaymentRequest** holds basic payment info: sender, receiver, amount ($\text{double}$), currency (string).
- **Banking systems** simulate success/failure using randomization.
- The **template method** `processPayment()` enforces the sequence: validate $\to$ initiate $\to$ confirm.
- **Proxy's `processPayment()`** implements retry loop calling real gateway's process method.
- **Factory** ensures client code does not depend on concrete gateway implementations.
- **Controller-Service layers** provide clean API and business logic separation.
- Error handling logs errors and throws exceptions if validation/initiation/confirmation fails.
- Retry mechanism is encapsulated in proxy, adhering to **Single Responsibility Principle**.

---

### Homework / Extensions Suggested

- Implement **scalable retry strategies** beyond simple linear retries, such as:
  - **Exponential backoff retry**
  - Different retry policies per provider
- Integrate **recurring payment flow** (subscription payments) using design patterns for extensibility.
- These extensions deepen understanding of template methods and scalable design.

---

### Key Insights

- **Payment Gateway acts as a modular, extensible middleware for payment processing across providers.**
- **Template Method Pattern enforces standardized payment workflow while allowing custom provider logic.**
- **Proxy Pattern centralizes retry and error handling, keeping provider classes focused on payment logic.**
- **Factory Pattern and Singleton ensure flexible, singleton instances and decoupling from concrete implementations.**
- **Separation of concerns via Controller-Service layers promotes clean architecture and maintainability.**
- The design accommodates future providers and complex retry policies without impacting clients.

---

This lecture provides a comprehensive end-to-end LLD for a payment gateway system, covering requirements, architecture, design patterns, and example code with detailed explanations, enabling learners to implement scalable, robust payment integrations.