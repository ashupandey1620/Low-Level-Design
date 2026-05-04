[00:00:00]  
**Introduction to Factory Design Pattern**  
- The lecture introduces the **Factory Design Pattern**, highlighting it as one of the most widely used design patterns in **Low-Level Design (LLD)**, especially crucial for interviews and real-world applications.  
- It is compared in importance to the **Strategy Design Pattern**, emphasizing understanding it well for both interview and practical use cases.  
- The Factory pattern conceptually relates to a real-world factory that creates objects or products. In software terms, a **factory is a class responsible for creating objects**.  

[00:00:56]  
**Why Factory Design Pattern is Needed**  
- Revisits the **Strategy Design Pattern** covered previously:  
  - There is a client and multiple interchangeable algorithms (strategies).  
  - Example: A Robot class with multiple behaviors (talkable, walkable, flyable).  
  - The Robot receives these strategy objects dynamically via dependency injection (composition).  
- The key point: The Robot class assumes these strategy objects are **already created elsewhere** and just calls their methods.  
- This raises the question: **Where and how are these objects created?**  
- The Factory Design Pattern addresses **separating object creation logic from business logic**, improving code readability and maintainability.  

[00:03:26]  
**Separation of Business Logic and Object Creation Logic**  
- Business logic is the core functionality of an application (e.g., in a notification system, deciding which notification to send).  
- Object creation logic involves deciding **when and which objects to instantiate**.  
- Mixing both in the same place makes code complex and harder to maintain.  
- Factory Design Pattern introduces a **separate factory class responsible solely for object creation**, decoupling object instantiation from the client/business logic.  
- The client simply **requests an object from the factory** without knowing how it is created.  
- This promotes **decoupling the client from the creation process**, a core design principle in software engineering.  

[00:05:20]  
**Types of Factory Patterns**  
- There are **three types** of factories:  
  1. **Simple Factory** (not a full design pattern but a design principle)  
  2. **Factory Method** (a proper design pattern)  
  3. **Abstract Factory Method** (an extended version of Factory Method)  
- The lecture proceeds to explain each type starting with the Simple Factory.  

[00:05:46]  
**Simple Factory Pattern Explained with Burger Shop Example**  
- The **product** is a `Burger` class with an abstract method `prepare()`.  
- Three concrete burger types: **Basic Burger, Standard Burger, Premium Burger**. Each overrides the `prepare()` method with specific preparation steps.  
- The problem: Client code might instantiate burgers with `new` keyword directly, which can get messy if burger types increase or change.  
- Solution: Create a **BurgerFactory class** with a method `createBurger(type: string)` that returns a `Burger` object based on the type parameter (`basic`, `standard`, `premium`).  
- The factory method uses conditional logic (`if-else`) to instantiate the correct burger type and returns it to the client.  
- This **encapsulates object creation and allows the client to simply request the desired burger type** without knowing the instantiation details.  

[00:09:19]  
**Simple Factory UML and Definition**  
- UML Diagram:  
  - Abstract Product: `Burger` (abstract class/interface)  
  - Concrete Products: `BasicBurger`, `StandardBurger`, `PremiumBurger`  
  - Factory: `BurgerFactory` which **has-a** relationship with Burger and decides which concrete burger to instantiate.  
- Definition:  
  - A **Simple Factory is a class that decides which concrete class to instantiate based on input parameters**.  
- Limitation: Simple Factory is a design principle, **not a formal design pattern**.  

[00:12:57]  
**Factory Method Pattern**  
- Extends the Simple Factory by introducing:  
  - An **abstract factory class** with an abstract method `createBurger()`.  
  - Multiple concrete factories override this method to create different burger types.  
- Example:  
  - Two burger shops (factories): **SyncBurger and KingBurger**.  
  - `SyncBurger` creates normal burgers (basic, standard, premium).  
  - `KingBurger` creates wheat-based healthy burgers (basic wheat, standard wheat, premium wheat).  
- The client can dynamically select which factory to use (either `SyncBurger` or `KingBurger`), and the factory decides which concrete burger to instantiate.  
- This introduces **polymorphism on the factory side**, allowing multiple factory implementations for different product families.  

[00:16:26]  
**Factory Method UML and Client Usage**  
- UML Diagram:  
  - Abstract Factory: `BurgerFactory` with abstract `createBurger()`  
  - Concrete Factories: `SyncBurgerFactory`, `KingBurgerFactory` overriding `createBurger()`  
  - Abstract Product: `Burger` with multiple concrete products (normal burgers, wheat burgers)  
- Client creates a factory object based on desired burger type and calls `createBurger()`.  
- The factory returns the appropriate burger object, hiding creation details from the client.  

[00:21:53]  
**Abstract Factory Pattern**  
- Further extension of Factory Method pattern.  
- The abstract factory now creates **families of related products**, not just one product type.  
- Example:  
  - In addition to burgers, the burger shops also make **garlic bread**.  
  - So the abstract factory defines two abstract methods: `createBurger()` and `createGarlicBread()`.  
  - Concrete factories (`SyncBurger`, `KingBurger`) override both methods to produce related products (burgers and garlic breads) in their style.  
