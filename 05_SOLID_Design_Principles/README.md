[00:00:00]  
### Introduction to SOLID Design Principles  
- The lecture begins with a **recap of OOP fundamentals** covered so far: *Abstraction, Encapsulation, Inheritance, Polymorphism*, class and object creation, data hiding, and data security.  
- Real-world projects involve thousands of classes, making management complex. Without proper design principles, this leads to **tight coupling** and **messy code**, resulting in difficult bug fixes and feature integrations.  
- Bugs in real applications cause monetary and technical losses, emphasizing the need for **clean, maintainable code and architecture**.  
- The lecture introduces **SOLID Design Principles** as a set of rules to manage code complexity and maintainability.  
- An analogy is given: A house with many tangled wires (electrical, internet, pipes) all coming from one place is hard to debug and maintain—similarly, tightly coupled classes in software are problematic.  

[00:02:38]  
### Common Problems in Real-World Projects  
- **Maintainability issues**: Difficulty in integrating new features without extensive code changes or introducing bugs.  
- **Readability problems**: New developers find it hard to understand tightly coupled, complex code, increasing onboarding time.  
- **Bug-proneness**: More bugs are introduced and debugging consumes excessive time.  
- SOLID principles aim to solve these problems by keeping code clean and architecture manageable.  
- Credit is given to **Robert C. Martin (Uncle Bob)**, who introduced these principles in 2000.  

[00:04:52]  
### Overview of SOLID Principles  
- SOLID is an acronym representing five key design principles:  
| Letter | Principle Name                    | Acronym | Description                              |
|--------|---------------------------------|---------|------------------------------------------|
| S      | Single Responsibility Principle | SRP     | Class should have only one reason to change. |
| O      | Open-Closed Principle            | OCP     | Classes open for extension, closed for modification. |
| L      | Liskov Substitution Principle   | LSP     | Subclasses should be substitutable for their base classes. |
| I      | Interface Segregation Principle | ISP     | Clients should not be forced to depend on interfaces they do not use. *Not covered in this video* |
| D      | Dependency Inversion Principle   | DIP     | Depend on abstractions, not on concretions. *Not covered in this video* |

- The lecture will cover each principle in-depth with practical examples starting from SRP.  

[00:06:37]  
### Single Responsibility Principle (SRP)  
- **Definition:** A class should have only one reason to change or handle only one responsibility.  
- Simplified: A class does exactly one thing.  
- **Real-world analogy:** A TV remote controls only the TV. If it also controls the fridge or AC, it becomes complex and hard to maintain.  
- **Implication:** All attributes and methods of a class should align with one responsibility only.  

[00:08:58]  
### SRP Example: Shopping Cart & Product Classes  
- A **Product class** represents items in an e-commerce site with minimal attributes: product name and price.  
- A **ShoppingCart class** maintains a list of multiple products (one-to-many relationship).  
- ShoppingCart methods discussed:  
  - `calculateTotalPrice()` sums all product prices.  
  - `printInvoice()` prints the invoice for all products.  
  - `saveToDB()` stores cart data in the database.  
- Problem: The ShoppingCart class violates SRP by handling multiple responsibilities: price calculation, invoice printing, and database storage.  
- Consequences: Any change in invoice printing or database logic requires changing ShoppingCart, leading to multiple reasons for change and reduced maintainability.  

[00:13:18]  
### Applying SRP via Class Decomposition  
- Solution: Break down ShoppingCart responsibilities into multiple classes:  
  - Keep ShoppingCart responsible only for **calculating total price**.  
  - Create a **ShoppingCartInvoicePrinter** class responsible solely for printing invoices.  
  - Create a **ShoppingCartDBStorage** class responsible solely for database persistence.  
- Both helper classes hold references to ShoppingCart (has-a relationship).  
- This decomposition ensures each class has a **single responsibility**, improving maintainability and ease of changes.  

