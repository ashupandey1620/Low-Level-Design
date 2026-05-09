### Summary

This video lecture presents a **Low-Level Design (LLD) of a dating application**, similar to popular apps like Tinder, Bumble, or Hinge. The focus is on designing core functionalities, integrating various features, and understanding design patterns and algorithms relevant to dating apps. The approach emphasizes **top-down design**, breaking down the system into manageable objects and modules, and integrating them using proven software design principles.

---

### Key Functional Requirements

- **User Profile Management:** Users can create and update profiles (name, bio, interests, photos, location, gender).
- **User Preferences:** Set preferences for age range, gender(s) interested in, maximum distance, and interests.
- **Swipe Mechanism:** Users can swipe left or right on other profiles.
- **Match Making:** Match occurs when two users right-swipe each other; match score based on multiple factors.
- **Nearby Profiles:** Users can view profiles within a specified radius (e.g., 5 km), using scalable location strategies.
- **Chat Room:** Once matched, users can chat in real-time.
- **Notifications:** Users receive notifications for new matches and messages.

---

### Non-Functional Requirements & Principles

- System should be **scalable** and follow design principles like **loose coupling**, **single responsibility**, and **modularity**.
- Use of **design patterns** like **Observer**, **Strategy**, and **Chain of Responsibility**.
- Modular and extensible code architecture is recommended.

---

### Design Approach

- **Top-Down Approach:** Start by defining large objects (User, NotificationService, LocationService, Matcher, ChatRoom) and then break them into smaller objects (Profile, Preferences, Interest, SwipeHistory, Message).
- Emphasizes **composition** with strong ownership (User owns Profile, Preferences, NotificationObserver).
- Use of **Observer pattern** for notification delivery.
- Use of **Strategy pattern** for location-based filtering.
- Use of **Chain of Responsibility (CoR)** pattern for match scoring to combine multiple matching criteria.

---

### Core Components and Their Responsibilities

| Component           | Responsibilities                                                                                   |
|---------------------|--------------------------------------------------------------------------------------------------|
| **User**            | Maintains profile, preferences, swipe history, and notification observer reference.              |
| **UserProfile**      | Stores name, age, gender (enum), bio, photos (paths), interests, and location.                    |
| **UserPreferences**  | Stores min/max age, max distance, interested genders, and interests (strings).                    |
| **Location**        | Holds latitude and longitude; calculates distance to another Location using Haversine formula.   |
| **NotificationService** (Observable) | Manages observers for users; sends notifications for matches and messages using Observer pattern. |
| **NotificationObserver** (Observer)  | Receives and processes notifications for a user.                                         |
| **SwipeHistory**     | Tracks left/right swipes on other users by user ID.                                              |
| **LocationService**  | Uses Strategy pattern to find nearby users based on location and max distance.                   |
| **LocationStrategy** | Interface defining method to find nearby users; multiple strategies can be implemented.          |
| **Matcher**          | Abstract match algorithm with method to calculate match score between two users.                 |
| **BasicMatcher**     | Checks basic criteria: gender preference, age range, distance. Returns 0 if criteria fail.       |
| **InterestMatcher**  | Extends BasicMatcher; calculates score based on shared interests.                                |
| **LocationMatcher**  | Extends InterestMatcher; adds proximity score based on user distance.                            |
| **MatcherFactory**   | Factory to create matcher instances based on type (Basic, Interest, Location).                   |
| **ChatRoom**         | Stores participants (multiple users) and messages; supports add/remove and display operations.   |
| **Message**          | Stores sender ID, content, and timestamp.                                                        |
| **DatingApp**        | Orchestrator managing users, chat rooms, matchers, swipe actions, and user interactions.         |

---

### Matching Algorithm Details

- Uses **Chain of Responsibility pattern**:
  - **BasicMatcher:** Validates gender and age preferences, distance constraints.
  - **InterestMatcher:** Builds on BasicMatcher, adds interest-based scoring.
  - **LocationMatcher:** Builds on InterestMatcher, adds proximity scoring.
- Each matcher calls the next matcher recursively, accumulating scores.
- Scores are normalized and combined to provide a final match percentage.
- Supports flexible order of evaluation (can start from location, interests, or basic).

---

### Notification System (Observer Pattern)

- **NotificationService** maintains a map of user IDs to their notification observers.
- Supports methods to register, deregister, and notify observers either individually or in bulk.
- Observers implement an **update(message)** method to receive notifications.
- Notification messages delivered when a match occurs or when a new chat message is received.

---

### Location Handling (Strategy Pattern)

