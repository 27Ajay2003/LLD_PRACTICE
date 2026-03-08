# High-Yield LLD / OOP Python — 15 Questions Ranked by Importance
### Target: SDE 1–2 Years | FAANG · Top PBC · India Startups · Remote Roles

---

## How a Real 45-Min Interview Works

| Time | What You Do |
|------|-------------|
| 0–5 min | Clarify scope. Ask what to include/exclude. Never assume. |
| 5–15 min | Identify entities, fields, relationships. Talk out loud. |
| 15–35 min | Write class skeletons + implement the 2–3 core methods fully |
| 35–45 min | Discuss extensions, tradeoffs, what you'd do differently at scale |

**You are never expected to write a complete working system in 45 min.**
You are evaluated on how you think, how you model a domain, and whether your core methods are clean.
Every problem below has an explicit **"What to actually code"** vs **"What to just discuss"** split.

---

## Master Ranking Table

| Rank | Problem | Pattern | Difficulty to finish in 45 min |
|------|---------|---------|-------------------------------|
| 1 | Parking Lot | Factory, OOP | Easy — fully codeable |
| 2 | Splitwise | Strategy | Easy — fully codeable |
| 3 | BookMyShow | Concurrency, Factory | Hard — scope it down heavily |
| 4 | LRU Cache | DSA-as-LLD | Easy — fully codeable |
| 5 | Ride Matching | Multi-pattern, State | Hard — scope it down heavily |
| 6 | Rate Limiter | Strategy | Medium — one algo only |
| 7 | Notification Service | Observer | Easy — fully codeable |
| 8 | Vending Machine | State Machine | Medium — 2-3 states only |
| 9 | Job Scheduler | Command, PQ | Medium — no recurring/retry in code |
| 10 | Food Delivery | Multi-pattern, State | Hard — scope it down heavily |
| 11 | In-Memory File System | Composite | Medium — mkdir + ls + get_size |
| 12 | Logger Framework | Chain of Responsibility | Easy — fully codeable |
| 13 | Chess | Rules Engine | Hard — 2-3 pieces only |
| 14 | Pub-Sub System | Observer (system level) | Medium — push model only |
| 15 | Twitter Feed | Multi-pattern, Fan-out | Hard — fan-out on write only |

---
---

# 1. Parking Lot ★★★★★
**Asked at:** Everywhere. Most asked LLD question, period.
**Pattern:** Factory, Singleton, basic OOP.

## ✅ What to Actually Code (35 min)
- `VehicleType` and `SpotStatus` enums
- `ParkingSpot`, `Vehicle`, `Ticket` with all fields
- `ParkingFloor.get_free_spot(vehicle_type)`
- `ParkingLot.park_vehicle()` and `ParkingLot.unpark_vehicle()`
- One concrete `PricingStrategy` (e.g. `HourlyPricing`)

## 💬 What to Just Discuss
- Multiple pricing strategies and how Strategy pattern makes them swappable
- Singleton implementation of `ParkingLot`
- How you'd add a live display board (Observer pattern)

## Core Entities
```
VehicleType    → BIKE / CAR / TRUCK
SpotStatus     → FREE / OCCUPIED
ParkingSpot    → id, spot_type: VehicleType, status: SpotStatus
Vehicle        → license_plate, vehicle_type: VehicleType
Ticket         → id, vehicle, spot, entry_time: datetime
ParkingFloor   → id, spots: List[ParkingSpot]
ParkingLot     → floors: List[ParkingFloor], active_tickets: Dict[str, Ticket]
```

## Important Methods
```python
# ParkingLot
park_vehicle(vehicle: Vehicle) -> Optional[Ticket]
unpark_vehicle(ticket: Ticket) -> float          # returns fare
find_available_spot(vehicle_type: VehicleType) -> Optional[ParkingSpot]

# ParkingFloor
get_free_spot(vehicle_type: VehicleType) -> Optional[ParkingSpot]

# PricingStrategy (abstract)
calculate_fare(entry_time, exit_time, vehicle_type) -> float
```

