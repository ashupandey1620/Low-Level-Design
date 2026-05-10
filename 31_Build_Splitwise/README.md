### Summary of Splitwise Application Design and Implementation

This video presents a comprehensive walkthrough of designing and implementing a **Splitwise clone**—an expense-sharing application commonly used among friends or groups. The content elaborates on the **low-level design (LLD)** aspects, core functionalities, class structures, design patterns, and an important **transaction simplification algorithm** used to optimize settlements.

---

### Core Features and Functional Requirements

- **User and Group Management**
  - Users can **join or leave groups**.
  - Users cannot leave a group if they have **pending unsettled expenses**.
  - Groups maintain lists of members and their balances.

- **Expense Management**
  - Users can **add expenses** in groups or as individual transactions.
  - Expenses can be **settled** between users.
  - Expenses can be split by:
    - **Equal Split**
    - **Percentage Split**
    - **Exact (Unequal) Split**
  - Each expense stores:
    - Expense ID
    - Description
    - Total amount
    - Paying user ID
    - Split details (which users owe how much)
    - Group ID (nullable for individual expenses)

- **Notification System**
  - When an expense is added or settled, **notifications** are sent to all group members.
  - Observer pattern is used with groups as **observable** and users as **observers**.

- **Transaction Simplification**
  - Uses a **greedy algorithm** to minimize the number of transactions required to settle debts within a group.
  - Simplifies multiple transactions into fewer transactions between users.

---

### Key Components and Their Responsibilities

| Component          | Responsibilities                                                                                  |
|--------------------|------------------------------------------------------------------------------------------------|
| **User**           | - Stores user info (ID, name, email)<br>- Maintains a balance map with other users (positive = owed, negative = owes)<br>- Implements Observer interface to receive notifications |
| **Group**          | - Observable entity<br>- Stores group ID, name, members, expenses, and balances<br>- Manages adding/removing users and expenses<br>- Sends notifications on updates |
| **Expense**        | - Contains expense ID, description, amount, payer, split details, and group association<br>- Represents a single expense record |
| **Split**          | - Represents a per-user split of an expense (user ID and amount)<br>- Facilitates multiple split strategies |
| **SplitStrategy**  | - Abstract interface for calculating splits<br>- Implementations: EqualSplit, PercentageSplit, ExactSplit |
| **SplitFactory**   | - Factory pattern to create split strategy instances based on split type enum (Equal, Percentage, Exact) |
| **Notification System** | - Implements Observer pattern<br>- Notifies users of expense additions and settlements |
| **DebtSimplifier** | - Utility class implementing the **greedy transaction simplification algorithm**<br>- Simplifies the balance map to minimize transactions |

---

### Design Patterns Used

| Pattern          | Purpose                                                                                      |
|------------------|----------------------------------------------------------------------------------------------|
| **Observer**     | For notification delivery from groups (Observable) to users (Observers)                      |
| **Strategy**     | To support various expense splitting strategies (Equal, Percentage, Exact)                   |
| **Factory**      | To instantiate appropriate SplitStrategy objects based on the split type                     |
| **Facade**       | The main Splitwise class acts as a facade managing users, groups, expenses, and delegations  |
| **Singleton**    | Used for the main Splitwise manager class, optionally for utility classes                    |

---

### Detailed Class and Data Structure Insights

- **User Class**
  - Fields: $userID$, $name$, $email$, $balances: Map<string, double>$ (maps other user IDs to balance amounts).
  - Methods: $update(message)$ (notification), $updateBalance(userID, amount)$, $getTotalOwing()$, $getTotalOwed()$.
  - Balance map semantics: **positive value means the user is owed money; negative means the user owes money**.

- **Group Class**
  - Fields:
    - $groupID$, $name$
    - $users: List<User>$
    - $balances: Map<string, Map<string, double>>$ (nested map storing balances between users)
    - $expenses: Map<string, Expense>$
  - Methods:
    - $addUser(user)$: adds user and initializes balance map entries.
    - $removeUser(userID)$: allowed only if balances are settled.
    - $addExpense(...)$: creates and stores an expense, updates balances accordingly.
    - $settlePayment(fromUserID, toUserID, amount)$: updates balances and notifies users.
    - $notify(message)$: notifies all users in the group.
    - $simplifyTransactions()$: calls DebtSimplifier to minimize settlement transactions.

- **Expense Class**
  - Fields:
    - $expenseID$, $description$, $amount$, $paidByUserID$, $splitDetails: List<Split>$, $groupID$ (nullable).
  - Represents one expense record detailing payer, amount, involved users, and split strategy used.