- Location represented by an object with **latitude ($lat$)** and **longitude ($long$)**.
- Distance calculation uses the **Haversine formula** to compute accurate distance in kilometers.
- **LocationService** delegates nearby user search to a pluggable **LocationStrategy**, allowing future enhancements like filtering blocked users or applying advanced geospatial queries.

---

### Chat and Messaging

- **ChatRoom** supports multiple participants (users).
- Stores messages with sender ID, content, and timestamp.
- Methods include adding/removing messages, displaying chat history.
- On message send, notification is sent to the receiver.

---

### User Swipe Handling

- Swipes are recorded as either **Left** or **Right** against other user IDs.
- Swipe history is stored in a map of user ID to swipe action.
- When a user swipes right, system checks if the other user has also swiped right, indicating a match.
- On match, a new chat room is created, and notifications are sent to both users.

---

### Example Scenario (Happy Flow)

- User1 (Rohan, 28, Male) sets profile and preferences (interested in females aged 25-30 within 10 km, interests: coding, traveling).
- User2 (Neha, 27, Female) sets profile and preferences (interested in males aged 27-30 within 15 km, interests: coding, movies).
- Location set for both users.
- User1 searches nearby users and finds User2.
- User1 swipes right on User2.
- User2 swipes right on User1.
- System detects mutual right swipe → match established.
- Chat room created between User1 and User2.
- Notifications sent to both users.
- Users exchange messages; notifications sent on new messages.
- Chat history can be displayed.

---

### UML and Code Structure Highlights

- The system consists of modular classes representing entities and services.
- All core classes have basic getters/setters and constructors.
- Notification and location services are implemented as **Singletons**.
- Design patterns are explicitly used and adapted for the domain.
- The entire codebase is organized into a single file for simplicity but modularization is recommended for production-level code.

---

### Important Formulas

**Haversine Distance Formula:**

Given two points with latitude and longitude $(\phi_1, \lambda_1)$ and $(\phi_2, \lambda_2)$:

$$
a = \sin^2\left(\frac{\Delta \phi}{2}\right) + \cos(\phi_1) \cdot \cos(\phi_2) \cdot \sin^2\left(\frac{\Delta \lambda}{2}\right)
$$

$$
c = 2 \cdot \arctan2(\sqrt{a}, \sqrt{1-a})
$$

$$
d = R \cdot c
$$

Where $d$ is distance, $R$ is Earth's radius (approximately 6371 km).

---

### Summary Table: Match Scoring Components

| Matcher Type      | Criteria                                                                                         | Score Range            | Notes                          |
|-------------------|------------------------------------------------------------------------------------------------|------------------------|--------------------------------|
| BasicMatcher      | Gender preferences, age range, distance within max limit                                        | 0 or 0.5 (base score)  | Returns 0 if criteria not met   |
| InterestMatcher   | Shared interests count normalized by max interests of either user                               | Adds up to 0.5 bonus   | Depends on BasicMatcher success |
| LocationMatcher   | Adds proximity score inversely proportional to distance (max 0.2 points)                        | Adds up to 0.2 bonus   | Depends on InterestMatcher success |

---

### Key Insights

- **Top-down design** is effective for complex applications with many small interacting objects.
- **Observer pattern** suits notification systems where users must be informed asynchronously.
- **Strategy pattern** allows flexible, pluggable algorithms for location-based filtering.
- **Chain of Responsibility pattern** elegantly combines multiple matching algorithms in sequence, enabling extensible and modular scoring.
- The design supports **scalability and extensibility** to add features like blocking users, advanced filtering, or multiple chat participants.
- Real-world dating applications rely heavily on **match scoring algorithms**, which consider multiple factors and dynamically adjust weights.

---

### Conclusion

This video provides a comprehensive, practical guide to designing a feature-rich dating application with a clean, modular architecture. It balances software design principles with domain-specific logic, demonstrating how to implement:

- User profile and preference management,
- Swipe and match logic,
- Notification handling,
- Location-based filtering,
- Multi-factor match scoring,
- Chat functionality.

By following this design and coding approach, developers can build scalable, maintainable dating apps and be well-prepared for related system design interviews.

---

### Keywords

- Low-Level Design (LLD)
- Dating Application
- Swipe Left/Right
- Matchmaking Algorithm
- Observer Pattern
- Strategy Pattern
- Chain of Responsibility Pattern
- User Profile & Preferences
- Location-Based Filtering
- Notification System
- Chat Room Design
- Haversine Formula
- Modular Architecture
- Singleton Pattern
- Software Design Principles