## Key Design Notes
- Each `VehicleType` maps 1:1 to a `SpotType` — no cross-parking by default
- `ParkingLot` iterates floors until a free spot is found — first-fit
- `PricingStrategy` is injected so fare logic is swappable

## Common Extension
> *"Add a live display board"* → Observer: floors notify board on every status change

---
---

# 2. Splitwise – Expense Sharing ★★★★★
**Asked at:** CRED, Razorpay, PhonePe, Groww, Niyo.
**Pattern:** Strategy (split types), Singleton.

## ✅ What to Actually Code (35 min)
- `User`, `Group`, `Expense`, `Split` (abstract) with all fields
- `EqualSplit` and `ExactSplit` concrete classes with `validate()` and `calculate_amounts()`
- `ExpenseManager.add_expense()` — update net balances after each expense
- `ExpenseManager.get_user_balance()`

## 💬 What to Just Discuss
- `PercentSplit` implementation (same pattern as the other two — mention it)
- `settle_balance()` flow
- Debt simplification extension — greedy graph approach

## Core Entities
```
User           → id, name
Group          → id, members: List[User], expenses: List[Expense]
Expense        → id, amount, paid_by: User, splits: List[Split]
Split          → user: User, amount: float
ExpenseManager → groups: Dict[str, Group], balances: Dict[str, Dict[str, float]]
```

## Important Methods
```python
# ExpenseManager
add_expense(group_id, expense: Expense) -> None
settle_balance(user_a: User, user_b: User, amount: float) -> None
get_user_balance(user_id: str) -> Dict[str, float]

# Split (abstract)
validate() -> bool
calculate_amounts(total: float) -> None
```

## Key Design Notes
- Balances are **net** — if A owes B ₹100 and B owes A ₹40, store net ₹60
- `validate()` before `calculate_amounts()` — fail fast
- New split type = new subclass only, zero changes to `ExpenseManager`

## Common Extension
> *"Minimize transactions to settle"* → net each balance, greedily pair max-creditor with max-debtor

---
---

# 3. BookMyShow – Ticket Booking ★★★★★
**Asked at:** Swiggy, Meesho, Nykaa, Juspay, MakeMyTrip.
**Pattern:** Factory, Singleton, Concurrency.

## ✅ What to Actually Code (35 min)
- `SeatStatus`, `BookingStatus` enums
- `Movie`, `Screen`, `Show`, `Seat`, `Booking` with fields
- `Show.is_seat_available()` and `Show.get_available_seats()`
- `BookingManager.lock_seats()` with a `threading.Lock()` — this is the whole point of the problem

## 💬 What to Just Discuss
- `confirm_booking` and payment flow
- TTL expiry on locks — how you'd implement a background cleanup thread
- `search_shows` — trivially a filter, not worth coding
- Payment abstraction and subclasses

## Core Entities
```
SeatType       → REGULAR / PREMIUM / RECLINER
SeatStatus     → AVAILABLE / LOCKED / BOOKED
BookingStatus  → PENDING / CONFIRMED / CANCELLED
Show           → id, movie, screen, start_time, seat_map: Dict[str, SeatStatus]
Seat           → id, row, number, seat_type: SeatType
Booking        → id, user, show, seats: List[Seat], status: BookingStatus
```

## Important Methods
```python
# BookingManager
lock_seats(show_id, seat_ids, user_id) -> bool   # THE core method — use per-show Lock
confirm_booking(booking_id, payment) -> Booking
cancel_booking(booking_id) -> None
release_locks(show_id, seat_ids) -> None

# Show
get_available_seats() -> List[Seat]
is_seat_available(seat_id) -> bool
```

## Key Design Notes
- `lock_seats` uses `threading.Lock()` **per show** — not one global lock
- Lazy TTL check: on `confirm_booking`, check if lock is still within 10-min window
- The interviewer's trap question: *"Two users select the same seat simultaneously"* → first to acquire lock wins

