# SDE2 LLD Interview — Final Preparation List

**Target:** 2–3 YOE SDE2  
**Interview:** ~1 hour total, ~45 minutes LLD  
**Goal:** Solve an unfamiliar LLD problem cleanly within 45 minutes.

---

# 1. Interview Strategy

## 45-Minute LLD Flow

| Time | What to do |
|---|---|
| 0–5 min | Clarify requirements + scope |
| 5–10 min | Identify entities + relationships |
| 10–35 min | Design + code core functionality |
| 35–40 min | Edge cases + extensibility |
| 40–45 min | Concurrency + interviewer questions |

### Priority

```text
Requirements
    ↓
Entities
    ↓
Responsibilities
    ↓
Relationships
    ↓
Core use case
    ↓
Code
    ↓
Edge cases
    ↓
Concurrency / extensibility
```

### Don't

- Over-engineer
- Add patterns just to show patterns
- Create repositories/DAOs unnecessarily
- Build microservices
- Build Kafka/Redis infrastructure unless asked
- Implement every possible feature
- Spend 20 minutes designing before writing code

> **Goal: Clean OOP + correct modeling + working core flow + reasonable extensibility**

---

# 2. 🔴 Tier 1 — Deeply Prepare

These are the problems you should be able to code from scratch.

---

## 1. Parking Lot ⭐⭐⭐⭐⭐

### Learn

- Entity modeling
- Composition
- Enums
- Interfaces / inheritance
- Resource allocation
- Service layer
- Strategy

### Core Classes

```text
ParkingLot
Floor
ParkingSpot
Vehicle
Ticket
```

### Optional Pattern

```text
ParkingSpotStrategy
    ├── NearestSpotStrategy
    └── FirstAvailableStrategy
```

Use Strategy only if the allocation algorithm can vary.

### Code

- Park vehicle
- Find available spot
- Generate ticket
- Exit vehicle
- Calculate fee

### Discuss

- Multiple floors
- Vehicle types
- Different pricing
- Concurrent parking requests

### Don't Build

- Payment gateway
- Repository layer
- Notification system
- Complex optimization

---

## 2. Movie Booking / BookMyShow ⭐⭐⭐⭐⭐

### Learn

- Reservation modeling
- Availability
- Relationships
- Booking workflow
- Concurrency
- Strategy

### Core Classes

```text
Movie
Theatre
Screen
Seat
Show
User
Booking
BookingService
PaymentStrategy
```

### Pattern

```text
PaymentStrategy
    ├── UPI
    ├── Card
    └── NetBanking
```

No Factory unless object creation itself becomes important.

### Core Flow

```text
Search movie
    ↓
Select show
    ↓
View seats
    ↓
Select seats
    ↓
Book
    ↓
Pay
```

### VERY IMPORTANT

A `Seat` is a physical seat.

Availability belongs to a `Show`.

```text
Seat
  A1

Show 1
  A1 → AVAILABLE

Show 2
  A1 → BOOKED
```

Do NOT simply put:

```python
seat.is_booked
```

because the same physical seat can be booked for different shows.

### Critical Discussion

Two users try to book the same seat.

Discuss:

- Database transaction
- Locking
- Atomic update
- Preventing double booking
- Temporary seat locking if required

---

## 3. Splitwise ⭐⭐⭐⭐⭐

### Learn

- Business rules
- Polymorphism
- Strategy
- Clean domain modeling

### Core Classes

```text
User
Group
Expense
ExpenseSplit
Balance
ExpenseService
```

### Strategy

```text
ExpenseSplitStrategy
    ├── EqualSplit
    ├── ExactSplit
    └── PercentageSplit
```

### Code

- Create expense
- Split expense
- Update balances
- Show balances

### Discuss

- Validation
- Groups
- Settlements
- Debt simplification

Don't spend interview time implementing sophisticated debt optimization unless asked.

---

## 4. Vending Machine ⭐⭐⭐⭐⭐

### Learn

- State pattern
- State transitions
- Encapsulation

### Core Classes

```text
VendingMachine
Product
Inventory
Payment
```

### State

```text
IDLE
  ↓
PRODUCT_SELECTED
  ↓
PAYMENT_PENDING
  ↓
DISPENSING
  ↓
IDLE
```

