# High-Yield LLD / OOP Python — 15 Questions Ranked by Importance
### Target: SDE 1–2 Years | FAANG · Top PBC · India Startups · Remote Roles

---

## How to Read This Sheet

Every problem has a **45-Min Scope** section that tells you exactly:
- ✅ **Code this** — write actual working Python classes and methods
- 💬 **Discuss this** — say it out loud, draw it, mention it — don't code it

Interviewers evaluate how you think, not whether your code compiles.
The goal in 45 min is clean class skeletons + 2–3 core methods fully implemented + intelligent discussion of the rest.

---

## Master Ranking Table

| Rank | Problem | Primary Pattern | Asked At |
|------|---------|----------------|----------|
| 1 | Parking Lot | Factory, OOP basics | Everywhere — most asked LLD ever |
| 2 | Splitwise | Strategy | CRED, Razorpay, PhonePe, Groww |
| 3 | BookMyShow | Concurrency, Factory | Swiggy, Meesho, Nykaa, Juspay |
| 4 | LRU Cache | DSA-as-LLD | Amazon, Flipkart, almost all Indian cos |
| 5 | Ride Matching (Uber Lite) | Multi-pattern, State | Uber, Ola, Rapido, Porter |
| 6 | Rate Limiter | Strategy | Every infra/backend/scale role |
| 7 | Notification Service | Observer | Every product company |
| 8 | Vending Machine | State Machine | Amazon, Paytm, Flipkart |
| 9 | Job Scheduler | Command, Priority Queue | Razorpay, Juspay, Zepto |
| 10 | Food Delivery (Swiggy Lite) | Multi-pattern, State | Swiggy, Zomato, Zepto, Dunzo |
| 11 | In-Memory File System | Composite | Google, Microsoft |
| 12 | Logger Framework | Chain of Responsibility | Adobe, Oracle, infra rounds |
| 13 | Chess | OOP + Rules Engine | Google, Microsoft, Atlassian |
| 14 | Pub-Sub System (Kafka Lite) | Observer at system level | Amazon, Flipkart, Hotstar |
| 15 | Twitter / Social Media Feed | Multi-pattern, Feed design | Meta, LinkedIn, ShareChat |

---
---

# 1. Parking Lot ★★★★★

**Asked at:** Literally every company. Blanking on this is an instant red flag.
**Patterns:** Factory, Singleton, basic OOP.

## 45-Min Scope

✅ **Code this:**
- `VehicleType` enum, `SpotStatus` enum
- `ParkingSpot`, `Vehicle` (abstract + one subclass), `Ticket` with all fields
- `ParkingFloor` with `get_free_spot(vehicle_type)`
- `ParkingLot` Singleton with `park_vehicle()`, `unpark_vehicle()`, `find_available_spot()`
- One concrete `PricingStrategy` — just `HourlyPricing.calculate_fare()`

💬 **Discuss this:**
- How you'd add a second pricing strategy without touching `ParkingLot`
- Why Singleton — and what breaks if you don't use it
- How a display board would work (Observer — floors notify board on status change)

## Core Entities
- `ParkingLot` *(Singleton)*
- `ParkingFloor`
- `ParkingSpot`
- `Vehicle` *(abstract)* → `Car` / `Bike` / `Truck`
- `Ticket`
- `PricingStrategy` *(abstract)* → `HourlyPricing` / `FlatPricing`

## Key Fields
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
# ParkingLot (Singleton)
park_vehicle(vehicle: Vehicle) -> Optional[Ticket]
unpark_vehicle(ticket: Ticket) -> float
find_available_spot(vehicle_type: VehicleType) -> Optional[ParkingSpot]
get_available_count(vehicle_type: VehicleType) -> int

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
**Patterns:** Strategy (split types), Singleton (ExpenseManager).

## 45-Min Scope

✅ **Code this:**
- `User`, `Group`, `Expense` with all fields
- `Split` abstract class with `validate()` and `calculate_amounts()`
- `EqualSplit` fully implemented — this is the one you write completely
- `PercentSplit` — implement `validate()` (sum must equal 100); stub `calculate_amounts()`
- `ExpenseManager` Singleton with `add_expense()` and `get_user_balance()`

