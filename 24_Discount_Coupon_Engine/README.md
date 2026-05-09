### Summary: Design of a Discount Coupon Management System

This lecture presents a comprehensive **Low-Level Design (LLD)** problem focused on building a **scalable, generic discount coupon management system** that can be integrated ("plug and play") into various applications such as e-commerce platforms, food delivery apps, and inventory management systems. The problem is relevant for system design interviews and practical software development.

---

### Key Functional Requirements

- **Dynamic coupon addition at runtime**: System supports adding multiple new coupons, including bank coupons, internal coupons, category-based coupons, etc.
- **Multi-level discounts**: Coupons can apply on the entire cart (card-level) or on specific products/categories (product-level).
- **Multiple coupon types**: Seasonal offers, loyalty discounts, banking discounts, and bulk purchase coupons are supported.
- **Flexible discount strategies**: 
  - Flat discount (e.g., ₹20 off)
  - Percentage discount (e.g., 20% off)
  - Percentage with a cap (e.g., 60% off up to ₹120 maximum)
- **Coupon combination rules**: Some coupons can be combined/applied together, while others are exclusive.
- **Thread safety**: Designed for millions of users with concurrency control to manage coupon usage limits (e.g., limited number of uses per coupon).
- **Plug and play architecture**: The coupon system can be easily integrated with various applications without depending on application-specific user classes.

---

### Core System Components and Their Relations

| Component        | Description                                                                                     |
|------------------|-------------------------------------------------------------------------------------------------|
| Product          | Generic entity with attributes: name, category, price                                         |
| CartItem         | Aggregates a Product and quantity; calculates total price for that item                        |
| Cart             | Contains list of CartItems, flags like isLoyaltyMember, original total, and final total         |
| DiscountStrategy | Abstract class defining `calculate(amount: double): double` method; concrete strategies implement this for flat, percentage, and capped discounts |
| Coupon           | Abstract base class for coupons; holds a reference to next coupon (Chain of Responsibility pattern); methods include `isApplicable()`, `isCombinable()`, and `getDiscount()` |
| CouponManager    | Singleton managing a linked list of coupons; methods to register coupons, apply all applicable coupons on a cart, and retrieve applicable coupons |
| StrategyManager  | Singleton factory class creating DiscountStrategy objects based on enum types (flat, percent, percent with cap) |

---

### Detailed Design Highlights

#### Product, CartItem, and Cart

- **Product**: Simple model with name, category, price.
- **CartItem**: Holds a Product and its quantity; calculates total price as $price \times quantity$.
- **Cart**: Maintains a list of CartItems; tracks original total and final total after discounts; flags if user is loyalty member; supports adding products and applying discounts.

#### Discount Strategies (Strategy Design Pattern)

- Abstract `DiscountStrategy` defines method:

  $$
  \text{calculate}(amount: double) \to double
  $$

- Implementations:

  - **FlatDiscountStrategy**: subtracts a fixed discount amount.
  
    $$
    \text{discounted\_price} = \max(0, amount - discountAmount)
    $$
  
  - **PercentageDiscountStrategy**: applies percentage off.
  
    $$
    \text{discounted\_price} = amount - \frac{percentage}{100} \times amount
    $$

  - **PercentageWithCapDiscountStrategy**: applies percentage off capped at a maximum value.
  
    $$
    \text{discount} = \min\left(\frac{percentage}{100} \times amount, cap\right)
    $$
  
    $$
    \text{discounted\_price} = amount - \text{discount}
    $$

#### Coupon Class Hierarchy and Chain of Responsibility Pattern

- Coupons form a singly linked list.
- Each coupon defines:
  - `isApplicable(cart)`: checks if the coupon applies to the given cart (e.g., category match, bank card match, minimum spend).
  - `isCombinable(cart)`: whether the coupon can be combined with others.
  - `getDiscount(cart)`: calculates discount amount using the associated strategy.
  - `applyDiscount(cart)`: template method that:
    - Checks applicability.
    - Gets discount via strategy.
    - Applies discount to cart's final total.
    - If combinable, recursively calls next coupon's `applyDiscount`.
- Concrete coupon types:
  - **SeasonalCoupon**: applies discount on specific product categories.
  - **LoyaltyCoupon**: applies discount if cart is marked as loyalty member.
  - **BulkPurchaseCoupon**: applies flat discount if order value crosses threshold.
  - **BankingCoupon**: applies discount if payment bank matches and minimum spend is met.

#### Coupon Manager

- Singleton class maintaining head of coupon chain.
- Thread-safe registration of coupons at the tail of linked list.
- Methods:
  - `registerCoupon(coupon)`: adds coupon to chain.
  - `applyAllCoupons(cart)`: applies all applicable coupons to cart in order.
  - `getApplicableCoupons(cart)`: returns list of coupon names applicable to the cart.

#### Strategy Manager (Factory)

- Singleton factory that creates discount strategy instances based on type and parameters.
- Supports flat, percentage, and percentage with cap strategies.
- Encapsulates decision logic to instantiate correct strategy objects.

---

### System Flow Summary

- User adds products to cart.
- Cart calculates original total from CartItems.
- CouponManager fetches registered coupons and applies them sequentially using Chain of Responsibility.
- Each coupon:
  - Checks applicability.
  - Uses DiscountStrategy to calculate discount.
  - Applies discount to cart's final total.
  - Checks if combinable; if yes, passes control to next coupon.
- Final price after all discounts is presented to user for payment.

---

### Important Insights

- **Use of Design Patterns**: Strategy pattern for discount calculation flexibility, Chain of Responsibility for coupon application sequence, Template Method in coupon application flow.
- **Thread Safety**: Ensured in CouponManager for concurrent coupon registration and application.
- **Scalability and Extensibility**: System allows easy addition of new coupon types and discount strategies without modifying existing logic substantially.
- **Separation of Concerns**: Product, Cart, Coupon, DiscountStrategy, and Managers have distinct responsibilities, promoting maintainability.
- **Real-World Applicability**: This design closely mimics practical implementations in e-commerce and service apps, as referenced by examples like Swiggy, Zomato, Amazon.

---

### Quantitative Example (From Demo)

| Product       | Category    | Price (₹) | Quantity | Total (₹) |
|---------------|-------------|-----------|----------|-----------|
| Jacket        | Clothing    | 1000      | 1        | 1000      |
| Smartphone    | Electronics | 2000      | 1        | 2000      |
| Jeans         | Clothing    | 1000      | 2        | 2000      |
| Headphones    | Electronics | 2000      | 1        | 2000      |
| **Cart Total**|             |           |          | **7000**  |

- Coupons applied sequentially:
  - Seasonal offer: 10% off on Clothing
  - Loyalty discount: 5% off total
  - Bulk purchase: ₹100 off if total > ₹1000
  - Banking coupon: 15% off with ₹500 cap for ABC bank

- Resulting final discounted total: significantly reduced from original total after applying all coupons.

---

### Conclusion

This lecture delivers a **detailed LLD solution** for a discount coupon system that:

- Integrates multiple coupon types and discount strategies.
- Supports complex combination and applicability rules.
- Uses robust design patterns for modularity and flexibility.
- Is thread-safe and scalable for real-world high-load environments.
- Provides a strong example useful for interviews and practical implementations.

The approach balances **theoretical rigor with practical coding techniques**, emphasizing UML modeling, design principles, and clean architecture aligned with industry expectations.