## Common Extension
> *"Add waitlist for sold-out shows"* → `Queue` per show; notify head of queue when a booking cancels

---
---

# 4. LRU Cache ★★★★★
**Asked at:** Amazon, Flipkart, Hotstar, Dunzo — almost all Indian product companies.
**Pattern:** DSA-as-LLD.

## ✅ What to Actually Code (35 min)
- `Node` class with `key`, `value`, `prev`, `next`
- `LRUCache.__init__` with dummy head/tail, capacity, hashmap
- `get()` — return value and move node to front
- `put()` — insert/update and evict LRU if over capacity
- All four internal helpers: `_remove`, `_add_to_front`, `_evict_lru`

## 💬 What to Just Discuss
- Thread-safety with `threading.Lock()`
- TTL extension — store expiry in Node, lazy check on `get`
- How this compares to Python's `OrderedDict` (which gives you LRU for free but interviewers want you to build it)

## Core Entities
```
Node      → key, value, prev: Node, next: Node
LRUCache  → capacity: int
            cache: Dict[int, Node]
            head: Node   # dummy — MRU end
            tail: Node   # dummy — LRU end
```

## Important Methods
```python
get(key: int) -> int                     # -1 if missing; moves node to front
put(key: int, value: int) -> None        # evicts LRU if at capacity

_remove(node: Node) -> None
_add_to_front(node: Node) -> None
_evict_lru() -> None
```

## Key Design Notes
- `head.next` = most recently used; `tail.prev` = least recently used
- `get` must move accessed node to front — the most common mistake is forgetting this
- Dummy nodes eliminate all edge cases

## Common Extension
> *"Make thread-safe"* → `threading.Lock()` wrapping `get` and `put`
> *"Add TTL"* → store `expiry_time` in Node, check lazily on `get`

---
---

# 5. Ride Matching – Uber Lite ★★★★★
**Asked at:** Uber, Ola, Rapido, Porter, Dunzo.
**Pattern:** Strategy (pricing), State (driver/ride lifecycle).

## ✅ What to Actually Code (35 min)
- `DriverStatus`, `RideStatus` enums
- `Location` with `distance_to()`
- `Driver`, `Rider`, `Ride` with all fields
- `MatchingService.request_ride()` — create ride, find nearest driver, assign
- `MatchingService.complete_ride()` — update statuses, calculate fare

## 💬 What to Just Discuss
- `find_nearest_drivers` — explicitly say you're mocking this; in prod it's a geo-index (S2, QuadTree, PostGIS)
- `PricingStrategy` and how surge pricing would plug in
- `cancel_ride` — straightforward status update, not worth coding
- Ride pooling extension

## Core Entities
```
DriverStatus   → AVAILABLE / ON_TRIP / OFFLINE
RideStatus     → REQUESTED / ACCEPTED / ONGOING / COMPLETED / CANCELLED
Location       → latitude: float, longitude: float
Driver         → id, name, location, status: DriverStatus
Rider          → id, name, location
Ride           → id, rider, driver, pickup, dropoff, status: RideStatus, fare: float
```

## Important Methods
```python
# MatchingService
request_ride(rider, pickup, dropoff) -> Ride
find_nearest_drivers(location, limit) -> List[Driver]   # MOCK — sort by distance_to()
assign_driver(ride, driver) -> None
complete_ride(ride_id) -> float

# Location
distance_to(other: Location) -> float    # Euclidean — fine in interview

# PricingStrategy (abstract)
calculate_fare(pickup, dropoff, demand_factor) -> float
```

## Key Design Notes
- Driver status transitions are mandatory to code: `AVAILABLE → ON_TRIP → AVAILABLE`
- Explicitly tell interviewer `find_nearest_drivers` is mocked — shows production awareness
- `PricingStrategy` is injected — swappable without touching ride logic

