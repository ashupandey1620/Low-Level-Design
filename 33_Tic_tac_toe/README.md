### Summary: Low-Level Design of a Scalable Tic-Tac-Toe Game

This video presents a **comprehensive low-level design (LLD) of a Tic-Tac-Toe game** with a focus on interview preparation and practical coding. The design emphasizes **scalability, modularity, flexibility, and extensibility** following software design principles and patterns such as **Strategy** and **Observer**.

---

### Key Requirements and Design Goals

- **Scalable Board Size:** The board size is not fixed to the classical $3 \times 3$ but can be $n \times n$ (e.g., $4 \times 4$, $5 \times 5$).
- **Standard and Customizable Rules:** The game supports standard Tic-Tac-Toe rules but can be extended with custom rules (e.g., disabling diagonal wins).
- **In-App Notification System:** Real-time notifications inform all players about moves and game results, implemented using the Observer design pattern.
- **Multiple Players Support:** Although primarily designed for two players, the architecture supports extension to multiple players.

---

### Core Components and Class Hierarchy

| Component        | Description                                                                                                   |
|------------------|---------------------------------------------------------------------------------------------------------------|
| **Game**         | The main orchestrator class managing the board, players, game rules, observers, and game state (e.g., game over). Contains the primary `play()` method controlling game flow. |
| **Board**        | Represents the scalable $n \times n$ game board as a 2D vector of symbols. Manages cell states, size, and marking symbols. |
| **Symbol**       | Represents a mark on the board (e.g., 'X', 'O', or empty '_'). Supports extension for different or custom symbols. |
| **Player**       | Stores player-specific data: ID, name, symbol, and score. Player actions are integrated through the game loop. |
| **Rule (Strategy Pattern)** | Abstract class defining core rule methods: `checkWin()`, `checkDraw()`, and `isValidMove()`. Allows multiple rule implementations (standard or custom). |
| **StandardRule** | Concrete implementation of the default Tic-Tac-Toe rules. Checks horizontal, vertical, and diagonal wins and move validity. |
| **Observer Pattern** | Implemented for in-app notifications. The `Game` class is Observable; observers (e.g., `ConsoleNotifier`) receive updates on moves and game status. |

---

### Design Details and Methods

#### Board Class

- **Attributes:**
  - `size`: integer for board dimension.
  - `grid`: 2D vector of `Symbol` objects.
  - `emptySymbol`: a special symbol (e.g., '_') representing an empty cell.
  
- **Key Methods:**
  - `isCellEmpty(row, col)`: Checks if a cell is empty.
  - `getCell(row, col)`: Returns the symbol at a cell.
  - `markCell(row, col, symbol)`: Marks a symbol on the board if the cell is empty.
  - `display()`: Prints the current board state.

#### Rule Class (Abstract)

- `checkWin(Board, Symbol)`: Returns boolean indicating if the symbol has a winning set.
- `checkDraw(Board)`: Returns boolean indicating if the board is full with no winner.
- `isValidMove(Board, row, col)`: Validates if a move is possible on the board.

#### StandardRule Class (Concrete)

- Implements all Rule methods using checks over:
  - Rows
  - Columns
  - Main and anti-diagonals
- Validates moves by verifying if the target cell is empty.

#### Player Class

- **Attributes:**
  - `id`: unique player identifier.
  - `name`: player's name.
  - `symbol`: symbol used by player.
  - `score`: integer score counter.
  
- **Methods:**
  - Getters/setters for attributes.
  - `incrementScore()`: Increases player score on win.

#### Game Class

- **Attributes:**
  - `board`: Board instance.
  - `players`: Deque of Player objects for turn management.
  - `rule`: Rule instance managing game rules.
  - `observers`: List of Observer instances for notifications.
  - `gameOver`: Boolean indicating end of game.
  
- **Key Methods:**
  - `addPlayer(Player)`: Adds players to the deque.
  - `addObserver(Observer)`: Adds observers for notifications.
  - `notify(message)`: Sends updates to all observers.
  - `setRules(Rule)`: Sets game rules.
  - `play()`: Main game loop that:
    - Iteratively dequeues a player.
    - Collects player move input.
    - Validates the move through rules.
    - Marks the board.
    - Checks win/draw condition.
    - Notifies observers of moves and game state.
    - Enqueues player back unless game ends.
  
#### Observer Pattern for Notifications

- **Observable:** `Game` class maintains a list of observers.
- **Observer Interface:** Defines an `update(String message)` method.
- **Concrete Observer:** `ConsoleNotifier` displays notifications on the console.
- Notifications include player moves, win/draw announcements, or invalid move alerts.

---

### Turn Management Using Deque

- Players are stored in a **double-ended queue (deque)**.
- On each turn:
  - Player is dequeued from the front.
  - Player makes a valid move.
  - Player is enqueued at the rear.
- This approach supports extending to more than two players easily.

---

### Game Flow Overview (Pseudo Timeline)

| Step                     | Description                                                                                  |
|--------------------------|----------------------------------------------------------------------------------------------|
| Initialization           | Board size and game type chosen. Game, Board, Players, Rules, Observers instantiated.         |
| Player Addition          | Players added to a deque with their symbols and IDs.                                        |
| Observer Addition        | Observers (like `ConsoleNotifier`) added to receive game updates.                           |
| Game Start               | `play()` method begins game loop, notifying start.                                          |
| Player Turn              | Current player dequeued, asked for move input.                                              |
| Move Validation          | Move validated using `isValidMove()` rule method.                                           |
| Marking Board            | Valid move marked on board and board displayed.                                            |
| Notification             | Move and game state notifications sent to observers.                                       |
| Win/Draw Check           | `checkWin()` and `checkDraw()` applied to check game termination.                          |
| Turn Rotation            | If game continues, player enqueued back; next player dequeued.                             |
| Game End                 | On win or draw, notify observers, update scores, set `gameOver = true`.                    |

---

### Design Patterns Utilized

| Pattern         | Purpose                                                                                                   |
|-----------------|-----------------------------------------------------------------------------------------------------------|
| **Strategy**    | Abstract `Rule` class with multiple concrete implementations enables flexible game rules.                 |
| **Observer**    | Supports in-app notifications to players about game state changes without tight coupling.                 |
| **Factory**     | Optional factory design pattern used to create game instances based on game type/rule.                    |
| **Deque for Turn Management** | Efficient rotation of players' turns, scalable to any number of players.                                   |

---

### Summary of Fulfilled Requirements

- **Scalable Board:** Board size is configurable as $n \times n$.
- **Extensible Rules:** Abstract rules allow both standard and custom Tic-Tac-Toe variants.
- **Notification Service:** Observer pattern enables real-time in-app notifications.
- **Player Management:** Supports multiple players using a deque to manage turns.
- **Clean Separation of Concerns:** Board manages cell state; rules manage game logic; game orchestrates overall flow.

---

### Important Highlights

- **Single Responsibility Principle:** Each class has a clear, focused responsibility.
- **Extensibility:** Easily add new rules or notification types without changing core logic.
- **User Input & Validation:** Moves are validated before marking, ensuring game integrity.
- **Game Over Detection:** Boolean flag tracks game completion, terminating the loop.
- **Real-Time Communication:** Observers receive move updates and game results instantly.
- **Factory Pattern:** Demonstrates best practices and impresses interviewers by controlling object creation.

---

### Conclusion

This video provides a **structured, modular, and scalable low-level design for a Tic-Tac-Toe game**, implementing classic software design principles and patterns. It covers all essential aspects from board management, player handling, rule enforcement, to in-app notification systems, making it an excellent example for interview preparation and practical software design.

