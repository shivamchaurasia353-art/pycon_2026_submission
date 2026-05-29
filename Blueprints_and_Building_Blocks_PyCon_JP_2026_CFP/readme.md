# PyCon JP 2026 — CFP Submission
# Blueprints & Building Blocks: Architectural Patterns vs Design Patterns in Python

**Conference:** PyCon JP 2026  
**Dates:** August 21–22, 2026  
**Venue:** International Conference Center Hiroshima  
**Submit via:** Pretalx (pretalx.com/pycon-jp-2026)  
**Deadline:** May 31, 2026 (AoE) = June 1, 2026 09:00 JST  

---

## 1. Submission Metadata

| Field | Value |
|---|---|
| **Title** | Blueprints & Building Blocks: Architectural Patterns vs Design Patterns in Python |
| **Speaker** | Shivam Chaurasia |
| **Affiliation** | Solution Architect 1, EPAM Systems, India |
| **Category** | Python Core and Ecosystem |
| **Audience Level** | Intermediate |
| **Duration** | 30 minutes |
| **45-min option** | Yes — can accommodate 45 minutes if requested |
| **Language** | English |
| **Slides language** | English (key terms annotated in Japanese) |
| **Python version** | 3.10+ (uses `Protocol`, structural pattern matching optional) |

---

## 2. Pretalx Form Fields *(copy-paste ready)*

### Abstract
*(Limit: 800 characters)*

> Every Python developer knows what a design pattern is. Fewer can explain why applying Strategy everywhere did not make their system easier to change — and almost nobody can articulate the difference between a design pattern and an architectural pattern without hesitating.
>
> This talk cuts through that confusion using the Three Altitudes framework: Idioms, Design Patterns, and Architectural Patterns are three different tools for three different scopes. Mix up the altitude and you get either over-engineered code or an under-designed system.
>
> We build four design patterns in Python — Strategy, Observer, Decorator, Command — then step up to two architectural patterns: Hexagonal Architecture and CQRS. Live code throughout. You leave with a decision framework you can apply to your next code review on Monday.

**Character count: ~743 / 800**

---

### Your experience with this topic
*(Limit: 500 characters)*

> As a Solution Architect at EPAM Systems I produce Solution Architecture Documents and lead whiteboarding sessions with client engineering teams weekly. The confusion between design and architectural patterns comes up in nearly every engagement — developers applying Observer at the system boundary, architects drawing class diagrams and calling them architecture. I have spent eleven years finding the right altitude for the right pattern, and this talk is that framework distilled.

**Character count: ~461 / 500**

---

### What discussions can you have with attendees?
*(Limit: 500 characters)*

> I want to hear: Have you ever applied a design pattern that made the code more complex, not less? Does your team have an architecture document that is actually a class diagram? Where in your system do you wish you had drawn a harder boundary? These are questions every engineering team debates and rarely resolves cleanly — and the Three Altitudes framework gives us a shared vocabulary to have that conversation at the right level.

**Character count: ~428 / 500**

---

### Additional fields

| Field | Answer |
|---|---|
| First time at PyCon JP? | [Fill in] |
| Travel grant needed? | [Check if applicable] |
| Code of Conduct | Agree |
| LLM data consent | Agree |

---

## 3. Core Thesis

> "Design patterns solve code-level problems. Architectural patterns solve system-level problems. Confusing the two leads to over-engineered code or under-designed systems."

---

## 4. The Three Altitudes Framework *(Talk Foundation)*

```
Altitude 3 — ARCHITECTURAL PATTERNS
             "How does the system hold together?"
             Scope: cross-service, cross-module, system-level boundaries
             Examples: Hexagonal, CQRS, Event-Driven, Layered, Microservices

             ↑ above this line = architectural decisions

Altitude 2 — DESIGN PATTERNS
             "How does this component work?"
             Scope: class/module level, within a service boundary
             Examples: Strategy, Observer, Decorator, Command, Factory

             ↑ above this line = design decisions

Altitude 1 — IDIOMS
             "How do we write this in Python?"
             Scope: language-specific expression, Pythonic conventions
             Examples: context managers, decorators, dataclasses, Protocols
```