## Common Extension
> *"Add ride pooling"* → `Ride.riders: List[Rider]`, capacity check in `assign_driver`

---
---

# 6. Rate Limiter ★★★★☆
**Asked at:** Amazon, Razorpay, Setu, Juspay, Hotstar.
**Pattern:** Strategy (swappable algorithm).

## ✅ What to Actually Code (35 min)
- `RateLimiter` abstract base class
- `Bucket` dataclass with `tokens`, `last_refill_time`
- `TokenBucketLimiter` fully — `_refill()` and `allow_request()`
- `RateLimiterFactory`

## 💬 What to Just Discuss
- `SlidingWindowLimiter` design — describe the `deque` approach, don't code it unless asked
- Distributed rate limiting with Redis — atomic INCR + EXPIRE
- Why token bucket allows bursts; why sliding window doesn't

## Core Entities
```python
# Bucket (per user)
tokens: float
last_refill_time: float

# TokenBucketLimiter
capacity: int
refill_rate: float               # tokens/second
user_buckets: Dict[str, Bucket]

# SlidingWindowLimiter
limit: int
window_seconds: int
user_requests: Dict[str, deque]
```

## Important Methods
```python
# RateLimiter (abstract)
allow_request(user_id: str) -> bool

# TokenBucketLimiter
_get_or_create_bucket(user_id) -> Bucket
_refill(bucket: Bucket) -> None
allow_request(user_id: str) -> bool
```

## Key Design Notes
- `_refill`: `tokens = min(capacity, tokens + refill_rate * elapsed_time)`
- Per-user buckets — each user has independent quota
- `RateLimiterFactory` returns correct impl — callers never depend on concrete class

## Common Extension
> *"Make distributed"* → Redis INCR + EXPIRE per user key; compare-and-swap for token bucket

---
---

# 7. Notification Service ★★★★☆
**Asked at:** Every product company.
**Pattern:** Observer, Factory.

## ✅ What to Actually Code (35 min)
- `ChannelType` enum
- `Notification` dataclass
- `NotificationChannel` abstract class with `send()`
- `EmailChannel` and `SMSChannel` concrete implementations
- `NotificationService` with `register_channel()` and `send()`

## 💬 What to Just Discuss
- `PushChannel` — same pattern, trivial to add
- `UserPreferences` and multi-channel fan-out
- Retry logic on failure
- Priority levels extension

## Core Entities
```
ChannelType       → EMAIL / SMS / PUSH
Notification      → id, user_id, message, channel_type: ChannelType, timestamp
NotificationService → channels: Dict[ChannelType, NotificationChannel]
```

## Important Methods
```python
# NotificationService
send(notification: Notification) -> None
register_channel(channel_type, channel: NotificationChannel) -> None

# NotificationChannel (abstract)
send(notification: Notification) -> bool
```

## Key Design Notes
- `send()` does a dict lookup — zero if-else on channel type anywhere in the codebase
- Adding WhatsApp channel = one new subclass + one `register_channel` call
- This is the cleanest Observer/Factory combo at this level

## Common Extension
> *"Critical alerts go through all channels regardless of user preference"* → add `priority` field to `Notification`

---
---

# 8. Vending Machine ★★★☆☆
**Asked at:** Amazon, Paytm, Flipkart.
**Pattern:** State Machine.

## ✅ What to Actually Code (35 min)
- `VendingState` abstract class
- `IdleState`, `HasMoneyState`, `DispensingState` — all three
- `VendingMachine` delegating every action to `self.state`
- `Inventory` with `has_stock()` and `deduct()`

## 💬 What to Just Discuss
- `OutOfStockState` — mention it, describe how machine transitions to it
- Admin restock mode extension
- How you'd add a new state without touching `VendingMachine`

## Core Entities
```
Product         → code, name, price: float
Inventory       → items: Dict[str, Tuple[Product, int]]
VendingMachine  → state: VendingState, inserted_money: float,
                  inventory: Inventory, selected_product: Optional[Product]
```