### Code

- Select product
- Insert money
- Dispense
- Return change
- Cancel

### Discuss

- Insufficient money
- Out of stock
- Invalid state
- Refund/cancellation

---

## 5. Rate Limiter ⭐⭐⭐⭐⭐

### Learn

- Time-based algorithms
- Thread safety
- Concurrency
- Atomic operations

### Core Classes

```text
RateLimiter
Bucket
```

### Strategy

Only use Strategy if multiple algorithms are relevant.

```text
RateLimiterStrategy
    ├── TokenBucket
    ├── FixedWindow
    └── SlidingWindow
```

If interviewer asks for Token Bucket, just implement Token Bucket.

### Core API

```python
allow_request(user_id)
```

### Discuss

- Concurrent requests
- Race conditions
- Locks
- Atomic updates
- Per-user limits
- Global limits
- Distributed rate limiting

Don't build Redis/distributed infrastructure unless asked.

---

## 6. LRU Cache ⭐⭐⭐⭐⭐

### Learn

- Data structures
- O(1) operations
- Encapsulation
- Thread-safety discussion

### Core Design

```text
HashMap
   +
Doubly Linked List
```

### Pattern

**NONE**

Don't force a design pattern.

### Code

```text
get()
put()
```

Both should be `O(1)`.

### Discuss

- Capacity
- Eviction
- Thread safety
- TTL if asked

---

## 7. Tic-Tac-Toe ⭐⭐⭐⭐

### Learn

- Basic OO design
- Game state
- Turn management
- Polymorphism

### Core Classes

```text
Game
Board
Player
Piece
```

### Pattern

**NONE**

### Code

```text
makeMove()
checkWinner()
switchTurn()
```

This is your warm-up problem.

You should be able to finish it comfortably in ~20–25 minutes.

---

# 3. 🟡 Tier 2 — Derive From Tier 1

Don't memorize separate implementations.

Learn how to map the same concepts to a new domain.

---

## 8. Ride Matching ⭐⭐⭐⭐

### Core Classes

```text
Driver
Rider
Ride
Location
MatchingService
```

### Possible Strategy

```text
MatchingStrategy
    ├── NearestDriver
    ├── LeastBusyDriver
    └── ...
```

### Driver State

```text
AVAILABLE
    ↓
ON_TRIP
    ↓
AVAILABLE
```

### Code

```text
requestRide()
findDriver()
assignDriver()
completeRide()
```

### Discuss

- Nearest driver
- Driver availability
- Concurrent assignment
- Reassignment
- Pricing strategy

This combines:

```text
Strategy
+
State
+
Matching
+
Concurrency
```

---

## 9. Elevator ⭐⭐⭐⭐

### Core Classes

```text
Elevator
ElevatorController
Request
```

### Strategy

```text
SchedulingStrategy
    ├── Nearest
    ├── SCAN
    └── LOOK
```

Don't implement multiple algorithms unless asked.

### Code

```text
requestElevator()
selectElevator()
moveElevator()
```

### Discuss

- Multiple elevators
- Direction
- Scheduling
- Concurrent requests

---

## 10. Amazon Locker ⭐⭐⭐⭐

### Derive From

Parking Lot.

```text
Vehicle → ParkingSpot
Package → Locker
```

### Core Classes

```text
Locker
Package
User
Delivery
Pickup
```

### Optional Strategy

```text
LockerSelectionStrategy
    ├── NearestLocker
    └── SmallestSuitableLocker
```

### Code

```text
assignLocker()
depositPackage()
pickupPackage()
```

### Discuss

- Locker size
- Expiration
- Unavailable lockers
- Concurrent assignment

---

## 11. IRCTC / Railway Reservation ⭐⭐⭐⭐

### Derive From

Movie Booking + Concurrency.

### Core Classes

```text
Train
Station
Route
Coach
Seat
User
Booking
```

### Optional Strategy

```text
SeatAllocationStrategy
```

### Code

```text
searchTrain()
checkAvailability()
bookSeat()
cancelBooking()
```

### Discuss

- Last-seat race condition
- Seat allocation
- Cancellation
- Waitlist
- Concurrent bookings