[00:17:22]  
### Code Walkthrough: SRP Violation and Compliance  
- Initial code shows all functions in ShoppingCart violating SRP: add/get products, calculate total price, print invoice, save to DB—all in one class.  
- Main function creates ShoppingCart, adds products, prints invoice, and saves to DB; this works but breaks SRP.  
- Refactored code removes invoice printing and DB persistence from ShoppingCart into separate classes.  
- ShoppingCart now only calculates total price and manages products.  
- InvoicePrinter and DBStorage classes each have a reference to ShoppingCart and handle their respective functions.  
- This structure adheres strictly to SRP, enabling independent modifications without affecting other responsibilities.  

[00:20:33]  
### Clarification on SRP Misconceptions  
- SRP does **not** mean a class can have only one method.  
- A class can have multiple methods as long as they serve the **same single responsibility**.  
- For example:  
  - ShoppingCart’s total price calculation might require multiple helper methods, all contributing to price calculation responsibility.  
  - Similarly, DBStorage might have multiple methods for storing data, all aligned with persistence responsibility.  

[00:22:55]  
### Open-Closed Principle (OCP)  
- **Definition:** A class should be **open for extension but closed for modification**.  
- Meaning: You can add new features without changing existing code.  
- Changing existing code risks breaking tested functionality; OCP encourages extension via new code without modifying existing classes.  
- Example problem: Adding new database storage types by modifying existing DBStorage class with new methods (e.g., saveToMongo(), saveToFile()) violates OCP.  

[00:26:27]  
### Applying OCP Using Abstraction, Inheritance, and Polymorphism  
- Solution: Introduce an **abstract class or interface** `DBPersistence` with a pure virtual method `save()`.  
- Concrete classes like `SQLPersistence`, `MongoPersistence`, and `FilePersistence` inherit from `DBPersistence` and implement their own `save()` methods.  
- The client code interacts with `DBPersistence` references, allowing new persistence types to be added without modifying existing classes.  
- This uses OOP concepts: **abstraction (interface), inheritance, polymorphism (method overriding)** to achieve OCP compliance.  

[00:35:44]  
### Code Walkthrough: OCP Violation and Compliance  
- Initial code violating OCP has all persistence methods in one class with multiple save methods (SQL, Mongo, File).  
- Refactored code creates:  
  - An abstract class `DBPersistence` defining a virtual `save()` method.  
  - Concrete classes `SQLPersistence`, `MongoPersistence`, `FilePersistence` override `save()` with specific logic.  
- Main function creates instances of each concrete persistence class and calls `save()`, demonstrating polymorphism.  
- This design eliminates modification of existing classes on feature addition, satisfying OCP.  

[00:39:55]  
### Liskov Substitution Principle (LSP)  
- **Definition:** Subclasses must be substitutable for their base classes without affecting correctness.  
- Stated as: *“Subclasses should be substitutable for their base classes.”*  
- It is a fundamental property of inheritance but often violated in practical scenarios.  
- **Key idea:** Any code using base class objects must work correctly when passed subclass objects.  
- Subclasses should **extend**, not **narrow** or break, the behavior of the base class.  

[00:43:01]  
### LSP Explained with Base and Subclass  
- Given base class A with methods $m_1, m_2, m_3$ and subclass B inheriting A and adding $m_4, m_5$.  
- Clients expecting A should function identically when passed B objects, calling only $m_1, m_2, m_3$.  
- B extends A’s behavior but does not remove or alter expected functionalities.  
- Violations occur when subclasses fail to fulfill base class contracts.  

[00:46:42]  
### LSP Violation Example: Bank Accounts  
- Base abstract class: **Account** with methods `deposit()` and `withdraw()`.  
- Subclasses:  
  - **SavingsAccount** and **CurrentAccount** implement both methods.  
  - **FixedDepositAccount** inherits Account but logically **does not allow withdrawal**.  
- To handle this, FixedDepositAccount overrides `withdraw()` to throw an exception—this violates LSP since it breaks substitutability.  
- Clients unaware of this restriction may cause runtime errors when invoking `withdraw()` on FixedDepositAccount instances.  