## Important Methods
```python
# VendingMachine — pure delegation, zero logic here
insert_money(amount) -> None       # delegates to self.state.insert_money(self, amount)
select_product(code) -> None
dispense() -> Optional[Product]
cancel() -> float

# VendingState (abstract)
insert_money(machine, amount) -> None
select_product(machine, code) -> None
dispense(machine) -> None
cancel(machine) -> float
```

## Key Design Notes
- The entire point: `VendingMachine` has **zero if-else** — every action is delegated to state
- States that don't support an action raise `InvalidStateError`
- Transitions: `Idle → HasMoney → Dispensing → Idle`

## Common Extension
> *"Admin restock mode"* → `AdminState` — blocks all user actions, only allows `restock(code, qty)`

---
---

# 9. Job Scheduler ★★★☆☆
**Asked at:** Razorpay, Juspay, Zepto, Curefit.
**Pattern:** Command, Priority Queue.

## ✅ What to Actually Code (35 min)
- `JobStatus` enum
- `Job` dataclass with `task: Callable`, `execution_time`, `priority`
- `Scheduler.schedule()` — enqueue to priority queue
- `Scheduler._dispatch_pending()` — pop jobs due now, assign to free worker
- `Worker.execute()` — calls `job.run()`, sets status

## 💬 What to Just Discuss
- Recurring jobs — re-enqueue with `execution_time += interval` after completion
- Retry logic — re-enqueue with backoff on failure
- Background thread for `_dispatch_pending` — mention `threading.Thread`, don't implement
- Cron expression parsing

## Core Entities
```
JobStatus   → PENDING / RUNNING / COMPLETED / FAILED / CANCELLED
Job         → id, task: Callable, execution_time: datetime, priority: int, status: JobStatus
Scheduler   → job_queue: PriorityQueue, workers: List[Worker]
Worker      → id, is_busy: bool
```

## Important Methods
```python
# Scheduler
schedule(job: Job) -> str
_dispatch_pending() -> None          # pop jobs where execution_time <= now
cancel(job_id) -> bool

# Worker
execute(job: Job) -> None
is_available() -> bool

# Job
run() -> None                        # calls self.task()
```

## Key Design Notes
- `PriorityQueue` ordered by `(execution_time, priority)` — sooner time first
- `_dispatch_pending` checks `job.execution_time <= datetime.now()` before dispatching
- Workers are independent — one blocked job doesn't stall others

## Common Extension
> *"Add cron support"* → parse cron string to compute next `execution_time`; re-enqueue after each run

---
---

# 10. Food Delivery – Swiggy Lite ★★★☆☆
**Asked at:** Swiggy, Zomato, Zepto, Dunzo, Blinkit.
**Pattern:** Multi-entity, State (order lifecycle), Strategy (discounts).

## ✅ What to Actually Code (35 min)
- `OrderStatus`, `DeliveryStatus` enums
- `MenuItem`, `OrderItem`, `Order` with fields
- `Order.add_item()` and `Order.calculate_total(discount)`
- `DeliveryService.place_order()` and `update_order_status()`
- One `DiscountStrategy` implementation

## 💬 What to Just Discuss
- `assign_agent` — same mocked geo-search as Uber Lite, mention it explicitly
- Full `OrderStatus` state machine transitions
- `cancel_order` validation — only before `OUT_FOR_DELIVERY`
- Promo code extension

## Core Entities
```
OrderStatus      → PLACED / CONFIRMED / PREPARING / OUT_FOR_DELIVERY / DELIVERED / CANCELLED
DeliveryStatus   → AVAILABLE / ASSIGNED / ON_DELIVERY
Order            → id, customer, restaurant, items: List[OrderItem], status, total, delivery_agent
OrderItem        → menu_item: MenuItem, quantity, price
DeliveryAgent    → id, name, location, status: DeliveryStatus
```