- UML Diagram shows:  
  - Two product categories: `Product A` (burger), `Product B` (garlic bread), each with their concrete implementations  
  - The abstract factory interface and multiple concrete factories handling these product families  
- Definition:  
  - **Abstract Factory provides an interface for creating families of related or dependent objects without specifying their concrete classes**.  

[00:27:45]  
**Practical Example: Notification System**  
- Factory pattern can be applied to a **notification system** where notifications can be of different types: SMS, Email, Push.  
- The factory decides which notification object to create based on input type.  
- Each notification type is a subclass of an abstract notification class and implements the sending functionality.  
- Comparison with Strategy Pattern:  
  - Strategy pattern can also be used if the intent is to **vary algorithms (notification sending methods) at runtime** by using composition and dependency injection.  
  - Factory pattern is preferred if the intent is to **separate object creation logic from business logic**, especially when the client should not know how objects are created.  
- Key insight: **Multiple design patterns can solve the same problem differently**. The choice depends on the specific scenario and design intent.  
- A practical rule of thumb:  
  - Use **Strategy** if you want to vary algorithms dynamically at runtime and objects are already created.  
  - Use **Factory** if you want to separate the creation of objects from business logic and manage object instantiation centrally.  

[00:31:34]  
**Conclusion**  
- Understanding Factory Design Pattern is essential for clean architecture and code maintainability.  
- Recognizing when to use Factory versus Strategy is important for effective design.  
- The lecture hopes to have clarified these concepts and promises more design pattern tutorials in future sessions.  

---

### Summary Table: Types of Factory Patterns

| Factory Type           | Description                                                   | Example Use Case                         | Key Characteristic                          |
|-----------------------|---------------------------------------------------------------|----------------------------------------|---------------------------------------------|
| **Simple Factory**     | Single class deciding which concrete class to instantiate.    | BurgerFactory creates Basic/Standard/Premium burgers | Conditional logic inside factory method.    |
| **Factory Method**     | Abstract factory class with multiple concrete factories.      | SyncBurgerFactory vs KingBurgerFactory | Polymorphic factories creating different product variants. |
| **Abstract Factory**   | Factory creates families of related products (multiple types).| Factory creates burgers & garlic breads| Factory interface with multiple creation methods.|

---

### Key Insights  
- **Factory Design Pattern decouples object creation from client code, improving maintainability and flexibility.**  
- **Simple Factory is a design principle, while Factory Method and Abstract Factory are formal design patterns.**  
- **Factory Method uses polymorphism on the factory level, enabling multiple factory implementations.**  
- **Abstract Factory manages families of related objects, useful when multiple product types need coordinated creation.**  
- **Strategy and Factory patterns can sometimes solve overlapping problems; choice depends on whether you want to vary algorithms or object creation.**  
- **Real-world analogy (burger shops) clarifies the differences and benefits of each pattern.**  
- **In practice, separating business logic from object creation logic reduces code complexity and enhances clarity.**

---

### Code Design Outline (Burger Example)

| Component          | Role                                               | Key Methods                    | Details                                      |
|--------------------|----------------------------------------------------|--------------------------------|----------------------------------------------|
| Burger (abstract)   | Product interface/class                             | `prepare()` (abstract)          | Extended by BasicBurger, StandardBurger, PremiumBurger, etc. |
| Concrete Burgers    | Specific product implementations                   | override `prepare()`            | Define burger preparation details.           |
| BurgerFactory (Simple Factory) | Creates burger object based on type string   | `createBurger(type: string)`    | Uses conditional logic to instantiate burger.|
| BurgerFactory (Abstract) | Abstract factory interface for burger creation | `createBurger()` (abstract)     | Extended by concrete burger factories.       |
| Concrete Factories  | Create specific burger types                        | override `createBurger()`        | E.g., SyncBurgerFactory, KingBurgerFactory.  |
| Abstract Factory    | Interface for creating multiple product types      | `createBurger()`, `createGarlicBread()` | Extended by concrete factories creating families. |

---

### FAQ

**Q1: Why separate object creation logic using Factory Pattern?**  
A1: To decouple business logic from object creation, making the code easier to maintain, test, and extend without modifying client code.

**Q2: How is Factory Method different from Simple Factory?**  
A2: Factory Method uses polymorphism and abstract factories, allowing multiple factory classes to decide which object to instantiate, while Simple Factory uses conditional logic in a single class.

**Q3: When to use Abstract Factory?**  
A3: When you need to create related families of products (e.g., burgers and garlic bread), ensuring consistency among products created by a factory.

**Q4: Can Strategy and Factory be used interchangeably?**  
A4: Not exactly; Strategy is for varying algorithms dynamically, assuming objects exist; Factory is for controlling object creation. The choice depends on the intent and design needs.

**Q5: Is Simple Factory a design pattern?**  
A5: No, it's a design principle that simplifies object creation but lacks formal pattern status.

---

This summary captures the full depth and examples of the Factory Design Pattern lecture, grounded strictly in the transcript content.