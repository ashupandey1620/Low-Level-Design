### Summary: Design and Implementation of a 10-Minute Delivery App Inventory Management System

This lecture presents a detailed **low-level design (LLD) and implementation** of an inventory management system for a fast 10-minute delivery app, similar to Zepto or Blinkit. The focus is on handling the unique challenges of quick commerce, where multiple dark stores serve orders rapidly, and inventory must be efficiently managed and distributed.

---

### Core Problem and Requirements

- **Quick commerce apps** must deliver items within 10 minutes, unlike regular food or e-commerce apps with longer delivery windows.
- Multiple **dark stores** (warehouses with inventory but no direct customer access) exist across locations.
- Not all items are available in every dark store, requiring:
  - Splitting orders across multiple dark stores.
  - Assigning multiple delivery partners accordingly.
- The system must support:
  - **Inventory management:** Add/remove items, check stock.
  - **Replenishment strategies:** Threshold-based, weekly, scalable for future strategies.
  - **User proximity-based product listings:** Show items only from dark stores within a certain radius (e.g., 5 km).
  - **Order fulfillment:** Either fully from one store or split among multiple stores and delivery agents.
- The design keeps **modularity and scalability** in mind, applying design patterns and separation of concerns.

---

### Key Classes and Components

| Component              | Description                                                                                                           | Key Methods/Attributes                                                                                     |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| **Product**            | Represents an item with unique SKU, name, and price.                                                                 | SKU (identifier), name, price                                                                                |
| **ProductFactory**     | Creates `Product` instances based on SKU.                                                                             | `createProduct(sku)`                                                                                         |
| **InventoryStore (abstract)** | Manages products and their quantities in a store (dark store). Abstract base class.                                   | `addProduct()`, `removeProduct()`, `checkStock()`, `listAllProducts()`                                       |
| **DBInventoryStore**   | Concrete implementation using in-memory maps to store products and stock quantities.                                  | Overrides abstract methods with map-based storage for efficiency                                            |
| **InventoryManager**   | Manages interaction with the `InventoryStore`, delegates add/remove/check operations, applies business logic.         | `addStock(sku, quantity)`, `removeStock(sku, quantity)`, `checkStock(sku)`, `listAvailableProducts()`       |
| **ReplenishStrategy (interface)** | Defines replenishment behavior, supporting different strategies.                                                     | `replenish(items, inventoryManager)`                                                                         |
| **ThresholdReplenishStrategy** | Replenishes stock when quantities fall below a threshold.                                                            | Holds threshold value, overrides `replenish()`                                                              |
| **WeeklyReplenishStrategy**    | Replenishes stock on a weekly schedule.                                                                              | Overrides `replenish()`                                                                                       |
| **DarkStore**          | Represents a physical dark store location with inventory managed by an `InventoryManager` and a replenishment strategy. | Name, coordinates (x, y), `distanceTo(userX, userY)`, `runReplenishment()`                                   |
| **DarkStoreManager (Singleton)** | Manages multiple dark stores, provides methods to find nearby stores within a radius.                                  | Stores list of `DarkStore`, `getNearbyStores(userX, userY, maxDistance)`, `registerDarkStore()`, etc.        |
| **User**               | Represents the app user with a name, location, and cart.                                                               | Name, coordinates, Cart                                                                                      |
| **Cart**               | Contains list of items user wants to order with quantities.                                                            | List of (Product, quantity), total amount, `addToCart()`, `removeFromCart()`, `getItems()`                    |
| **DeliveryAgent**      | Represents a delivery person assigned to fulfill parts of an order.                                                     | Name                                                                                                         |
| **Order**              | Represents a placed order including user, items, total amount, and assigned delivery agents.                            | OrderID, User, list of (Product, quantity), total amount, list of DeliveryAgents                              |
| **OrderManager (Singleton)** | Handles placement of orders by checking inventory availability across nearby dark stores and assigning delivery agents. | `placeOrder(user, cart, darkStoreManager)`, `getAllOrders()`, maintains list of orders                       |
| **Zepto (Orchestrator)** | Coordinates overall interaction, entry point for showing items and placing orders.                                        | `showAllItems(user)`, `initialize()`, delegates to other components                                         |

---

### Functional Flow Overview

- **User Interaction:**
  - User opens app, sees all items available within 5 km radius.
  - User adds items to cart and places an order.
- **Order Placement:**
  - OrderManager requests the DarkStoreManager for nearby dark stores.
  - Checks if the nearest dark store can fulfill all items.
    - If yes, assigns one delivery agent.
    - If no, splits order across multiple dark stores, assigning multiple delivery agents.
  - Updates inventory stock accordingly.
  - Calculates total order amount.