[00:51:54]  
### Bad Practice to Handle LSP Violation  
- Client code is modified to check the account type before calling `withdraw()`, skipping it for FixedDepositAccount.  
- This introduces **type checking and tight coupling** in the client, breaking **Open-Closed Principle** and defeating abstraction purpose.  
- Clients should not be aware of subclass specifics or contain conditional logic based on subclass types.  

[00:55:27]  
### Correct Approach to Fix LSP Violation  
- The root cause: FixedDepositAccount does not fully implement the base class contract (withdraw method).  
- Solution: **Split the Account abstraction into two interfaces/abstract classes**:  
| Interface/Abstract Class           | Methods             | Description                          |
|----------------------------------|---------------------|------------------------------------|
| NonWithdrawableAccount (DepositOnly) | `deposit()`         | For accounts that only allow deposits. |
| WithdrawableAccount (extends NonWithdrawableAccount) | `deposit()`, `withdraw()` | For accounts supporting withdrawal. |

- FixedDepositAccount inherits NonWithdrawableAccount (deposit only).  
- SavingsAccount and CurrentAccount inherit WithdrawableAccount (deposit and withdraw).  
- Clients maintain **two separate lists**: one for withdrawable accounts, one for deposit-only accounts, and call appropriate methods accordingly.  
- This respects LSP by ensuring subclasses fully implement the interface contracts they inherit.  

[01:01:00]  
### Code Walkthrough: LSP Violation and Resolution  
- Initial code: Single Account interface with deposit and withdraw, subclasses including FixedDepositAccount throw exceptions on withdraw.  
- Client processes accounts uniformly, leading to exceptions and broken substitutability.  
- Refactored code:  
  - Split Account into two abstract classes/interfaces: DepositOnlyAccount and WithdrawableAccount (which extends DepositOnlyAccount).  
  - FixedDepositAccount inherits DepositOnlyAccount only.  
  - SavingsAccount and CurrentAccount inherit WithdrawableAccount.  
  - Client maintains two separate collections and calls only supported methods on each.  
- Result: No exceptions due to unsupported operations, full compliance with LSP.  

[01:07:31]  
### Summary and Next Steps  
- The lecture covered three SOLID principles in detail:  
  - **Single Responsibility Principle (SRP):** Classes should have one responsibility.  
  - **Open-Closed Principle (OCP):** Classes should be open for extension but closed for modification, achieved via abstraction and polymorphism.  
  - **Liskov Substitution Principle (LSP):** Subclasses must be substitutable for their base classes, requiring interface segregation when behavior differs.  
- The remaining two principles, **Interface Segregation Principle (ISP)** and **Dependency Inversion Principle (DIP)**, will be covered in the next lecture.  

---

**Key Insights:**  
- SOLID principles dramatically improve code maintainability, readability, and reduce bugs.  
- Violations of these principles often occur unintentionally but can be avoided by understanding and applying proper design patterns and OOP concepts like abstraction, inheritance, and polymorphism.  
- Real-world examples such as shopping cart management and banking systems effectively illustrate these principles and their common pitfalls.  
- Decomposing responsibilities and segregating interfaces are critical techniques for adhering to SOLID principles.  

**Terminology Table:**  

| Term                      | Explanation                                                                                 |
|---------------------------|---------------------------------------------------------------------------------------------|
| Tight Coupling             | When classes or modules are highly dependent on each other, making changes difficult.       |
| Abstraction               | Hiding complex implementation details and exposing only essential features via interfaces.  |
| Polymorphism              | Ability to call the same method on different objects, each responding in their own way.     |
| Inheritance               | Mechanism where one class inherits properties and behaviors from another.                    |
| Has-a Relationship        | Composition where one class contains references to other classes.                            |
| Is-a Relationship        | Inheritance where a subclass is a type of its superclass.                                   |
| Interface Segregation     | Splitting interfaces so clients only depend on what they actually use.                       |
| Dependency Inversion      | High-level modules should not depend on low-level modules; both should depend on abstractions.|

---

This summary captures the lecture’s content in a clear, structured, and professional format strictly grounded in the provided transcript without any unsupported information.