**The key insight:**
- Design patterns operate **inside** a service boundary
- Architectural patterns **define** the boundaries themselves
- Applying a design pattern at architectural scope = over-engineering
- Applying an architectural pattern at class scope = misuse

---

## 5. Talk Outline — 30 Minutes

| Time | Section | Content |
|---|---|---|
| 0:00–3:00 | Section 1 — The Confusion Hook | Code review scenario; team debate moment |
| 3:00–7:00 | Section 2 — Three Altitudes | Framework introduction; scope definitions |
| 7:00–17:00 | Section 3 — Design Patterns in Python | Strategy, Observer, Decorator, Command (live code) |
| 17:00–25:00 | Section 4 — Architectural Patterns | Hexagonal, CQRS (live code) |
| 25:00–28:00 | Section 5 — Interaction Layer + Decision Matrix | Where the patterns meet; scope decision table |
| 28:00–30:00 | Section 6 — Closing + Discussion | Key message; QR code; hallway prompt |

---

## 6. Detailed Section Notes

### Section 1 — The Confusion Hook `[0:00–3:00]`

- Open with a real code review scenario: a "Strategy pattern" that spans three services
- Show an actual architecture diagram that is a class diagram with arrows renamed
- Question posed to audience: *"What is the difference between a design pattern and an architectural pattern?"*
- Preview the Three Altitudes framework as the talk's navigation tool

---

### Section 2 — The Three Altitudes `[3:00–7:00]`

- Introduce the three levels with examples from real codebases
- Demonstrate the scope mismatch: Observer used to wire two microservices together (design pattern at architectural altitude)
- Demonstrate the inverse: drawing Hexagonal Architecture boxes around 3 classes in a single module (architectural pattern at code altitude)
- Core rule: *"Match the tool to the scope."*

---

### Section 3 — Design Patterns in Python `[7:00–17:00]`

#### 3a — Strategy Pattern `[7:00–9:30]`

**Problem:** Swapping algorithms at runtime without if/else chains.

```python
# Python-first: callables are strategies — no abstract class needed
from typing import Callable

def fixed_discount(price: float) -> float:
    return price * 0.9

def seasonal_discount(price: float) -> float:
    return price * 0.75

class PriceCalculator:
    def __init__(self, strategy: Callable[[float], float]) -> None:
        self._strategy = strategy

    def calculate(self, price: float) -> float:
        return self._strategy(price)

calc = PriceCalculator(strategy=seasonal_discount)
print(calc.calculate(100.0))    # 75.0
```

**Pythonic insight:** Functions are first-class — no `IStrategy` interface or abstract base class needed.  
**Altitude:** Design (class scope, inside one service).  
**Use when:** Swappable algorithms, A/B testing, rule engines.

---

#### 3b — Observer / Event Bus `[9:30–11:30]`

**Problem:** Decoupling producers from consumers without direct import coupling.

```python
from typing import Protocol, Callable

class EventHandler(Protocol):
    def __call__(self, event: dict) -> None: ...

class EventBus:
    def __init__(self) -> None:
        self._handlers: dict[str, list[EventHandler]] = {}

    def subscribe(self, event_type: str, handler: EventHandler) -> None:
        self._handlers.setdefault(event_type, []).append(handler)

    def publish(self, event_type: str, payload: dict) -> None:
        for handler in self._handlers.get(event_type, []):
            handler(payload)

bus = EventBus()
bus.subscribe("order.placed", lambda e: print(f"Email sent for order {e['id']}"))
bus.subscribe("order.placed", lambda e: print(f"Audit logged: {e['id']}"))
bus.publish("order.placed", {"id": "ORD-001", "total": 99.0})
```

**Pythonic insight:** `Protocol` enables structural typing — subscribers don't inherit from anything.  
**Altitude:** Design (module scope, within a service).  
**Use when:** Audit logging, notifications, side-effect isolation, decoupled listeners.

---

#### 3c — Decorator Pattern `[11:30–13:30]`

**Problem:** Adding cross-cutting concerns without touching the original function.