- **Split Class**
  - Fields:
    - $userID$, $amount$
  - Used inside Expense to denote how much each user owes.

- **SplitStrategy Interface**
  - Method: $$calculateSplit(amount: double, userIDs: List<string>, values?: List<double>) \to List<Split}$$
  - Implementations:
    - **EqualSplit**: divides amount equally.
    - **PercentageSplit**: divides amount according to provided percentages.
    - **ExactSplit**: divides amount based on exact amounts specified.

- **SplitFactory Class**
  - Creates instances of SplitStrategy based on enum type: Equal, Percentage, Exact.

- **DebtSimplifier (TransactionSimplifier)**
  - Input: $balances: Map<string, Map<string, double>>$
  - Process:
    - Computes **net amounts** per user (sum of amounts owed and owed to).
    - Separates users into **debtors (negative net)** and **creditors (positive net)**.
    - Uses a greedy approach to match the largest creditor with the largest debtor, settling amounts iteratively.
    - Produces a minimized set of transactions required to settle all debts.
  - Purpose: reduce the number of transactions to simplify group settlements.

---

### Transaction Simplification Algorithm (Greedy Approach)

1. **Calculate net balance** for each user:
   - For each pair $(A, B)$, if $B$ owes $A$ $\$x$, add $+x$ to $A$'s net and $-x$ to $B$'s net.
2. **Separate users** into:
   - **Creditors**: users with positive net balance (to receive money).
   - **Debtors**: users with negative net balance (to pay money).
3. **Sort** creditors and debtors by absolute net balance descending.
4. **Iteratively settle debts**:
   - Match the highest creditor and debtor.
   - Transfer the minimum of their owed/owing amounts.
   - Update their net balances.
   - Remove settled users (net zero).
5. **Result** is a minimized list of transactions.

This algorithm ensures **minimal transaction count** while preserving total settlement amounts.

---

### Example Walkthrough

- Group members: Aditya, Rohit, Manish, Saurabh.
- Transactions:
  - Aditya paid ₹800 for lunch split equally among all four.
  - Manish paid ₹700 for dinner split exactly among Aditya, Manish, and Saurabh.
- Before simplification:
  - Multiple transactions exist between members.
- After applying simplification:
  - Transactions reduce to two:
    - Saurabh pays ₹400 to Aditya.
    - Rohit pays ₹200 to Manish.
- Individual expenses and balances are also managed similarly.

---

### Summary Table: Split Types

| Split Type      | Description                          | Input Needed                 | Example (₹800 split among 4)            |
|-----------------|----------------------------------|-----------------------------|----------------------------------------|
| EqualSplit      | Divide amount equally             | Amount, list of users        | Each pays ₹200                         |
| PercentageSplit | Divide amount by given percentages | Amount, list of users, %     | User2: 20%, User3: 30%, User4: 50%   |
| ExactSplit      | Divide amount by exact values     | Amount, list of users, amounts | User2: ₹100, User3: ₹300, User4: ₹400 |

---

### Overall Architecture

- **Splitwise (Facade Singleton)**
  - Manages all users, groups, and expenses.
  - Provides API for user/group creation, expense addition, settlement, and simplification.
- **Groups** act as **observables** sending notifications to users.
- **Users** act as **observers** receiving updates.
- **Expense** objects represent transactions.
- **Split strategies** enable flexible expense division.
- **Debt simplifier** optimizes settlement transactions.
- Modular design with clear separation of concerns facilitates scalability and maintainability.

---

### Key Insights

- **Observer pattern** effectively handles notifications to multiple users.
- **Strategy and factory patterns** enable extensible split calculation methods.
- **Transaction simplification** using a greedy algorithm reduces complexity and number of payments.
- **Nested maps** for balances allow efficient tracking of debts between users within groups.
- Users cannot leave groups with outstanding balances, ensuring data consistency.
- Individual expenses outside groups are supported similarly.
- The design is interview-relevant, demonstrating both LLD skills and algorithmic problem-solving.

---

### Conclusion

The video thoroughly explains the design, class structure, and algorithms behind a **Splitwise clone**, emphasizing:

- **Functional completeness**: user/group management, expense handling, and notifications
- **Design robustness**: using design patterns and data structures effectively
- **Algorithmic efficiency**: transaction simplification to minimize settlements

This serves as an excellent reference for building scalable expense-sharing applications and preparing for system design interviews involving complex business logic and algorithms.

