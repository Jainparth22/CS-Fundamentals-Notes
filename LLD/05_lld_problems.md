# LLD Problems — Class Designs

> Each problem: clarify requirements → identify key entities → define interfaces → design class diagram → code core logic.

---

## 1. Parking Lot

**Requirements:** Multiple floors, multiple spots per floor, different vehicle types (car, bike, truck), issue/release tickets, track availability.

```python
from enum import Enum
from datetime import datetime
from typing import Optional, Dict, List

class VehicleType(Enum): BIKE = 1; CAR = 2; TRUCK = 3

class Vehicle:
    def __init__(self, plate: str, vtype: VehicleType):
        self.plate = plate
        self.type = vtype

class ParkingSpot:
    def __init__(self, spot_id: str, vtype: VehicleType):
        self.spot_id = spot_id
        self.allowed_type = vtype
        self.vehicle: Optional[Vehicle] = None

    @property
    def is_free(self) -> bool: return self.vehicle is None

    def park(self, v: Vehicle): self.vehicle = v
    def release(self): self.vehicle = None

class Ticket:
    def __init__(self, ticket_id: str, spot: ParkingSpot, vehicle: Vehicle):
        self.ticket_id = ticket_id
        self.spot = spot
        self.vehicle = vehicle
        self.entry_time = datetime.now()
        self.exit_time: Optional[datetime] = None

    def close(self) -> float:
        self.exit_time = datetime.now()
        hours = (self.exit_time - self.entry_time).seconds / 3600
        return max(1, hours) * 20  # ₹20/hour, min 1 hour

class ParkingFloor:
    def __init__(self, floor_id: int, spots: List[ParkingSpot]):
        self.floor_id = floor_id
        self.spots = spots

    def find_free_spot(self, vtype: VehicleType) -> Optional[ParkingSpot]:
        return next((s for s in self.spots if s.allowed_type == vtype and s.is_free), None)

class ParkingLot:  # Singleton
    _instance = None

    @classmethod
    def get_instance(cls) -> 'ParkingLot':
        if not cls._instance:
            cls._instance = ParkingLot()
        return cls._instance

    def __init__(self):
        self.floors: List[ParkingFloor] = []
        self.active_tickets: Dict[str, Ticket] = {}
        self._ticket_counter = 0

    def park(self, vehicle: Vehicle) -> Optional[Ticket]:
        for floor in self.floors:
            spot = floor.find_free_spot(vehicle.type)
            if spot:
                spot.park(vehicle)
                self._ticket_counter += 1
                ticket = Ticket(f"TKT-{self._ticket_counter:06}", spot, vehicle)
                self.active_tickets[ticket.ticket_id] = ticket
                return ticket
        return None  # lot full

    def exit(self, ticket_id: str) -> float:
        ticket = self.active_tickets.pop(ticket_id, None)
        if not ticket:
            raise ValueError("Invalid ticket")
        fee = ticket.close()
        ticket.spot.release()
        return fee
```

---

## 2. Library Management System

**Key entities:** Book, BookCopy, Member, Librarian, Loan, Catalog.

```python
class Book:
    def __init__(self, isbn: str, title: str, author: str):
        self.isbn = isbn; self.title = title; self.author = author
        self.copies: List['BookCopy'] = []

class BookCopy:
    def __init__(self, copy_id: str, book: Book):
        self.copy_id = copy_id; self.book = book
        self.is_available = True

class Loan:
    def __init__(self, copy: BookCopy, member: 'Member'):
        self.copy = copy; self.member = member
        self.due_date = datetime.now() + timedelta(days=14)
        self.returned_at: Optional[datetime] = None

    def return_book(self):
        self.returned_at = datetime.now()
        self.copy.is_available = True
        fine = 0
        if self.returned_at > self.due_date:
            days_late = (self.returned_at - self.due_date).days
            fine = days_late * 10  # ₹10/day
        return fine
```

---

## 3. ATM System

**States:** Idle, CardInserted, PINVerified, SelectingTransaction, Dispensing.

```python
class ATM:
    def __init__(self):
        self._state: ATMState = IdleState(self)
        self._balance = 50000  # cash in ATM
        self._current_account: Optional[str] = None

    def insert_card(self, card_no: str): self._state.insert_card(card_no)
    def enter_pin(self, pin: str): self._state.enter_pin(pin)
    def withdraw(self, amount: int): self._state.withdraw(amount)
    def eject_card(self): self._state.eject_card()
    def set_state(self, state: 'ATMState'): self._state = state

# State pattern for ATM states (see behavioral patterns)
```

---

## 4. Elevator System

**Key entities:** Elevator, ElevatorController, Request.

**Scheduling algorithm:** SCAN (elevator door algorithm):
- Move in one direction, pick up requests, reverse at end.