## Important Methods
```python
# DeliveryService
place_order(customer, restaurant_id, items) -> Order
assign_agent(order: Order) -> Optional[DeliveryAgent]
update_order_status(order_id, status: OrderStatus) -> None

# Order
calculate_total(discount: DiscountStrategy) -> float
add_item(item: OrderItem) -> None

# DiscountStrategy (abstract)
apply(order_total: float) -> float
```

## Key Design Notes
- `CANCELLED` only valid before `OUT_FOR_DELIVERY` — enforce this in `update_order_status`
- `assign_agent` is mocked — say this upfront, same as Uber Lite
- `DiscountStrategy` injected into `calculate_total` — no discount logic inside `Order`

## Common Extension
> *"Add promo codes"* → `PromoCode` entity maps to a `DiscountStrategy`

---
---

# 11. In-Memory File System ★★★☆☆
**Asked at:** Google, Microsoft.
**Pattern:** Composite (File and Directory as uniform Node).

## ✅ What to Actually Code (35 min)
- `FileSystemNode` abstract with `get_size()` and `is_directory()`
- `File` and `Directory` concrete classes
- `FileSystem.mkdir()`, `create_file()`, `ls()`
- `Directory.get_size()` — recursive sum of children
- Path parser: split path string, traverse from root

## 💬 What to Just Discuss
- `find()` — describe DFS traversal, don't code unless time allows
- `delete` for non-empty directories — recursive, mention it
- `move(src, dest)` — detach from old parent, attach to new
- Permissions extension

## Core Entities
```
FileSystemNode → name: str, parent: Optional[Directory]
File           → content: str, size: int
Directory      → children: Dict[str, FileSystemNode]
FileSystem     → root: Directory
```

## Important Methods
```python
# FileSystem
mkdir(path: str) -> None
create_file(path: str, content: str) -> None
ls(path: str) -> List[str]
read(path: str) -> str

# Directory
add_node(node: FileSystemNode) -> None
get_node(name: str) -> Optional[FileSystemNode]

# FileSystemNode (abstract)
get_size() -> int              # File: self.size  |  Directory: sum(child.get_size())
is_directory() -> bool
```

## Key Design Notes
- `get_size()` on Directory is recursive — this is the Composite pattern in one method
- Path parsing: `"/a/b/c".split("/")` → traverse from root, step by step
- Callers never do `isinstance` checks — always call `get_size()` on `FileSystemNode`

## Common Extension
> *"Add move"* → detach from old parent's `children`, attach to new parent
> *"Add permissions"* → `permissions: Dict[user_id, Set[Permission]]` on each node

---
---

# 12. Logger Framework ★★★☆☆
**Asked at:** Adobe, Oracle, Salesforce, infra rounds.
**Pattern:** Chain of Responsibility, Singleton.

## ✅ What to Actually Code (35 min)
- `LogLevel` enum
- `LogMessage` dataclass
- `LogHandler` abstract with `handle()`, `set_next()`, `_write()`
- `ConsoleHandler` and `FileHandler` concrete classes
- `Logger` with `log()`, `add_handler()`, and convenience methods
- `LogManager` Singleton with `get_logger()`

## 💬 What to Just Discuss
- Async logging — `LogMessage` onto a `Queue`, background thread drains it
- Log rotation in `FileHandler`
- Adding a new handler (e.g. `AlertHandler`) — zero changes to existing code

## Core Entities
```
LogLevel     → DEBUG=1 / INFO=2 / WARNING=3 / ERROR=4 / CRITICAL=5
LogMessage   → level: LogLevel, message: str, timestamp: datetime, logger_name: str
Logger       → name, level: LogLevel, handlers: List[LogHandler]
LogHandler   → level: LogLevel, next_handler: Optional[LogHandler]
```