💬 **Discuss this:**
- How `ExactSplit` differs from `EqualSplit`
- How balances are stored as net amounts and why
- Debt simplification extension — greedy graph approach (don't code it, just explain)

## Core Entities
- `User`
- `Group`
- `Expense`
- `Split` *(abstract)* → `EqualSplit` / `ExactSplit` / `PercentSplit`
- `ExpenseManager` *(Singleton)*

## Key Fields
```
User           → id, name
Group          → id, members: List[User], expenses: List[Expense]
Expense        → id, amount, paid_by: User, splits: List[Split]
Split          → user: User, amount: float
ExpenseManager → groups: Dict[str, Group], balances: Dict[str, Dict[str, float]]
```

## Important Methods
```python
# ExpenseManager (Singleton)
add_expense(group_id, expense: Expense) -> None
settle_balance(user_a: User, user_b: User, amount: float) -> None
get_user_balance(user_id: str) -> Dict[str, float]
get_group_balances(group_id: str) -> Dict[str, Dict[str, float]]

# Split (abstract)
validate() -> bool
calculate_amounts(total) -> None
```

## Key Design Notes
- Balances are **net** — if A owes B ₹100 and B owes A ₹40, store net ₹60
- `validate()` called before `calculate_amounts()` — fail fast
- Adding a new split type = new subclass only, zero changes to `ExpenseManager`

## Common Extension
> *"Minimize settlement transactions"* → net each balance, greedily pair max-creditor with max-debtor

---
---

# 3. BookMyShow – Ticket Booking ★★★★★

**Asked at:** Swiggy, Meesho, Nykaa, Juspay, MakeMyTrip.
**Patterns:** Factory, Singleton, Concurrency.

## 45-Min Scope

✅ **Code this:**
- All enums: `SeatType`, `SeatStatus`, `BookingStatus`
- `Movie`, `Screen`, `Seat`, `Show` with fields
- `BookingManager` Singleton skeleton
- `lock_seats()` fully implemented with `threading.Lock()` per show — this is what the interviewer actually cares about
- `confirm_booking()` with lazy TTL check

💬 **Discuss this:**
- `Payment` abstract class and subclasses — just name them, don't implement
- How you'd handle lock expiry with a background thread
- Why per-show lock instead of one global lock — throughput argument
- `search_shows()` — mention you'd use DB filters in prod, stub it here

## Core Entities
- `Movie`
- `Theatre` / `Screen`
- `Show`
- `Seat`
- `Booking`
- `BookingManager` *(Singleton)*
- `Payment` *(abstract)* → `UPIPayment` / `CardPayment`

## Key Fields
```
SeatType       → REGULAR / PREMIUM / RECLINER
SeatStatus     → AVAILABLE / LOCKED / BOOKED
BookingStatus  → PENDING / CONFIRMED / CANCELLED
Show           → id, movie, screen, start_time, seat_map: Dict[str, SeatStatus],
                 lock: threading.Lock, lock_expiry: Dict[str, datetime]
Booking        → id, user, show, seats: List[Seat], status: BookingStatus, payment
```

## Important Methods
```python
# BookingManager (Singleton)
search_shows(movie_id, date, city) -> List[Show]
lock_seats(show_id, seat_ids, user_id) -> bool   # thread-safe; 10 min TTL
confirm_booking(booking_id, payment) -> Booking
cancel_booking(booking_id) -> None
release_locks(show_id, seat_ids) -> None

# Show
get_available_seats() -> List[Seat]
is_seat_available(seat_id) -> bool
```

## Key Design Notes
- `lock_seats` is the critical section — `threading.Lock()` per show, not one global lock
- Seat locking has a TTL — lazy expiry check inside `confirm_booking`
- The interviewer will ask the double-booking question — your answer is the per-show lock

## Common Extension
> *"Add a waitlist for sold-out shows"* → queue per show; on cancellation, notify first in queue

---
---

# 4. LRU Cache ★★★★★

**Asked at:** Amazon, Flipkart, Hotstar, Dunzo — almost every Indian product company.
**Patterns:** DSA-as-LLD.

## 45-Min Scope

✅ **Code this:**
- `Node` with all DLL fields
- `LRUCache.__init__` with dummy head/tail setup
- `_remove(node)`, `_add_to_front(node)` — the DLL helpers
- `get(key)` and `put(key, value)` fully implemented end-to-end

💬 **Discuss this:**
- Why DLL + HashMap and not just an OrderedDict (Python gotcha — mention OrderedDict is valid too but interviewers want you to know the underlying structure)
- Thread-safety extension — just say `threading.Lock()` wrapping both methods
- TTL extension — add `expiry_time` to Node, check on `get`

## Core Entities
- `LRUCache`
- `Node` *(DLL node)*

## Key Fields
```
Node      → key, value, prev: Node, next: Node
LRUCache  → capacity: int
            cache: Dict[int, Node]
            head: Node                   # dummy — MRU end
            tail: Node                   # dummy — LRU end
```

## Important Methods
```python
get(key: int) -> int                     # -1 if not found; moves node to front
put(key: int, value: int) -> None        # evict LRU if over capacity

_remove(node: Node) -> None
_add_to_front(node: Node) -> None
_evict_lru() -> None
```

## Key Design Notes
- `head.next` = most recently used; `tail.prev` = least recently used
- `get` must move accessed node to front — forgetting this is the most common mistake
- Dummy head/tail eliminate edge cases for empty list

## Common Extension
> *"Thread-safe version"* → `threading.Lock()` wrapping `get` and `put`
> *"TTL cache"* → add `expiry_time` to Node; check on `get`, lazy evict

---
---

# 5. Ride Matching – Uber Lite ★★★★★

**Asked at:** Uber, Ola, Rapido, Porter, Dunzo.
**Patterns:** Multi-pattern — Strategy (pricing), State (ride/driver lifecycle).

## 45-Min Scope

✅ **Code this:**
- All enums: `DriverStatus`, `RideStatus`
- `Location` with `distance_to()` using simple Euclidean
- `Driver`, `Rider`, `Ride` with all fields
- `MatchingService.request_ride()` and `find_nearest_drivers()` — mock the geo-search by sorting on distance
- `complete_ride()` with status transitions and fare calculation

💬 **Discuss this:**
- `PricingStrategy` — name the subclasses, explain the interface, don't implement surge logic
- Why `find_nearest_drivers` is mocked — say you'd use a geospatial index (PostGIS, Redis GEO) in prod
- Driver status state machine transitions
- Ride pooling extension — how `Ride` entity changes

## Core Entities
- `Rider`, `Driver`, `Ride`, `Location`
- `MatchingService`
- `PricingStrategy` *(abstract)* → `SurgePricing` / `FlatPricing`

## Key Fields
```
DriverStatus   → AVAILABLE / ON_TRIP / OFFLINE
RideStatus     → REQUESTED / ACCEPTED / ONGOING / COMPLETED / CANCELLED
Location       → latitude: float, longitude: float
Driver         → id, name, location, status: DriverStatus, rating: float
Ride           → id, rider, driver, pickup, dropoff, status: RideStatus, fare: float
```

## Important Methods
```python
# MatchingService
request_ride(rider, pickup, dropoff) -> Ride
find_nearest_drivers(location, limit) -> List[Driver]
assign_driver(ride, driver) -> None
complete_ride(ride_id) -> float
cancel_ride(ride_id, cancelled_by) -> None

# Location
distance_to(other: Location) -> float

# PricingStrategy (abstract)
calculate_fare(pickup, dropoff, demand_factor) -> float
```

## Key Design Notes
- Explicitly tell interviewer `find_nearest_drivers` is mocked — shows awareness
- Driver status transitions are strict — going from `OFFLINE` to `ON_TRIP` directly should raise an error

## Common Extension
> *"Add ride pooling"* → `Ride.riders: List[Rider]`, capacity check in `assign_driver`

---
---

# 6. Rate Limiter ★★★★☆

**Asked at:** Amazon, Razorpay, Setu, Juspay, Hotstar.
**Patterns:** Strategy, Abstract class.

## 45-Min Scope

✅ **Code this:**
- `RateLimiter` ABC with `allow_request()` as the only contract
- `Bucket` dataclass with `tokens`, `last_refill_time`
- `TokenBucketLimiter` fully — `_refill()` and `allow_request()` with per-user buckets
- `RateLimiterFactory.get_limiter()`

💬 **Discuss this:**
- `SlidingWindowLimiter` — explain the deque approach, don't fully implement unless time permits
- Token bucket vs sliding window tradeoff — burst traffic vs strict uniformity
- Distributed extension — Redis INCR + EXPIRE, mention atomicity requirement

## Core Entities
- `RateLimiter` *(abstract)*
- `TokenBucketLimiter` / `SlidingWindowLimiter`
- `Bucket`
- `RateLimiterFactory`

## Key Fields
```python
Bucket                 → tokens: float, last_refill_time: float
TokenBucketLimiter     → capacity: int, refill_rate: float, user_buckets: Dict[str, Bucket]
SlidingWindowLimiter   → limit: int, window_seconds: int, user_requests: Dict[str, deque]
```

## Important Methods
```python
# RateLimiter (abstract)
allow_request(user_id: str) -> bool

# TokenBucketLimiter
_refill(bucket: Bucket) -> None
allow_request(user_id: str) -> bool

# SlidingWindowLimiter
_evict_old(user_id, now) -> None
allow_request(user_id: str) -> bool
```

## Key Design Notes
- Token bucket: refill tokens based on elapsed time since `last_refill_time`; cap at `capacity`
- Sliding window: evict timestamps older than `now - window_seconds`; count remaining

## Common Extension
> *"Distributed rate limiter"* → Redis INCR + EXPIRE per user key; Lua script for atomicity

---
---

# 7. Notification Service ★★★★☆

**Asked at:** Every product company with any user-facing feature.
**Patterns:** Observer, Factory.

## 45-Min Scope

✅ **Code this:**
- `ChannelType` enum, `Notification` dataclass
- `NotificationChannel` ABC with `send()` abstract method
- `EmailChannel` and `SMSChannel` concrete implementations
- `NotificationService` with `register_channel()` and `send()`
- `ChannelFactory.get_channel()`

💬 **Discuss this:**
- `UserPreferences` entity and how `send_to_user` respects preferred channels
- Retry logic on failed send — don't implement, just explain the wrapper approach
- How adding `WhatsAppChannel` requires zero changes to `NotificationService`

## Core Entities
- `NotificationService`
- `Notification`
- `NotificationChannel` *(abstract)* → `EmailChannel` / `SMSChannel` / `PushChannel`
- `UserPreferences`

## Key Fields
```
ChannelType      → EMAIL / SMS / PUSH
Notification     → id, user_id, message, channel_type: ChannelType, timestamp
UserPreferences  → user_id, preferred_channels: List[ChannelType]
```

## Important Methods
```python
# NotificationService
send(notification: Notification) -> None
register_channel(channel_type, channel: NotificationChannel) -> None

# NotificationChannel (abstract)
send(notification: Notification) -> bool

# ChannelFactory
get_channel(channel_type: ChannelType) -> NotificationChannel
```

## Key Design Notes
- `NotificationService` holds `Dict[ChannelType, NotificationChannel]` — zero if-else on channel type
- Adding a new channel = new subclass + one `register_channel()` call

## Common Extension
> *"Add priority — critical alerts bypass user preferences and use all channels"*

---
---

# 8. Vending Machine ★★★☆☆

**Asked at:** Amazon, Paytm, Flipkart.
**Patterns:** State Machine.

## 45-Min Scope

✅ **Code this:**
- `Product`, `Inventory` with all fields and methods
- `VendingState` ABC with all action signatures
- `IdleState` and `HasMoneyState` fully implemented
- `VendingMachine` with `set_state()` and all delegating action methods

💬 **Discuss this:**
- `DispensingState` and `OutOfStockState` — describe transitions, don't fully code
- Why this is better than a giant if-else in `VendingMachine`
- Admin mode extension — new `AdminState` that blocks user actions

## Core Entities
- `VendingMachine`
- `VendingState` *(abstract)* → `IdleState` / `HasMoneyState` / `DispensingState` / `OutOfStockState`
- `Product`, `Inventory`

## Key Fields
```
Product         → code: str, name: str, price: float
Inventory       → items: Dict[str, Tuple[Product, int]]
VendingMachine  → state: VendingState, inserted_money: float,
                  inventory: Inventory, selected_product: Optional[Product]
```

## Important Methods
```python
# VendingMachine — all delegate to self.state
insert_money(amount: float) -> None
select_product(code: str) -> None
dispense() -> Optional[Product]
cancel() -> float

# VendingState (abstract)
insert_money(machine, amount) -> None
select_product(machine, code) -> None
dispense(machine) -> None
cancel(machine) -> float
```

## Key Design Notes
- Zero if-else in `VendingMachine` — every action delegates to current state
- States that don't support an action raise `InvalidStateError`
- Transitions: `Idle → HasMoney → Dispensing → Idle`

---
---

# 9. Job Scheduler ★★★☆☆

**Asked at:** Razorpay, Juspay, Zepto, Curefit.
**Patterns:** Command, Priority Queue.

## 45-Min Scope

✅ **Code this:**
- `JobStatus` enum, `Job` dataclass with all fields
- `Job.run()` with try/except and retry decrement logic
- `Scheduler` with `schedule()` and `_dispatch_pending()` using `PriorityQueue`
- `Worker.execute()` with busy flag management

💬 **Discuss this:**
- Background thread loop calling `_dispatch_pending()` — describe it, don't implement threading boilerplate
- Recurring jobs — re-enqueue with `execution_time += interval` after completion
- Cron expression extension — mention you'd parse cron string to next datetime

## Core Entities
- `Job`, `Scheduler`, `Worker`
- `JobStatus` *(enum)*

## Key Fields
```
JobStatus  → PENDING / RUNNING / COMPLETED / FAILED / CANCELLED
Job        → id, task: Callable, execution_time: datetime,
             priority: int, status: JobStatus, retry_count: int, max_retries: int
Scheduler  → job_queue: PriorityQueue, workers: List[Worker]
Worker     → id, is_busy: bool, current_job: Optional[Job]
```

## Important Methods
```python
# Scheduler
schedule(job: Job) -> str
_dispatch_pending() -> None

# Worker
execute(job: Job) -> None
is_available() -> bool

# Job
run() -> None    # calls task(); on exception → retry if count > 0
```

## Key Design Notes
- `PriorityQueue` orders by `(execution_time, priority)` — sooner first, lower number = higher priority
- On failure: re-enqueue with backoff if `retry_count > 0`

---
---

# 10. Food Delivery – Swiggy Lite ★★★☆☆

**Asked at:** Swiggy, Zomato, Zepto, Dunzo, Blinkit.
**Patterns:** Multi-entity, State (order lifecycle), Strategy (discounts).

## 45-Min Scope

✅ **Code this:**
- All enums: `OrderStatus`, `DeliveryStatus`
- `MenuItem`, `OrderItem`, `Order` with fields
- `Order.calculate_total()` with injected `DiscountStrategy`
- `DeliveryService.place_order()` and `update_order_status()` with state validation
- `assign_agent()` — sort available agents by distance, assign first

💬 **Discuss this:**
- `DiscountStrategy` subclasses — name them, show the interface, skip implementation
- Why `CANCELLED` is only valid before `OUT_FOR_DELIVERY` — state machine constraint
- `assign_agent` geo-search is mocked — same caveat as Uber Lite
- Promo code extension

## Core Entities
- `Customer`, `Restaurant`, `MenuItem`
- `Order`, `OrderItem`, `DeliveryAgent`
- `DeliveryService`
- `DiscountStrategy` *(abstract)*

## Key Fields
```
OrderStatus     → PLACED / CONFIRMED / PREPARING / OUT_FOR_DELIVERY / DELIVERED / CANCELLED
DeliveryStatus  → AVAILABLE / ASSIGNED / ON_DELIVERY
Order           → id, customer, restaurant, items: List[OrderItem],
                  status: OrderStatus, total, delivery_agent
OrderItem       → menu_item: MenuItem, quantity, price
```

## Important Methods
```python
# DeliveryService
place_order(customer, restaurant_id, items) -> Order
assign_agent(order: Order) -> Optional[DeliveryAgent]
update_order_status(order_id, status: OrderStatus) -> None
cancel_order(order_id) -> bool

# Order
calculate_total(discount: DiscountStrategy) -> float

# DiscountStrategy (abstract)
apply(order_total: float) -> float
```

## Key Design Notes
- State transitions are strict — validate in `update_order_status` before applying
- `DiscountStrategy` injected into `calculate_total` — no discount logic inside `Order`

---
---

# 11. In-Memory File System ★★★☆☆

**Asked at:** Google, Microsoft.
**Patterns:** Composite.

## 45-Min Scope

✅ **Code this:**
- `FileSystemNode` abstract class with `get_size()` and `is_directory()`
- `File` with `content`, `size`, implementing `get_size()`
- `Directory` with `children: Dict`, implementing recursive `get_size()`
- `FileSystem` with `mkdir()`, `create_file()`, `ls()`, `read()`
- Path parsing helper — split by `"/"` and traverse from root

💬 **Discuss this:**
- `delete()` — recursive delete for directories; describe the logic
- `find(name)` — DFS traversal returning all matching paths
- `move(src, dest)` — detach from old parent, attach to new
- Permission extension — `Dict[user_id, Permission]` on each node

## Core Entities
- `FileSystemNode` *(abstract)*
- `File(FileSystemNode)`
- `Directory(FileSystemNode)`
- `FileSystem`

## Key Fields
```
FileSystemNode → name: str, parent: Optional[Directory], created_at: datetime
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
delete(path: str) -> None

# FileSystemNode (abstract)
get_size() -> int
is_directory() -> bool

# Directory
add_node(node) -> None
get_node(name) -> Optional[FileSystemNode]
```

## Key Design Notes
- `Directory.get_size()` is recursive — sum of all children's `get_size()`
- Composite: callers use `FileSystemNode` uniformly — no `isinstance` checks in traversal
- Path parsing: split on `"/"`, walk from root, raise error if node missing

---
---

# 12. Logger Framework ★★★☆☆

**Asked at:** Adobe, Oracle, Salesforce — infra/platform rounds.
**Patterns:** Chain of Responsibility, Singleton.

## 45-Min Scope

✅ **Code this:**
- `LogLevel` enum with numeric values
- `LogMessage` dataclass
- `LogHandler` ABC with `handle()`, `set_next()`, and abstract `_write()`
- `ConsoleHandler` fully implemented
- `Logger` with `log()`, `add_handler()`, `set_level()`
- `LogManager` Singleton with `get_logger()`

💬 **Discuss this:**
- `FileHandler` — same structure as `ConsoleHandler`, just writes to file
- Chain setup: `console.set_next(file).set_next(alert_handler)` — fluent chaining
- Why Chain of Responsibility over a simple list of handlers — each handler decides whether to pass on
- Async logging extension — handler pushes to a `Queue`; background thread drains it

## Core Entities
- `Logger`, `LogManager` *(Singleton)*
- `LogHandler` *(abstract)* → `ConsoleHandler` / `FileHandler`
- `LogMessage`, `LogLevel`

## Key Fields
```
LogLevel    → DEBUG=1 / INFO=2 / WARNING=3 / ERROR=4 / CRITICAL=5
LogMessage  → level: LogLevel, message: str, timestamp: datetime, logger_name: str
Logger      → name: str, level: LogLevel, handlers: List[LogHandler]
LogHandler  → level: LogLevel, next_handler: Optional[LogHandler]
```

## Important Methods
```python
# Logger
log(level: LogLevel, message: str) -> None
add_handler(handler: LogHandler) -> None

# LogHandler (abstract)
handle(log_message: LogMessage) -> None   # check level → _write → pass to next
set_next(handler: LogHandler) -> LogHandler
_write(log_message: LogMessage) -> None   # subclasses implement

# LogManager (Singleton)
get_logger(name: str) -> Logger
```

## Key Design Notes
- `handle()`: if `log_message.level >= self.level` → call `_write()`, then call `next_handler.handle()`
- `set_next()` returns `next_handler` for fluent chaining
- `LogManager` caches by name — same name always returns same `Logger` instance

---
---

# 13. Chess ★★☆☆☆

**Asked at:** Google, Microsoft, Atlassian.
**Patterns:** OOP + Rules Engine.

## 45-Min Scope

✅ **Code this:**
- `Color` enum, `Position` dataclass, `GameStatus` enum
- `Piece` abstract class with `get_possible_moves()` abstract
- `King`, `Rook`, `Bishop` — implement `get_possible_moves()` for these three only
- `Board` with `grid`, `get_piece()`, `move_piece()`, `is_within_bounds()`
- `Chess.make_move()` — validate turn, call `get_valid_moves`, apply move

💬 **Discuss this:**
- Remaining piece types — same structure, different movement logic
- `is_in_check()` — describe the approach: after each move, check if own King is attacked
- `is_checkmate()` — no valid moves left while in check
- Castling — `has_moved` flag on King and Rook, no pieces between them

## Core Entities
- `Chess` *(controller)*
- `Board`, `Piece` *(abstract)*, `Player`, `Move`
- Pieces: `King` / `Queen` / `Rook` / `Bishop` / `Knight` / `Pawn`

## Key Fields
```
Color       → WHITE / BLACK
Position    → row: int, col: int
Piece       → color: Color, position: Position, has_moved: bool
Board       → grid: List[List[Optional[Piece]]]
```

## Important Methods
```python
# Chess
make_move(player, from_pos, to_pos) -> bool
get_valid_moves(piece: Piece) -> List[Position]
is_in_check(color: Color) -> bool
is_checkmate(color: Color) -> bool

# Piece (abstract)
get_possible_moves(board: Board) -> List[Position]

# Board
get_piece(pos) -> Optional[Piece]
move_piece(from_pos, to_pos) -> None
is_within_bounds(pos) -> bool
```

## Key Design Notes
- Implement 3 piece types in 45 min — tell interviewer the rest follow the same pattern
- `get_valid_moves` = `get_possible_moves` filtered by moves that don't leave own King in check
- Don't implement a full engine — clean class hierarchy + move validation is the goal

---
---

# 14. Pub-Sub System – Kafka Lite ★★☆☆☆

**Asked at:** Amazon, Flipkart, Hotstar, Razorpay — backend/event-driven roles.
**Patterns:** Observer at system/service level.

## 45-Min Scope

✅ **Code this:**
- `Message` dataclass, `Topic` with `messages` and `subscribers`
- `Subscriber` ABC with `on_message()`
- `PubSubBroker` Singleton with `create_topic()`, `publish()`, `subscribe()`, `unsubscribe()`
- `Topic.notify_subscribers()` — push model, calls each subscriber's `on_message()`
- One concrete `Subscriber` implementation as a demo

💬 **Discuss this:**
- `Subscription.offset` for message replay — describe the field, don't build replay logic
- Pull model alternative — `get_messages(from_offset)` — explain tradeoff vs push
- Consumer group extension — shared offset, only one member processes each message
- How this differs from `NotificationService` — service-to-service vs user-facing push

## Core Entities
- `Topic`, `Message`
- `Publisher`, `Subscriber` *(abstract)*
- `PubSubBroker` *(Singleton)*
- `Subscription`

## Key Fields
```
Message      → id, topic_name, payload, published_at: datetime
Topic        → name: str, messages: List[Message], subscribers: List[Subscriber]
Subscription → subscriber, topic_name, offset: int
PubSubBroker → topics: Dict[str, Topic]
```

## Important Methods
```python
# PubSubBroker (Singleton)
create_topic(name: str) -> Topic
publish(topic_name: str, message: Message) -> None
subscribe(topic_name: str, subscriber: Subscriber) -> Subscription
unsubscribe(topic_name: str, subscriber: Subscriber) -> None

# Subscriber (abstract)
on_message(message: Message) -> None

# Topic
add_message(message: Message) -> None
notify_subscribers(message: Message) -> None
```

## Key Design Notes
- Push model: `publish()` → `topic.notify_subscribers()` → each subscriber's `on_message()`
- `Subscription.offset` enables replay but implementing full replay is out of 45-min scope
- Clearly different from Notification Service — this is service-to-service, not user-facing

---
---

# 15. Design Twitter (LeetCode #355) ★★☆☆☆

**Asked at:** Amazon, Google, Uber — any company that wants DSA wrapped in OOP.
**Patterns:** OOP + Heap Merge (k-sorted lists). This is a coding problem, not a system design problem. The trick is `getNewsFeed` — merge tweet timelines of all followed users and return the 10 most recent by timestamp.

## 45-Min Scope

✅ **Code this:**
- `Tweet` dataclass with `tweet_id`, `user_id`, `timestamp`
- `User` with `tweets: List[Tweet]` and `following: Set[int]`
- `Twitter` class — the single controller
- `postTweet()` — prepend to user's tweet list, increment global timestamp
- `follow()` / `unfollow()` — set operations on `user.following`
- `getNewsFeed()` — **this is the whole problem** — heap merge of followed users' timelines, return top 10

💬 **Discuss this:**
- Why a max-heap on timestamp — merging k sorted lists efficiently
- Why you keep only the latest 10 per user during heap construction — pruning
- How this would change at scale — mention fan-out, don't implement

## Core Entities
- `Tweet`
- `User`
- `Twitter` *(single controller — maps to LeetCode class interface)*

## Key Fields
```
Tweet    → tweet_id: int, user_id: int, timestamp: int
User     → user_id: int, tweets: List[Tweet], following: Set[int]
Twitter  → users: Dict[int, User], timestamp: int   # global monotonic counter
```

## Important Methods
```python
# Twitter
postTweet(user_id: int, tweet_id: int) -> None
getNewsFeed(user_id: int) -> List[int]     # returns up to 10 most recent tweet_ids
follow(follower_id: int, followee_id: int) -> None
unfollow(follower_id: int, followee_id: int) -> None

# getNewsFeed — the core implementation
def getNewsFeed(self, user_id: int) -> List[int]:
    # 1. Collect candidate users = following + self
    # 2. For each, take their most recent tweet (index 0) → push to max-heap
    #    heap entry: (-timestamp, tweet_id, user_id, tweet_index)
    # 3. Pop heap up to 10 times; after each pop push next tweet from same user
    # 4. Return collected tweet_ids
```

## Full Implementation
```python
import heapq
from collections import defaultdict

class Tweet:
    def __init__(self, tweet_id, user_id, timestamp):
        self.tweet_id = tweet_id
        self.user_id = user_id
        self.timestamp = timestamp

class User:
    def __init__(self, user_id):
        self.user_id = user_id
        self.tweets = []          # ordered newest-first
        self.following = set()

class Twitter:
    def __init__(self):
        self.users = {}           # user_id → User
        self.timestamp = 0

    def _get_or_create(self, user_id) -> User:
        if user_id not in self.users:
            self.users[user_id] = User(user_id)
        return self.users[user_id]

    def postTweet(self, user_id: int, tweet_id: int) -> None:
        user = self._get_or_create(user_id)
        user.tweets.insert(0, Tweet(tweet_id, user_id, self.timestamp))
        self.timestamp += 1

    def getNewsFeed(self, user_id: int) -> list[int]:
        user = self._get_or_create(user_id)
        candidates = user.following | {user_id}

        # max-heap: store as negative timestamp for min-heap to act as max-heap
        heap = []
        for uid in candidates:
            u = self._get_or_create(uid)
            if u.tweets:
                t = u.tweets[0]
                heapq.heappush(heap, (-t.timestamp, t.tweet_id, uid, 0))

        feed = []
        while heap and len(feed) < 10:
            neg_ts, tweet_id, uid, idx = heapq.heappop(heap)
            feed.append(tweet_id)
            next_idx = idx + 1
            u = self.users[uid]
            if next_idx < len(u.tweets):
                t = u.tweets[next_idx]
                heapq.heappush(heap, (-t.timestamp, t.tweet_id, uid, next_idx))

        return feed

    def follow(self, follower_id: int, followee_id: int) -> None:
        self._get_or_create(follower_id).following.add(followee_id)

    def unfollow(self, follower_id: int, followee_id: int) -> None:
        self._get_or_create(follower_id).following.discard(followee_id)
```

## Key Design Notes
- Global `timestamp` counter makes recency comparison trivial — no datetime needed
- Heap entry is `(-timestamp, tweet_id, user_id, index)` — negative timestamp turns min-heap into max-heap
- After each pop, push the **next tweet from the same user** at `index + 1` — this is the k-sorted merge pattern
- Time complexity: `O(N log K)` where N=10 results, K=number of followed users

## Common Extension
> *"What if a user follows 10,000 people?"* → Initial heap push is O(10,000 log 10,000) — discuss caching the feed or fan-out on write at scale

---
---

## Preparation Strategy

### Phase 1 — Weeks 1–2 (Problems 1–7)
Covers ~80% of what gets asked. Solve each one cold under a 45-min timer.
Use the scope cuts — don't gold-plate. Stop when the ✅ items are done.

### Phase 2 — Week 3 (Problems 8–12)
State machines (8), Command/PQ (9), multi-entity (10), Composite (11), Chain (12).
These feel harder but the patterns repeat — once you see it, you see it everywhere.

### Phase 3 — Week 4 (Problems 13–15)
Chess, Pub-Sub, Twitter. Solve these to differentiate at SDE 2-leaning rounds.
For Twitter especially — practise the fan-out discussion out loud. The talking is the interview.

---

## Common Extensions — Be Ready for Every Problem

| Problem | Extension |
|---------|-----------|
| Parking Lot | Live display board → Observer |
| Splitwise | Minimize settlement transactions → greedy graph |
| BookMyShow | Waitlist for sold-out shows |
| LRU Cache | Thread-safe / add TTL expiry |
| Ride Matching | Ride pooling / post-ride ratings |
| Rate Limiter | Distributed with Redis |
| Notification Service | Priority levels / retry on failure |
| Vending Machine | Admin restock mode → new AdminState |
| Job Scheduler | Cron expression support |
| Food Delivery | Promo codes / real-time tracking |
| In-Memory File System | move(src, dest) / per-user permissions |
| Logger Framework | Async logging with queue |
| Chess | Castling / en passant |
| Pub-Sub | Consumer groups / message replay |
| Twitter Feed | Trending hashtags / hybrid fan-out |