Don't implement the entire railway system.

---

## 12. Notification System ⭐⭐⭐⭐

### Learn

- Observer
- Strategy
- Event-driven thinking

### Core Classes

```text
Notification
NotificationService
NotificationChannel
User
```

### Strategy

```text
NotificationChannel
    ├── Email
    ├── SMS
    └── Push
```

### Observer Concept

```text
Event
  ↓
NotificationService
  ↓
Subscribers
```

### Code

```text
sendNotification()
```

### Discuss

- Multiple channels
- User preferences
- Event-driven notifications

Don't build Kafka, queues, retry frameworks, or template engines unless asked.

---

## 13. Pub/Sub ⭐⭐⭐

### Core Classes

```text
Publisher
Topic
Subscriber
Message
```

### Core Flow

```text
Publisher
    ↓
Topic
    ↓
Subscribers
```

### Learn

- Publisher/subscriber relationships
- Event delivery
- Observer-style thinking

### Discuss Only

- Consumer groups
- Pull vs push
- Durability
- Offsets
- Retry
- Message ordering

Don't try to build Kafka.

---

## 14. Quick Commerce ⭐⭐⭐

### Core Classes

```text
Product
Inventory
Store
Customer
Order
Delivery
```

### Code

```text
browse()
addToCart()
placeOrder()
reserveInventory()
```

### Discuss

- Inventory race conditions
- Multiple stores
- Cancellation
- Order lifecycle

Don't build an entire Zepto/Swiggy architecture.

---

## 15. Hotel Reservation ⭐⭐⭐

### Derive From

Movie Booking.

```text
Movie       → Hotel
Screen      → Room
Seat        → Room
Show        → Availability Slot
Booking     → Reservation
```

### Code

```text
searchRooms()
checkAvailability()
reserve()
cancel()
```

### Discuss

- Date ranges
- Room types
- Concurrent booking

---

## 16. Seller Experience ⭐⭐⭐

### Core Classes

```text
Seller
Product
Listing
Inventory
Order
```

### Code

```text
createListing()
updateInventory()
viewOrders()
```

### Pattern

None required.

---

## 17. Customer Reviews ⭐⭐

### Core Classes

```text
User
Product
Review
Rating
```

### Code

```text
addReview()
updateReview()
deleteReview()
getRating()
```

### Discuss

- One review per user/product
- Rating aggregation
- Moderation
- Concurrent rating updates

Quick problem. Don't spend much preparation time here.

---

# 4. 🟢 Tier 3 — Review Only

Understand the modeling but don't deeply practice implementation.

---

## 18. Messenger / WhatsApp

### Core Classes

```text
User
Conversation
Message
Participant
```

### Discuss

- 1:1 chat
- Group chat
- Message status
- Delivery/read receipts

Observer is optional.

Don't design distributed WhatsApp infrastructure.

---

## 19. File System

### Core Classes

```text
File
Folder
User
Storage
```

### Pattern

Composite — only if tree operations matter.

```text
Folder
 ├── File
 ├── File
 └── Folder
      └── File
```

---

## 20. File Download System

### Core Classes

```text
File
DownloadRequest
DownloadService
Storage
```

### Pattern

Strategy only if multiple download/storage mechanisms exist.

Otherwise:

**No pattern required.**

---

## 21. Chess

### Core Classes

```text
Game
Board
Player
Piece
Position
```

### Polymorphism

```text
Piece
 ├── King
 ├── Queen
 ├── Rook
 ├── Bishop
 ├── Knight
 └── Pawn
```

### Code

```text
move()
validateMove()
checkGameState()
```

Don't implement every chess rule unless asked.

---

# 5. Design Pattern Cheat Sheet

## Strategy ⭐⭐⭐⭐⭐

Use when the algorithm/behavior can vary.

Examples:

```text
Payment
Expense Split
Pricing
Driver Assignment
Locker Assignment
Elevator Scheduling
Rate Limiting
```

## State ⭐⭐⭐⭐

Use when object behavior changes depending on current state.

Examples:

```text
Vending Machine
Elevator
Ride Lifecycle
ATM
```

## Observer ⭐⭐⭐

Use when multiple components need to react to an event.

