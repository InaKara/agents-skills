---
keywords: [dependency, injection, depends, fastapi, testing, inversion-of-control, constructor, provider, mocking, python]
---

# `Dependency Injection`

> A technique where an object receives the things it needs to do its job from the outside, rather than creating them itself.

## The Problem It Solves

Imagine a class that sends emails. If it creates its own email client internally, you can never swap that client for a test fake or a different provider without editing the class itself. Every piece of code that *uses* a dependency is also *responsible* for wiring it up — so changes ripple everywhere. Dependency Injection (DI) moves that wiring responsibility out of the class and into the caller, keeping each piece of code focused on one job.

## The Mental Model

Think of a restaurant kitchen. The chef doesn't go out and buy ingredients — a supplier delivers them. The chef just cooks with whatever arrives. If the restaurant switches meat suppliers, the chef's technique doesn't change at all.

Your class is the chef. Its dependencies (a database connection, an email client, a logger) are the ingredients. DI is the supplier: it delivers what the class needs, so the class never has to worry about *where* those things come from.

## How It Works

Instead of constructing dependencies inside a class, you accept them as parameters — most commonly through the constructor.

```python
# Without DI — the class controls its own dependency
class OrderService:
    def __init__(self):
        self.mailer = SmtpMailer(host="smtp.example.com")  # hard-wired

    def confirm(self, order):
        self.mailer.send(order.email, "Confirmed!")


# With DI — the dependency is handed in from outside
class OrderService:
    def __init__(self, mailer):          # accepts any mailer
        self.mailer = mailer

    def confirm(self, order):
        self.mailer.send(order.email, "Confirmed!")


# Production usage
service = OrderService(mailer=SmtpMailer(host="smtp.example.com"))

# Test usage — swap in a fake, no code changes needed
service = OrderService(mailer=FakeMailer())
```

That's the whole idea. The class declares what it needs; something else provides it.

## Comparison to Alternatives

Code that constructs its own dependencies works fine in tiny scripts, but becomes painful as soon as you need testing, configuration flexibility, or multiple environments.

| Aspect | Dependency Injection | Without it (self-constructed) |
|---|---|---|
| Testing | Swap real deps for fakes trivially | Must patch internals or use live services |
| Flexibility | Change behaviour by passing a different object | Must edit the class itself |
| Clarity | Dependencies are explicit in the signature | Hidden inside the implementation |
| Complexity | Slight upfront wiring cost | Simpler initially, harder later |

## Common Misconceptions

- **"DI requires a framework."** Wrong. Passing an argument to a constructor *is* dependency injection. Frameworks (like Python's `dependency-injector`) automate the wiring, but they are optional.

- **"DI means my class doesn't know what it's working with."** Wrong. The class still knows the *interface* it needs (e.g., "something with a `.send()` method"). It just doesn't know — or care — which concrete object fulfils it.

- **"DI is only for large enterprise projects."** Wrong. Any time you write a unit test and want to avoid hitting a real database or API, DI is the tool that makes substitution clean and easy.

## `Depends()` in FastAPI

FastAPI has first-class support for DI through its `Depends()` function. It lets you declare that a route handler needs some resource — a database session, the current logged-in user, a validated config — and FastAPI resolves and injects it automatically before calling your function.

**Purpose:** Move shared setup logic (auth checks, DB connections, input validation) out of individual route handlers and into reusable provider functions.

**How it works:**

```python
from fastapi import Depends, FastAPI
from sqlalchemy.orm import Session
from database import SessionLocal

app = FastAPI()


# --- Dependency provider ---
def get_db():
    """Opens a DB session, yields it, then closes it when the request ends."""
    db = SessionLocal()
    try:
        yield db          # FastAPI injects this value into the route
    finally:
        db.close()


# --- Route that uses the dependency ---
@app.get("/orders/{order_id}")
def read_order(order_id: int, db: Session = Depends(get_db)):
    # `db` is already open and ready — no setup code here
    return db.query(Order).filter(Order.id == order_id).first()
```

**Key behaviours of `Depends()`:**

| Behaviour | Detail |
|---|---|
| Automatic resolution | FastAPI calls the provider function before the route handler runs |
| Lifetime scoping | Use `yield`-based providers for resources that need cleanup (DB sessions, file handles) |
| Composability | A dependency can itself declare `Depends()`, forming a chain |
| Override in tests | Use `app.dependency_overrides` to swap any dependency for a fake without touching route code |

**Overriding in tests:**

```python
def fake_db():
    return FakeSession()

app.dependency_overrides[get_db] = fake_db   # applied for the duration of the test
```

This is the same DI principle from the earlier sections — FastAPI just automates the wiring so you never call `get_db()` yourself.