```python
class Direction(Enum): UP = 1; DOWN = -1; IDLE = 0

class Request:
    def __init__(self, floor: int, direction: Direction):
        self.floor = floor; self.direction = direction

class Elevator:
    def __init__(self, elevator_id: int, max_floor: int):
        self.id = elevator_id
        self.current_floor = 0
        self.direction = Direction.IDLE
        self.requests: List[int] = []

    def add_request(self, floor: int): heapq.heappush(self.requests, floor)

    def move(self):
        if not self.requests: self.direction = Direction.IDLE; return
        target = self.requests[0]
        if target > self.current_floor: self.current_floor += 1; self.direction = Direction.UP
        elif target < self.current_floor: self.current_floor -= 1; self.direction = Direction.DOWN
        else: heapq.heappop(self.requests)  # arrived
```

---

## 5. Chess Game

**Key entities:** Board, Piece (King/Queen/Bishop/Knight/Rook/Pawn), Player, Game.

```python
class Position:
    def __init__(self, row: int, col: int): self.row = row; self.col = col
    def is_valid(self): return 0 <= self.row < 8 and 0 <= self.col < 8

class Piece(ABC):
    def __init__(self, color: str, pos: Position):
        self.color = color; self.pos = pos
    @abstractmethod
    def valid_moves(self, board: 'Board') -> List[Position]: ...

class Knight(Piece):
    def valid_moves(self, board: 'Board') -> List[Position]:
        deltas = [(-2,-1),(-2,1),(-1,-2),(-1,2),(1,-2),(1,2),(2,-1),(2,1)]
        moves = []
        for dr, dc in deltas:
            p = Position(self.pos.row + dr, self.pos.col + dc)
            if p.is_valid():
                occupant = board.get(p)
                if not occupant or occupant.color != self.color:
                    moves.append(p)
        return moves
```

---

## 6. Online Food Delivery (Swiggy/Zomato Style)

**Entities:** User, Restaurant, MenuItem, Order, OrderItem, DeliveryAgent, Payment.

**State machine for Order:**
```
PLACED → CONFIRMED → PREPARING → READY → PICKED_UP → DELIVERED
       ↘ CANCELLED (from PLACED or CONFIRMED only)
```

---

## 7. Movie Ticket Booking (BookMyShow Style)

```python
class Show:
    def __init__(self, movie: str, hall: int, start_time: datetime):
        self.movie = movie; self.hall = hall; self.start_time = start_time
        self.seats: Dict[str, 'Seat'] = {}

class Seat:
    def __init__(self, seat_id: str, seat_type: str, price: float):
        self.seat_id = seat_id; self.type = seat_type; self.price = price
        self.status = "AVAILABLE"  # AVAILABLE, LOCKED, BOOKED

class BookingService:
    def lock_seats(self, show: Show, seat_ids: List[str], user_id: str, ttl_seconds: int = 300):
        """Lock seats for N seconds while user pays."""
        for sid in seat_ids:
            seat = show.seats[sid]
            if seat.status != "AVAILABLE":
                raise ValueError(f"Seat {sid} not available")
            seat.status = "LOCKED"
        # schedule unlock after TTL (use Redis EXPIRY in production)

    def confirm_booking(self, show: Show, seat_ids: List[str]) -> str:
        for sid in seat_ids:
            show.seats[sid].status = "BOOKED"
        return f"BOOKING-{uuid.uuid4().hex[:8].upper()}"
```

---

## 8. Rate Limiter

**Algorithms:**
- Token Bucket (burst-friendly)
- Leaky Bucket (smooth output)
- Fixed Window Counter
- Sliding Window Log
- Sliding Window Counter

```python
import time
from collections import deque

class SlidingWindowRateLimiter:
    """Allow max `limit` requests per `window` seconds per key."""
    def __init__(self, limit: int, window: int):
        self.limit = limit; self.window = window
        self.requests: Dict[str, deque] = {}

    def is_allowed(self, key: str) -> bool:
        now = time.time()
        if key not in self.requests:
            self.requests[key] = deque()
        q = self.requests[key]
        while q and q[0] < now - self.window:
            q.popleft()  # remove expired
        if len(q) < self.limit:
            q.append(now)
            return True
        return False
```

---

## 9. Cache — LRU Implementation

(See Hash Tables notes for full implementation)

---

## 10. Pub/Sub Message Broker

```python
class MessageBroker:
    def __init__(self):
        self._topics: Dict[str, List[Callable]] = {}

    def subscribe(self, topic: str, handler: Callable):
        self._topics.setdefault(topic, []).append(handler)

    def publish(self, topic: str, message: dict):
        for handler in self._topics.get(topic, []):
            handler(message)

broker = MessageBroker()
broker.subscribe("order.placed", lambda m: print(f"Email: order {m['id']}"))
broker.subscribe("order.placed", lambda m: print(f"SMS: order {m['id']}"))
broker.publish("order.placed", {"id": "ORD-001", "total": 500})
```