Examples:

```text
Notification
Messenger
Pub/Sub
```

## Composite ⭐⭐

Use when the structure is hierarchical/tree-like.

Example:

```text
File System
```

## Factory ⭐⭐

Know it, but don't force it.

Use when object creation genuinely needs abstraction.

Don't create factories just to demonstrate a pattern.

## No Pattern

These are completely fine without patterns:

```text
LRU Cache
Tic-Tac-Toe
Hotel Reservation
Quick Commerce
Seller Experience
Customer Reviews
Chess
```

---

# 6. Pattern Decision Rule

```text
Does behavior/algorithm vary?
        ↓
      YES
        ↓
    Strategy
```

```text
Does behavior change based on state?
        ↓
      YES
        ↓
      State
```

```text
Do multiple objects react to an event?
        ↓
      YES
        ↓
    Observer
```

```text
Is the structure hierarchical/tree-like?
        ↓
      YES
        ↓
    Composite
```

```text
Is object creation genuinely complex?
        ↓
      YES
        ↓
    Factory
```

Otherwise:

```text
Plain OOP
```

is enough.

---

# 7. SDE2 Concepts You Must Be Comfortable Discussing

## OOP

Know:

- Encapsulation
- Abstraction
- Polymorphism
- Composition vs inheritance
- Interfaces
- Abstract classes

## SOLID

Especially:

### SRP

One class should have one clear responsibility.

### OCP

New behavior should be addable without heavily modifying existing code.

This is where Strategy often helps.

## Dependency Injection

Understand:

```text
Class A
   ↓
depends on
   ↓
Interface B
```

instead of tightly coupling A to one concrete implementation.

## Concurrency

Be able to discuss:

- Race conditions
- Locks
- Atomic operations
- Transactions
- Optimistic vs pessimistic locking
- Thread safety

Especially for:

```text
Movie Booking
IRCTC
Rate Limiter
Parking Lot
Ride Matching
Inventory
```

## Idempotency

Understand why operations like these may need idempotency:

```text
book()
pay()
cancel()
createOrder()
```

---

# 8. Final Study Order

## Phase 1 — Deep Coding

```text
1. Parking Lot
2. Movie Booking
3. Splitwise
4. Vending Machine
```

## Phase 2 — SDE2 Concepts

```text
5. Rate Limiter
6. LRU Cache
7. Tic-Tac-Toe
```

## Phase 3 — Derivation Practice

```text
8. Ride Matching
9. Elevator
10. Amazon Locker
11. IRCTC
12. Notification System
13. Pub/Sub
14. Quick Commerce
15. Hotel Reservation
16. Seller Experience
17. Customer Reviews
```

## Phase 4 — Review Only

```text
18. Messenger
19. File System
20. File Download
21. Chess
```

---

# 9. How to Prepare Each Problem

For every Tier 1 problem, practice this exact sequence:

```text
1. Scope requirements
        ↓
2. Identify entities
        ↓
3. Define responsibilities
        ↓
4. Define relationships
        ↓
5. Identify the main use case
        ↓
6. Start coding
        ↓
7. Add one natural extension
        ↓
8. Discuss concurrency
```

You should be able to explain:

1. Why each class exists
2. Why each responsibility belongs where it does
3. Why you used a pattern, if you used one
4. Why you did not use unnecessary patterns
5. How the main use case flows
6. What happens under concurrent requests
7. How you would extend the design

---

# 10. What NOT to Over-Engineer

Avoid these unless explicitly asked:

```text
Repository layer
DAO layer
Controller layer
Factory everywhere
Singleton everywhere
Builder everywhere
Microservices
Kafka
Redis
Distributed locks
Complex DI frameworks
Database schemas
Payment gateways
Notification systems
```

The interviewer is mainly evaluating:

```text
Can you model the problem?
        ↓
Can you assign responsibilities correctly?
        ↓
Can you write clean OO code?
        ↓
Can you extend the design?
        ↓
Can you identify race conditions?
        ↓
Can you explain your decisions?
```

> **The goal is not to fit as many design patterns as possible into 45 minutes.**
>
> **The goal is to show strong OO design, practical judgment, and the ability to evolve the design.**