```python
import time, functools, logging

logger = logging.getLogger(__name__)

def timed(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = fn(*args, **kwargs)
        elapsed = time.perf_counter() - start
        logger.info("%s completed in %.3fs", fn.__name__, elapsed)
        return result
    return wrapper

def memoize(fn):
    cache: dict = {}
    @functools.wraps(fn)
    def wrapper(*args):
        if args not in cache:
            cache[args] = fn(*args)
        return cache[args]
    return wrapper

@timed
@memoize
def compute_report(year: int, month: int) -> dict:
    # expensive computation
    return {"year": year, "month": month, "rows": 10_000}
```

**Pythonic insight:** Python's `@decorator` syntax is first-class language support for this pattern.  
**Altitude:** Design (function scope, within a module).  
**Use when:** Logging, timing, retry, caching — any cross-cutting concern.

---

#### 3d — Command Pattern `[13:30–17:00]`

**Problem:** Encapsulating actions as objects for undo, audit trails, and message queuing.

```python
from dataclasses import dataclass
from typing import Any

@dataclass(frozen=True)
class TransferFundsCommand:
    from_account: str
    to_account: str
    amount: float

@dataclass(frozen=True)
class CancelOrderCommand:
    order_id: str
    reason: str

class CommandBus:
    def __init__(self) -> None:
        self._handlers: dict[type, Any] = {}
        self._audit_log: list[tuple[str, Any]] = []

    def register(self, cmd_type: type, handler) -> None:
        self._handlers[cmd_type] = handler

    def execute(self, command) -> Any:
        handler = self._handlers[type(command)]
        result = handler(command)
        self._audit_log.append((type(command).__name__, command))
        return result

bus = CommandBus()
bus.register(TransferFundsCommand, lambda cmd: print(f"Transfer ${cmd.amount}"))
bus.execute(TransferFundsCommand("ACC-001", "ACC-002", 500.0))
```

**Pythonic insight:** `@dataclass(frozen=True)` gives immutable, hashable, loggable commands with zero boilerplate.  
**Altitude:** Design (module scope, within a service).  
**Use when:** Undo/redo, audit trails, message queues, CQRS write side.

---

### Section 4 — Architectural Patterns in Python `[17:00–25:00]`

#### 4a — Hexagonal Architecture (Ports & Adapters) `[17:00–21:00]`

**Problem:** Business logic coupled to frameworks and databases — hard to test, impossible to swap.

```
            ┌─────────────────────────────┐
            │         Core Domain         │
            │  (pure Python — zero        │
            │   imports from Flask,       │
            │   SQLAlchemy, or httpx)     │
            └──────────┬──────────────────┘
                       │ Ports (Protocols)
          ┌────────────┼──────────────────────┐
          │            │                      │
   ┌──────┴──────┐ ┌───┴──────┐ ┌────────────┴────┐
   │  FastAPI    │ │  CLI     │ │  SQLAlchemy      │
   │  Adapter   │ │  Adapter │ │  Adapter         │
   │  (primary) │ │  (primary│ │  (secondary)     │
   └─────────────┘ └──────────┘ └─────────────────┘
```

```python
from typing import Protocol
from dataclasses import dataclass

# ── Domain Model ──────────────────────────────────────────
@dataclass
class User:
    id: int
    name: str
    email: str

# ── Port — defined in core domain ─────────────────────────
class UserRepository(Protocol):
    def find(self, user_id: int) -> User | None: ...
    def save(self, user: User) -> None: ...

# ── Use Case — zero infrastructure imports ─────────────────
class CreateUserUseCase:
    def __init__(self, repo: UserRepository) -> None:
        self._repo = repo

    def execute(self, name: str, email: str) -> User:
        user = User(id=1, name=name, email=email)
        self._repo.save(user)
        return user

# ── Adapter — in-memory (used in tests) ───────────────────
class InMemoryUserRepository:
    def __init__(self) -> None:
        self._store: dict[int, User] = {}

    def find(self, user_id: int) -> User | None:
        return self._store.get(user_id)

    def save(self, user: User) -> None:
        self._store[user.id] = user

# Test — no database, no HTTP, no mocking framework needed
repo    = InMemoryUserRepository()
use_case = CreateUserUseCase(repo)
user    = use_case.execute("Alice", "alice@example.com")
assert user.name == "Alice"
```