## Important Methods
```python
# Logger
log(level: LogLevel, message: str) -> None
debug / info / warning / error / critical(msg) -> None   # call self.log()
add_handler(handler: LogHandler) -> None

# LogHandler (abstract)
handle(log_message: LogMessage) -> None     # check level → _write() → pass to next
set_next(handler: LogHandler) -> LogHandler
_write(log_message: LogMessage) -> None     # subclasses implement

# LogManager (Singleton)
get_logger(name: str) -> Logger
```

## Key Design Notes
- `handle()`: if `log_message.level >= self.level` → call `_write()`, then call `next_handler.handle()`
- `set_next()` returns `next_handler` for fluent chaining: `console.set_next(file).set_next(alert)`
- `LogManager` caches by name — same name always returns same `Logger`

## Common Extension
> *"Async logging"* → handler enqueues `LogMessage`; background thread drains and writes

---
---

# 13. Chess ★★☆☆☆
**Asked at:** Google, Microsoft, Atlassian.
**Pattern:** OOP + Rules Engine.

## ✅ What to Actually Code (35 min)
- `Color`, `GameStatus` enums
- `Position` dataclass
- `Piece` abstract with `get_possible_moves(board)`
- `King`, `Rook`, `Bishop` — implement `get_possible_moves()` for these three only
- `Board` with `get_piece()`, `move_piece()`, `is_within_bounds()`
- `Chess.make_move()` — validate, execute, check for check

## 💬 What to Just Discuss
- Remaining piece types — same pattern, trivially extendable
- `is_checkmate()` — describe the algorithm (no valid moves AND in check)
- Castling, en passant — acknowledge, say out of scope
- Full game loop

## Core Entities
```
Color         → WHITE / BLACK
GameStatus    → IN_PROGRESS / CHECK / CHECKMATE / STALEMATE
Position      → row: int, col: int
Piece         → color: Color, position: Position, has_moved: bool
Board         → grid: List[List[Optional[Piece]]]
```

## Important Methods
```python
# Chess
make_move(player, from_pos, to_pos) -> bool
get_valid_moves(piece) -> List[Position]   # possible - moves leaving King in check
is_in_check(color) -> bool

# Piece (abstract)
get_possible_moves(board: Board) -> List[Position]

# Board
get_piece(pos) -> Optional[Piece]
move_piece(from_pos, to_pos) -> None
is_within_bounds(pos) -> bool
```

## Key Design Notes
- Implement 3 piece types max — that's enough to show the pattern
- `get_valid_moves` = `get_possible_moves` filtered by moves that don't leave own King in check
- Tell interviewer explicitly: *"I'll implement King, Rook, Bishop — others follow the same pattern"*

## Common Extension
> *"Add castling"* → check `King.has_moved` and `Rook.has_moved`, validate empty squares between

---
---

# 14. Pub-Sub System – Kafka Lite ★★☆☆☆
**Asked at:** Amazon, Flipkart, Hotstar, Razorpay.
**Pattern:** Observer at system/service level.

## ✅ What to Actually Code (35 min)
- `Message` dataclass
- `Topic` with `messages`, `subscribers`, `add_message()`, `notify_subscribers()`
- `Subscriber` abstract with `on_message()`
- `PubSubBroker` Singleton with `create_topic()`, `publish()`, `subscribe()`
- One concrete `Subscriber` (e.g. `LoggingSubscriber`)

## 💬 What to Just Discuss
- `Subscription.offset` for message replay — describe it, don't code
- Pull model vs push model tradeoff
- Consumer groups — one offset shared across a group of subscribers
- Persistence / durability at scale

## Core Entities
```
Message      → id, topic_name, payload, published_at: datetime
Topic        → name, messages: List[Message], subscribers: List[Subscriber]
Subscription → subscriber, topic_name, offset: int
PubSubBroker → topics: Dict[str, Topic]
```

## Important Methods
```python
# PubSubBroker (Singleton)
create_topic(name) -> Topic
publish(topic_name, message) -> None
subscribe(topic_name, subscriber) -> Subscription
unsubscribe(topic_name, subscriber) -> None

# Topic
add_message(message) -> None
notify_subscribers(message) -> None   # calls subscriber.on_message() for each

# Subscriber (abstract)
on_message(message: Message) -> None
```