- **Inventory Management:**
  - InventoryManager delegates operations to concrete InventoryStore.
  - Supports adding/removing products and checking stock efficiently using maps (SKU to quantity and SKU to Product).
- **Replenishment:**
  - DarkStore triggers replenishment via the assigned replenishment strategy.
  - Strategies can be threshold-based or periodic (weekly), scalable for future extensions.
- **Delivery Assignment:**
  - Delivery agents are assigned per dark store fulfilling part of the order.
  - Agents currently modeled minimally (name only), to be extended.

---

### Design Patterns and Principles Applied

- **Factory Pattern:** For creating products (`ProductFactory`).
- **Singleton Pattern:** Used for managers like `DarkStoreManager` and `OrderManager` to ensure single instance across app.
- **Bridge Pattern:** Separation between abstraction (`InventoryManager`) and implementation (`InventoryStore`, `DBInventoryStore`) allowing flexibility.
- **Strategy Pattern:** For replenishment strategies enabling different replenishment mechanisms.
- **Composition over Inheritance:** Strong preference for composition (e.g., `DarkStore` has `InventoryManager`), promoting loose coupling.
- **Delegation & Least Knowledge Principle:** Managers delegate tasks to stores; classes interact only with immediate friends.
- **Modularity:** Clear separation of concerns for easier maintenance and extension.

---

### Quantitative Data: Example Inventory and Orders

| SKU   | Product   | Price (₹) | Dark Store A Qty | Dark Store B Qty | Dark Store C Qty |
|-------|-----------|-----------|------------------|------------------|------------------|
| 1001  | Apple     | 20        | 5                | 5                | 0                |
| 102   | Banana    | 10        | 2                | 0                | 5                |
| 103   | Chocolate | 50        | 0                | 10               | 0                |
| 2001  | T-Shirt   | 500       | 0                | 0                | 0                |

- Example order: User orders 4 Apples, 3 Bananas, 2 Chocolates.
- Fulfillment:
  - Dark Store A provides 4 Apples, 2 Bananas.
  - Dark Store C provides 1 Banana.
  - Dark Store B provides 2 Chocolates.
- Total delivery agents assigned: 3 (one per dark store fulfilling part of order).
- Total price calculated accordingly.

---

### Highlights and Key Insights

- **Handling multi-store inventory:** Items may be split across stores; the system must intelligently assign delivery partners and split orders.
- **Proximity filtering:** Users see only items available in dark stores within a configurable radius (default 5 km).
- **Flexible replenishment:** Supports multiple replenishment strategies; design allows easy extension.
- **Efficient stock management:** Uses map-based storage for O(1) lookups, avoiding linear scans.
- **Separation of concerns:** Managers handle orchestration and delegation, stores handle data, users handle carts.
- **Order placement algorithm:** Checks availability in closest store first; if insufficient, iterates through nearby stores to fulfill remaining items.
- **Design extensibility:** Possible to add persistence layers, payment gateways, coupon systems, and rider assignment algorithms as future enhancements.

---

### Timeline Table: High-Level Flow of Order Placement

| Step | Action                                                                                      |
|-------|---------------------------------------------------------------------------------------------|
| 1     | User opens app; Zepto fetches all available items within 5 km using DarkStoreManager.      |
| 2     | User adds items to cart (`Cart` object).                                                   |
| 3     | User places order; `OrderManager.placeOrder()` called with user, cart, DarkStoreManager.  |
| 4     | OrderManager fetches nearby dark stores via DarkStoreManager.                             |
| 5     | Checks if nearest dark store can fulfill entire order.                                    |
| 6     | If yes, assign one delivery agent; update inventory.                                      |
| 7     | If no, split order across multiple dark stores; assign multiple delivery agents accordingly.|
| 8     | Calculate and set total order amount.                                                     |
| 9     | Store order in OrderManager's list; return confirmation to user.                          |

---

### Summary of Extensions Suggested (Homework)

- Implement **Rider Mapping Algorithm** to assign delivery agents based on proximity, rating, or other heuristics.
- Integrate **Payment Gateway** or **Coupon Mechanism** for real-world payment flows.
- Modularize code into multiple files for better maintainability.
- Enhance inventory persistence with real database integration (SQL, MongoDB).
- Improve order fulfillment algorithm for optimization.

---

### Conclusion

This comprehensive design and implementation walkthrough provides a clear blueprint for building a scalable, maintainable 10-minute delivery app backend focused on **inventory management**, **order fulfillment** across multiple dark stores, and **delivery partner assignment**. The system balances complexity with modularity and applies core software design patterns to ensure robustness and extensibility.

This solution can be leveraged for technical interviews or real-world applications requiring rapid delivery logistics and inventory control.