**Key points:**
- `Protocol` defines the port — adapters satisfy it structurally, no inheritance
- The core domain has zero framework imports — it is purely Python
- Swap the database adapter for a test double with a one-line change at the call site

**Altitude:** Architectural (defines module/service boundaries).  
**Use when:** Any system where domain logic must survive infrastructure changes.

---

#### 4b — CQRS: Command Query Responsibility Segregation `[21:00–25:00]`

**Problem:** One model trying to serve reads and writes leads to compromises on both.

```
Write side                         Read side
──────────                         ─────────
PlaceOrderCommand                  OrderSummaryQuery
       │                                  │
       ▼                                  ▼
 Command Handler                    Query Handler
(validates, persists,           (optimised read model,
 publishes OrderPlaced event)    denormalised for UI)
```

```python
from dataclasses import dataclass
from typing import Protocol

# ── Write side ────────────────────────────────────────────
@dataclass(frozen=True)
class PlaceOrderCommand:
    customer_id: str
    items: tuple[str, ...]
    total: float

class OrderWriteRepository(Protocol):
    def save(self, order_id: str, command: PlaceOrderCommand) -> None: ...

def handle_place_order(cmd: PlaceOrderCommand, repo: OrderWriteRepository) -> str:
    order_id = f"ORD-{hash(cmd.customer_id)}"
    repo.save(order_id, cmd)
    return order_id

# ── Read side ─────────────────────────────────────────────
@dataclass(frozen=True)
class OrderSummaryQuery:
    order_id: str

@dataclass
class OrderSummary:           # optimised for display — not for mutation
    order_id: str
    customer_name: str
    total_items: int
    status: str
    formatted_total: str      # pre-formatted for the UI

class OrderReadRepository(Protocol):
    def get_summary(self, order_id: str) -> OrderSummary | None: ...

def handle_order_summary(query: OrderSummaryQuery, repo: OrderReadRepository) -> OrderSummary | None:
    return repo.get_summary(query.order_id)
```

**Key points:**
- Read and write models evolve independently — add a display field without touching the write model
- Read side can be a Redis cache, materialised view, or Elasticsearch index
- The Command pattern (from Section 3d) naturally maps onto the CQRS write side — this is where design and architectural patterns intersect

**Altitude:** Architectural (system-level boundary decision).  
**Use when:** High read/write asymmetry, complex read projections, independent scaling needs.

---

### Section 5 — Interaction Layer + Decision Matrix `[25:00–28:00]`

#### Where design and architectural patterns meet

```
Hexagonal core
    └── uses Strategy to swap pricing rules        ← design pattern inside architectural boundary
    └── uses Command as use-case input object      ← design pattern inside architectural boundary
    └── uses Observer to publish domain events     ← design pattern inside architectural boundary

CQRS write side
    └── uses Command pattern for write operations  ← naming intentionally aligned across altitudes
```

The patterns are not mutually exclusive — architectural patterns **define** where service boundaries are; design patterns **work inside** those boundaries.

#### Decision Framework

```
Ask: "What scope is this problem at?"

      Code / class scope?
          → Design Pattern
          (Strategy / Observer / Decorator / Command / Factory)

      Module / service boundary scope?
          → Architectural Pattern
          (Hexagonal / CQRS / Event-Driven / Layered)

      Confusing them?
          → Over-engineering code, or under-designing the system
```

#### One-slide matrix *(photographable)*

| Pattern | Altitude | Python Tool | Use When |
|---|---|---|---|
| Strategy | Design | Callable / Protocol | Swappable algorithms |
| Observer | Design | EventBus + Protocol | Decoupled side-effects |
| Decorator | Design | `@functools.wraps` | Cross-cutting concerns |
| Command | Design | Frozen dataclass + Bus | Auditable actions, queuing |
| Hexagonal | Architectural | Protocol (Ports) | Isolate domain from infrastructure |
| CQRS | Architectural | Separate read/write models | Read/write asymmetry |