## Key Design Notes
- `publish` → `topic.add_message()` → `topic.notify_subscribers()` — clean three-step flow
- Push model: broker calls `on_message` directly — simple, implement this in 45 min
- Pull model: subscriber calls `get_messages(from_offset)` — discuss as extension

## Common Extension
> *"Add consumer groups"* → group shares one offset — only one member processes each message

---
---

# 15. Twitter / Social Media Feed ★★☆☆☆
**Asked at:** Meta, Twitter/X, LinkedIn, ShareChat, Koo.
**Pattern:** Multi-pattern — Fan-out, Observer, basic feed ranking.

## ✅ What to Actually Code (35 min)
- `Tweet` dataclass
- `User` with `followers`, `following`
- `TweetService.post_tweet()` and `like_tweet()`
- `FollowService.follow()` and `unfollow()`
- `FeedService.get_feed()` — fan-out on write: push tweet to all followers' feeds on post

## 💬 What to Just Discuss
- Fan-out on read — pull and merge followees' timelines on request
- Hybrid model — fan-out on write for normal users, fan-out on read for celebrities with millions of followers
- Feed ranking — reverse-chronological is fine to implement; ML ranking is out of scope
- Trending hashtags extension

## Core Entities
```
Tweet      → id, author_id, content, created_at, like_count
User       → id, username, followers: Set[str], following: Set[str]
Feed       → user_id, tweets: List[Tweet]   # pre-built per user in write model
FeedService → user_feeds: Dict[str, List[Tweet]]
```

## Important Methods
```python
# TweetService
post_tweet(user_id, content) -> Tweet
like_tweet(user_id, tweet_id) -> None
delete_tweet(tweet_id) -> None

# FollowService
follow(follower_id, followee_id) -> None
unfollow(follower_id, followee_id) -> None

# FeedService
get_feed(user_id, page, page_size) -> List[Tweet]
_fan_out_on_write(tweet: Tweet) -> None    # push to all followers' feeds on post
```

## Key Design Notes
- Fan-out on write: on `post_tweet`, push tweet to every follower's feed list — fast reads
- The celebrity problem: user with 10M followers makes fan-out on write unusable — this is what the interviewer wants you to raise
- In 45 min: implement fan-out on write only; discuss hybrid verbally

## Common Extension
> *"Trending hashtags"* → `HashtagService` counts frequency per tag in a sliding time window

---
---

## Preparation Strategy

### Phase 1 — Weeks 1–2 (Problems 1–7)
These are fully codeable in 45 min. Solve cold with a timer. Review design notes after.
Priority order: LRU Cache → Parking Lot → Notification Service → Splitwise → Rate Limiter → Vending Machine → BookMyShow

### Phase 2 — Week 3 (Problems 8–12)
These need scoping discipline. Practice saying *"I'll implement X, and just discuss Y"* out loud.

### Phase 3 — Week 4 (Problems 13–15)
These are SDE 2 differentiators. Chess and Twitter will separate you from candidates who only prepped the standard 7.

---

## Common Extensions — One Per Problem

| Problem | Extension |
|---------|-----------|
| Parking Lot | Live display board → Observer |
| Splitwise | Minimize settlement transactions → greedy graph |
| BookMyShow | Waitlist for sold-out shows |
| LRU Cache | Thread-safe + TTL expiry |
| Ride Matching | Ride pooling |
| Rate Limiter | Distributed with Redis |
| Notification Service | Priority levels |
| Vending Machine | Admin restock mode |
| Job Scheduler | Cron expression support |
| Food Delivery | Promo codes |
| In-Memory File System | Move + permissions |
| Logger Framework | Async logging with queue |
| Chess | Castling |
| Pub-Sub | Consumer groups |
| Twitter Feed | Hybrid fan-out for celebrities |