---

### Section 6 — Closing + Discussion `[28:00–30:00]`

- Recap: Three Altitudes — Idioms / Design Patterns / Architectural Patterns
- Key message: *"The altitude tells you which pattern to reach for. Match the tool to the scope."*
- QR code → companion repo with all runnable Python examples
- Discussion prompt: *"In your codebase, where have you applied a design pattern at architectural scope — or vice versa?"*

---

## 7. Code Examples Summary

| File | Pattern | Type |
|---|---|---|
| `design_patterns/strategy.py` | Callable strategies, no abstract base | Design |
| `design_patterns/observer.py` | Protocol-typed EventBus | Design |
| `design_patterns/decorator_pattern.py` | `@timed`, `@memoize`, `@retry` | Design |
| `design_patterns/command.py` | Frozen dataclass + CommandBus + audit log | Design |
| `architectural_patterns/hexagonal/hexagonal_architecture.py` | Ports & Adapters with Protocol | Architectural |
| `architectural_patterns/cqrs/cqrs_pattern.py` | Split read/write models | Architectural |

---

## 8. Slide Structure

| # | Slide Title | Notes |
|---|---|---|
| 1 | Title | Title, speaker, EPAM, PyCon JP 2026, QR |
| 2 | The Confusion | Code review story — Strategy pattern across three services |
| 3 | The Question | *"What is the difference?"* — pause for audience |
| 4 | Three Altitudes | Visual ladder: Idioms → Design Patterns → Architectural Patterns |
| 5 | Altitude 1 — Idioms | Python-specific examples: context managers, dataclasses, Protocols |
| 6 | Altitude 2 — Design Patterns | Scope definition; "within a boundary" |
| 7 | Altitude 3 — Architectural Patterns | Scope definition; "defines the boundaries" |
| 8 | Strategy Pattern — Code | Callable strategies; no abstract class needed |
| 9 | Strategy — When to Use | Rule engines, A/B testing, swappable algorithms |
| 10 | Observer / EventBus — Code | Protocol handler; subscribe/publish |
| 11 | Observer — When to Use | Audit, notifications, side-effect isolation |
| 12 | Decorator — Code | `@timed` + `@memoize` stacked |
| 13 | Command — Code | Frozen dataclass + CommandBus + audit trail |
| 14 | Design Patterns Recap | Quick summary table of 4 patterns |
| 15 | Hexagonal — Diagram | Ports and Adapters visual |
| 16 | Hexagonal — Code | Protocol port; in-memory adapter; test with zero mocks |
| 17 | Hexagonal — Key Insight | Core domain: zero framework imports |
| 18 | CQRS — Diagram | Write side vs read side flow |
| 19 | CQRS — Code | Split models; separate handlers |
| 20 | Where They Meet | Design patterns working inside architectural boundaries |
| 21 | Decision Matrix | Altitude → pattern table (photographable) |
| 22 | Q&A + Discussion Prompt | *"Where have you mixed up the altitude?"* |

---

## 9. Review Criteria Self-Assessment

| Criterion | How this proposal addresses it |
|---|---|
| **Firsthand experience** | Shivam produces Solution Architecture Documents and leads whiteboarding sessions at EPAM weekly. The confusion between design and architectural patterns surfaces in nearly every client engagement — this framework is the one he actually uses. |
| **Originality** | Most talks cover design patterns OR architectural patterns. This talk explicitly addresses the confusion between them using the Three Altitudes framework — a mental model not found in GoF or Fowler's books, but built from real architectural practice. |
| **Clarity** | Structured as a progressive ladder: four design patterns (with Python-first code), then two architectural patterns, then the matrix showing how they relate. Each code sample is the minimum needed to prove the point. |
| **Interactivity** | Opens with a question the audience will have a strong opinion on. Closes with a discussion prompt that maps directly to the audience's own codebases. Every pattern demo is followed by "When to use / when not to use." |
| **Relevance** | Every Python developer writes design patterns. Every developer working in a team has encountered system-level decisions. The confusion between the two levels is universal and rarely addressed directly. |

---

## 10. Speaker Bio

### Short Bio *(≤100 words — schedule page)*

Shivam Chaurasia is a Solution Architect at EPAM Systems, India, with over eleven years of experience designing and delivering Python-driven systems for enterprise clients. He specialises in software architecture, legacy modernisation, and platform engineering. He leads architectural assessments, produces Solution Architecture Documents, and runs whiteboarding sessions with client engineering teams. His talks translate the lessons from those sessions — including the hard-won clarity on when to use a design pattern versus an architectural pattern — into frameworks that developers can apply immediately.

---

### Full Bio *(≤300 words — Pretalx speaker profile)*

Shivam Chaurasia is a Solution Architect 1 at EPAM Systems, India, with over eleven years of professional experience delivering efficient, reusable, and maintainable software solutions across tight deadlines and complex enterprise environments.

His primary competency is Python and software architecture, with deep expertise across backend engineering, platform engineering, legacy modernisation, data engineering, RESTful API design, and enterprise integration. He has managed every phase of the software development lifecycle — from requirement gathering and architectural assessment through implementation, unit testing, documentation, and production release in CI/CD environments.

The distinction between design patterns and architectural patterns is not academic for Shivam — it comes up in every Solution Architecture Document he produces, every whiteboarding session he facilitates, and every code review where a team debates whether an Observer should cross a service boundary. He has developed the Three Altitudes framework over years of explaining to client engineering teams why adding more design patterns did not make their systems more modular — because the problem was at the wrong altitude.

Beyond technical work, Shivam brings management experience in resource management, stakeholder management, staffing, negotiation, and conflict resolution — giving him the perspective of both the engineer choosing a pattern and the architect explaining its implications to the organisation.

---

### Expertise

| Domain | Skills |
|---|---|
| Engineering | Framework, Platform, Backend, Legacy Modernisation, Data Engineering |
| APIs & Integration | RESTful API Engineering, Enterprise Integration |
| Architecture | Solution Architecture, OOD, High/Low Level Design, SAD, SARR, Whiteboarding |
| Management | Resource & Stakeholder Management, Staffing, Negotiation |

---

## 11. Pre-Submission Checklist

- [ ] Abstract pasted into Pretalx — ≤800 chars (Section 2)
- [ ] Experience pasted into Pretalx — ≤500 chars (Section 2)
- [ ] Discussion pasted into Pretalx — ≤500 chars (Section 2)
- [ ] Category selected: **Python Core and Ecosystem**
- [ ] Audience level selected: **Intermediate**
- [ ] Duration: **30 minutes** (45-min option: Yes)
- [ ] First time at PyCon JP answered
- [ ] Travel grant checkbox filled
- [ ] Code of Conduct: Agree
- [ ] LLM data consent: Agree
- [ ] Speaker bio added to Pretalx profile (short bio, Section 10)
- [ ] Headshot uploaded (min 800×800 px)
- [ ] LinkedIn / GitHub links confirmed before submitting
- [ ] Code examples repo link confirmed and public
- [ ] Submitted before **May 31, 2026 (AoE) = June 1, 2026 09:00 JST**

---

## 12. References

| Resource | Link |
|---|---|
| Design Patterns: GoF | Gamma, Helm, Johnson, Vlissides — Addison-Wesley 1994 |
| Patterns of Enterprise Application Architecture | Martin Fowler — Addison-Wesley 2002 |
| Architecture Patterns with Python | Percival & Gregory — O'Reilly 2020 (free online) |
| Python `typing.Protocol` docs | docs.python.org/3/library/typing.html#typing.Protocol |
| Hexagonal Architecture (original) | alistair.cockburn.us/hexagonal-architecture |
| CQRS by Martin Fowler | martinfowler.com/bliki/CQRS.html |
| Python `dataclasses` docs | docs.python.org/3/library/dataclasses.html |

---

*Speaker: Shivam Chaurasia, Solution Architect 1, EPAM Systems*
