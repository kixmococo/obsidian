# Annotated & Practical App Modules

## 01_core_language_concepts — Core language concepts

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 1: Core Language Concepts — variables, types, operators,
# control flow, and truthiness — with a note on every line.
# =============================================================

# --- VARIABLES & TYPES ---
# Python variables aren't "declared" with a type keyword like
# `int age;` in C/Java. You just assign a value, and the name
# points at whatever object that value is. This is called
# "dynamic typing" — the TYPE lives on the value, not the name.

name = "u1"
# `name` now points at a `str` (string) object holding "u1".
# You could reassign `name = 5` later and Python wouldn't stop
# you — the variable itself has no fixed type.

age = 30
# `age` points at an `int` (whole number) object.

height = 1.8
# `height` points at a `float` (decimal number) object.

is_dev = True
# `is_dev` points at a `bool` (boolean) object. Python's booleans
# are actually `True`/`False`, capitalized — not lowercase like
# JS's `true`/`false`.

nothing = None
# `None` is Python's "no value" / null equivalent. It's its own
# type, `NoneType`, and there is only ever one `None` object.

# Even though Python is dynamically typed, it is STRONGLY typed:
# it will not silently coerce types for you the way JavaScript
# does with `"5" + 5`. Mixing incompatible types (e.g. str + int)
# raises a `TypeError` instead of guessing what you meant.

# --- CONTROL FLOW: if / elif / else ---

if age >= 18:
    # `if` checks a condition; if it's truthy, this indented
    # block runs. Indentation is not a style choice here — it's
    # how Python knows what belongs inside the `if`.
    print("adult")
elif age >= 13:
    # `elif` ("else if") only gets checked if every condition
    # above it was false. You can chain as many as you like.
    print("teen")
else:
    # `else` is the catch-all — runs only if nothing above matched.
    print("kid")

# --- CONTROL FLOW: for loop ---

for i in range(5):
    # `range(5)` produces the sequence 0, 1, 2, 3, 4 (5 numbers,
    # stopping BEFORE 5). `for i in ...` binds each value in turn
    # to `i` and runs the indented block once per value.
    print(i)

# --- CONTROL FLOW: while loop ---

n = 5
# A normal variable assignment, used as our loop counter.

while n > 0:
    # `while` keeps running its block as long as the condition
    # stays truthy. Unlike `for`, there's no automatic iteration —
    # you're responsible for eventually making the condition false.
    n -= 1
    # `n -= 1` is shorthand for `n = n - 1`. Without this line,
    # the condition `n > 0` would never become false, and the
    # loop would run forever (an infinite loop).

# --- PRINTING EVERYTHING ---

print(name, age, height, is_dev, nothing)
# `print()` accepts any number of comma-separated arguments and
# prints them space-separated on one line. This line proves all
# five variables still hold the values we assigned above.
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: Grade Calculator with limited retry attempts
#
# Real-world relevance: this is the same shape as any input-
# validation flow you've seen in a real app — a login form that
# gives you 3 tries, a quiz that scores an answer, a checkout
# form that rejects bad input and asks again. It's built entirely
# from Module 1 concepts: variables, types, if/elif/else, and a
# while loop with a counter.
# =============================================================

MAX_ATTEMPTS = 3
# A variable in ALL_CAPS is a Python convention (not a rule the
# interpreter enforces) meaning "treat this as a constant — don't
# reassign it elsewhere in the program."

attempts = 0
score = None
# We start `score` as `None` (falsy, "no value yet") rather than
# 0, because 0 is a *valid* score — using None avoids confusing
# "no input yet" with "scored zero".

while attempts < MAX_ATTEMPTS and score is None:
    # This loop keeps asking for input until either:
    #   1) the user provides a valid number, or
    #   2) they've used up all their attempts.
    # `is None` (not `== None`) is the idiomatic way to check for
    # None in Python — it compares identity, which is exactly
    # what you want for a singleton like None.

    raw = input(f"Enter a test score 0-100 (attempt {attempts + 1}/{MAX_ATTEMPTS}): ")
    # `input()` always returns a `str`, even if the user typed
    # digits. That's why real-world input handling always needs
    # a conversion + validation step, shown next.

    try:
        candidate = int(raw)
        # `int(raw)` attempts to convert the string to a whole
        # number. If `raw` isn't a valid integer (e.g. "abc"),
        # this raises a ValueError — caught below.
    except ValueError:
        print("That's not a whole number. Try again.")
        attempts += 1
        continue
        # `continue` skips the rest of this loop iteration and
        # jumps back to the `while` condition check.

    if candidate < 0 or candidate > 100:
        print("Score must be between 0 and 100.")
        attempts += 1
        continue

    score = candidate
    # Only reaches here once we have a genuinely valid score,
    # which ends the while loop (score is no longer None).

if score is None:
    # We exhausted MAX_ATTEMPTS without ever getting valid input.
    print("No valid score entered. Exiting.")
else:
    # This is the same if/elif/else pattern from the guide,
    # just applied to a real decision: mapping a number to a
    # letter grade, exactly like a school grading system would.
    if score >= 90:
        grade = "A"
    elif score >= 80:
        grade = "B"
    elif score >= 70:
        grade = "C"
    elif score >= 60:
        grade = "D"
    else:
        grade = "F"

    is_passing = grade != "F"
    # A bool derived from a comparison — shows booleans aren't
    # just literals, they're usually the RESULT of an expression.

    print(f"Score: {score} -> Grade: {grade} (passing: {is_passing})")
```

## 02_data_structures — Data structures

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 2: Data Structures — list, tuple, dict, set, and
# comprehensions — with a note on every line.
# =============================================================

# --- LIST: ordered, mutable, allows duplicates ---

scores = [10, 20, 30]
# Square brackets create a `list`. Lists remember insertion
# order and you can change them in place (add/remove/reassign).

scores.append(40)
# `.append(x)` adds `x` to the end of the list. `scores` is now
# [10, 20, 30, 40]. This mutates the list in place — it does not
# create a new list.

print(scores[0])
# Indexing starts at 0, so `scores[0]` is the FIRST element: 10.

print(scores[-1])
# Negative indices count from the end. `-1` is the LAST element:
# 40. This is a Python-specific convenience — no need to write
# `scores[len(scores) - 1]`.

print(scores[1:3])
# Slicing: `[start:stop]` returns a NEW list containing elements
# from index `start` up to (but not including) `stop`. So this
# grabs indices 1 and 2 -> [20, 30].

# --- DICT: key-value pairs, ordered (3.7+), mutable ---

player = {"name": "u1", "level": 12}
# Curly braces with `key: value` pairs create a `dict`. Keys are
# typically strings but can be any hashable type (numbers, tuples).

player["level"] += 1
# Looks up the current value at key "level" (12), adds 1, and
# stores 13 back under that same key. Equivalent to
# `player["level"] = player["level"] + 1`.

print(player.get("mana", 0))
# `.get(key, default)` looks up "mana"; since it doesn't exist in
# `player`, it returns the DEFAULT value (0) instead of raising
# a KeyError the way `player["mana"]` would.

# --- TUPLE: ordered, IMMUTABLE, allows duplicates ---

coords = (4, 5)
# Parentheses create a `tuple`. Once built, you cannot change,
# add, or remove its elements — attempting `coords[0] = 9` raises
# a TypeError. Use tuples for fixed groupings of values, like a
# coordinate pair or an (x, y) point that should never change.

# --- SET: unordered, mutable, NO duplicates ---

unique_ids = {1, 2, 2, 3}
# Curly braces WITHOUT `key: value` pairs create a `set`. Sets
# automatically drop duplicates, so `unique_ids` ends up as
# {1, 2, 3} even though we wrote 2 twice. Sets also have no
# guaranteed order and no indexing (`unique_ids[0]` is invalid).

# --- COMPREHENSIONS: build a collection in one expression ---

squares = [x**2 for x in range(10)]
# A list comprehension. Read it right-to-left conceptually:
# "for x in range(10)" (0 through 9), compute "x**2", collect
# all the results into a new list. Equivalent to writing a for
# loop that appends to an empty list, but in one line.

evens = [x for x in range(20) if x % 2 == 0]
# Same idea, with a filter: the `if` clause only keeps values of
# `x` where `x % 2 == 0` (no remainder when divided by 2, i.e.
# even numbers) from range(20).

lookup = {x: x**2 for x in range(5)}
# A DICT comprehension — same pattern, but building key: value
# pairs instead of a flat list. Produces
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}.

print(coords, unique_ids, squares, evens, lookup)
# Prints all five data structures so you can see their final
# shapes and confirm mutability/order behaved as described above.
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: Mini Shopping Cart / Inventory Summary
#
# Real-world relevance: this is the actual shape of data you'd
# get back from an e-commerce API or database query — a list of
# records (dicts), where each record has fixed fields. Every data
# structure from Module 2 earns its place here: list, dict,
# tuple, set, and comprehensions.
# =============================================================

# A LIST of DICTS — the most common "table of records" shape in
# real Python code (think: rows from a database or a JSON API).
cart = [
    {"name": "Keyboard", "category": "Peripherals", "price": 45.00, "qty": 1},
    {"name": "Mouse", "category": "Peripherals", "price": 20.00, "qty": 2},
    {"name": "Monitor", "category": "Displays", "price": 150.00, "qty": 1},
    {"name": "USB Cable", "category": "Accessories", "price": 8.50, "qty": 3},
]

# TUPLE for a fixed, never-changing pair — a currency label bundled
# with a tax rate. Using a tuple instead of two loose variables
# signals "these two values travel together and won't be edited."
CURRENCY, TAX_RATE = ("USD", 0.08)

# Dict comprehension: build a quick lookup of "item name -> line total"
# without a manual for-loop + empty dict + append pattern.
line_totals = {item["name"]: item["price"] * item["qty"] for item in cart}

# List comprehension with a filter: pull out only the items whose
# line total crosses a threshold — e.g. for a "high value items"
# shipping-insurance flag.
high_value_items = [name for name, total in line_totals.items() if total >= 40]

# Set comprehension (same syntax family): collect the UNIQUE
# categories present in the cart, since a real inventory report
# cares about categories once each, not once per item.
categories = {item["category"] for item in cart}

subtotal = sum(line_totals.values())
tax = round(subtotal * TAX_RATE, 2)
total = round(subtotal + tax, 2)

print(f"Categories in cart: {sorted(categories)}")
print(f"High-value items (>= $40 line total): {high_value_items}")
print(f"Subtotal: {subtotal:.2f} {CURRENCY}")
print(f"Tax ({TAX_RATE:.0%}): {tax:.2f} {CURRENCY}")
print(f"Total: {total:.2f} {CURRENCY}")

# .get() with a default — safe lookup for a category that might
# not exist, the way you'd defensively read an optional API field.
gift_wrap_fee = {"Peripherals": 0, "Displays": 5.00}.get("Accessories", 2.50)
print(f"Gift wrap fee for Accessories: {gift_wrap_fee}")
```

## 03_functions_and_scope — Functions and scope

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 3: Functions & Scope — def, defaults, *args/**kwargs,
# lambdas, and global/local scope — with a note on every line.
# =============================================================

def greet(name, greeting="Hello"):
    # `def` defines a function named `greet`. `name` is a
    # required parameter — you must supply it. `greeting="Hello"`
    # is a parameter WITH a default value, so it's optional; if
    # the caller doesn't pass one, "Hello" is used.
    return f"{greeting}, {name}!"
    # `return` sends a value back to wherever the function was
    # called. Without `return`, a function implicitly returns
    # `None`.

print(greet("u1"))
# Calling with one POSITIONAL argument. "u1" fills `name` because
# of its position (first slot); `greeting` falls back to its
# default "Hello". Prints "Hello, u1!".

print(greet(name="u1", greeting="Hey"))
# Calling with KEYWORD arguments — naming each parameter
# explicitly. Order doesn't matter here since Python matches by
# name, not position. Prints "Hey, u1!".


def total(*args, **kwargs):
    # `*args` collects any extra POSITIONAL arguments into a
    # `tuple`. `**kwargs` collects any extra KEYWORD arguments
    # into a `dict`. This lets a function accept an arbitrary,
    # unknown-in-advance number of inputs.
    print(args)
    print(kwargs)

total(1, 2, 3, tax=0.1, shipping=5)
# `1, 2, 3` (positional, no matching named parameter) get bundled
# into `args` as the tuple (1, 2, 3). `tax=0.1, shipping=5`
# (keyword) get bundled into `kwargs` as {"tax": 0.1, "shipping": 5}.

square = lambda x: x * x
# `lambda` creates a small, unnamed (anonymous) function inline.
# `lambda x: x * x` is equivalent to writing:
#   def square(x):
#       return x * x
# but as a single expression — handy for short throwaway
# functions, e.g. as a `key=` argument to `sorted()`.

print(square(5))
# Calls the lambda like any other function: 5 * 5 = 25.

counter = 0
# A variable defined at the TOP LEVEL of the file (module scope).
# Any function below can READ this, but modifying it requires
# an explicit `global` declaration (see below) — otherwise Python
# assumes an assignment inside a function creates a new LOCAL
# variable instead.

def increment():
    global counter
    # `global counter` tells Python: "when I assign to `counter`
    # inside this function, modify the module-level variable, not
    # a new local one." Without this line, `counter += 1` below
    # would raise an UnboundLocalError.
    counter += 1
    # Reads the current value of the global `counter`, adds 1,
    # and writes it back to that same global variable.

increment()
increment()
# Each call increments the shared global counter by 1: 0 -> 1 -> 2.

print(counter)
# Prints 2, proving the function actually mutated the outer
# variable rather than a private local copy.

# Rule of thumb from the guide: avoid relying on `global` in real
# code — prefer passing values IN as parameters and getting
# results OUT via `return`. It's demonstrated here only to show
# how scope actually works under the hood.
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: Order Total Calculator
#
# Real-world relevance: this mirrors how a checkout/billing
# function in a real app is structured — a flexible function
# signature (unknown number of line items, optional fees), pure
# functions that take input and return output (no globals), and
# a lambda used the way it actually gets used in practice: as a
# sort key.
# =============================================================

def calculate_order(*item_prices, discount_pct=0, **fees):
    # `*item_prices` lets the caller pass any number of prices
    # as loose positional args — like `calculate_order(9.99, 14.50)`
    # instead of forcing them to build a list first.
    # `discount_pct=0` is a keyword-only-by-convention default.
    # `**fees` lets the caller tack on named extra costs, e.g.
    # `shipping=4.99, gift_wrap=2.00`, without the function needing
    # to know those fee names in advance.

    subtotal = sum(item_prices)
    discount_amount = subtotal * (discount_pct / 100)
    fee_total = sum(fees.values())
    grand_total = subtotal - discount_amount + fee_total

    # Returning a dict of results ("out parameters") instead of
    # mutating a global — the caller decides what to do with it.
    return {
        "subtotal": round(subtotal, 2),
        "discount": round(discount_amount, 2),
        "fees": dict(fees),
        "total": round(grand_total, 2),
    }


order = calculate_order(9.99, 14.50, 3.25, discount_pct=10, shipping=4.99, gift_wrap=2.00)
print(order)

# --- lambda in its most common real use: a sort key ---

customers = [
    {"name": "Ada", "lifetime_spend": 420.50},
    {"name": "Grace", "lifetime_spend": 1290.00},
    {"name": "Linus", "lifetime_spend": 75.10},
]

# `sorted()` takes a `key` function that tells it what to compare
# each element BY. Writing a full `def` just to extract one field
# would be overkill, so a lambda is the idiomatic choice here.
top_spenders = sorted(customers, key=lambda c: c["lifetime_spend"], reverse=True)

for customer in top_spenders:
    print(f"{customer['name']}: ${customer['lifetime_spend']:.2f}")
```

## 04_oop — Oop

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 4: Object-Oriented Programming — classes, instances,
# inheritance, and dunder methods — with a note on every line.
# =============================================================

class Character:
    # `class` defines a new blueprint/type named `Character`.
    # Everything indented under it belongs to that blueprint.

    species = "human"
    # A CLASS attribute — defined directly on the class body, not
    # inside `__init__`. It's shared by every instance unless a
    # specific instance overrides it. Think of it as a default
    # that lives on the blueprint itself.

    def __init__(self, name, hp=100):
        # `__init__` is the CONSTRUCTOR — Python calls it
        # automatically whenever you create a new Character, e.g.
        # `Character("u1")`. `self` refers to the specific
        # instance being built; it's always the first parameter
        # of an instance method, though Python passes it for you
        # automatically — you never type it yourself when calling.
        self.name = name
        # An INSTANCE attribute — stored on THIS particular object,
        # not shared with other Characters the way `species` is.
        self.hp = hp
        # `hp` defaults to 100 if the caller doesn't specify one
        # (same default-parameter mechanic as Module 3's functions).

    def take_damage(self, amount):
        # An instance METHOD — a function that lives on the class
        # and always receives `self` (the instance it was called
        # on) as its first argument automatically.
        self.hp -= amount
        # Mutates THIS instance's `hp`, not a global or class-level
        # value — each Character keeps its own independent hp.
        return self.hp
        # Returns the updated hp so the caller can use it directly.

    def __repr__(self):
        # A DUNDER ("double underscore") method — `__repr__`
        # controls what `print(some_character)` or typing the
        # object in a REPL shows. Without it, printing an object
        # gives an unhelpful default like `<Character object at
        # 0x...>`. Defining `__repr__` plugs your class into
        # Python's built-in printing behavior.
        return f"Character({self.name}, hp={self.hp})"


class Mage(Character):
    # `class Mage(Character):` means Mage INHERITS from Character
    # — it automatically gets everything Character has (species,
    # __init__ behavior, take_damage, __repr__) unless Mage
    # overrides it, which it does for `__init__` below.

    def __init__(self, name, mana=50):
        # Mage's own constructor — it needs an extra `mana` field
        # that plain Characters don't have.
        super().__init__(name)
        # `super()` refers to the PARENT class (Character). Calling
        # `super().__init__(name)` runs Character's constructor
        # first, which sets up `self.name` and `self.hp = 100`
        # (the default). This avoids duplicating that setup logic.
        self.mana = mana
        # Then Mage adds its own extra instance attribute on top.

    def cast(self, spell):
        # A method that only exists on Mage, not on plain
        # Character — this is one way subclasses extend behavior.
        self.mana -= 10
        return f"{self.name} casts {spell}"


hero = Character("u1")
# Creates a new Character instance. Python calls
# `Character.__init__(hero, "u1")` behind the scenes, so
# hero.name = "u1" and hero.hp = 100 (the default).

hero.take_damage(30)
# Calls the method on `hero`; Python passes `hero` in automatically
# as `self`. hero.hp becomes 100 - 30 = 70.

print(hero)
# Triggers `__repr__` under the hood, printing:
# Character(u1, hp=70)

wiz = Mage("Zed")
# Creates a Mage. Its __init__ calls super().__init__("Zed") first
# (giving it hp=100 via Character's default), then sets mana=50.

print(wiz.cast("Fireball"))
# Calls Mage's own `cast` method — not something Character has.
# Prints "Zed casts Fireball" and mana drops to 40.

print(wiz)
# Mage didn't define its own `__repr__`, so it INHERITS
# Character's — this is POLYMORPHISM/inheritance in action:
# Character(Zed, hp=100). Note mana isn't shown, since __repr__
# only knows about name/hp.
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: Bank Account System
#
# Real-world relevance: BankAccount/SavingsAccount is a classic
# example because it maps 1:1 onto real banking software:
# encapsulated balance (you never touch `account.balance`
# directly from outside, you go through methods that enforce
# rules), inheritance for account types that share behavior but
# differ in one rule (interest), and dunder methods so accounts
# behave naturally with Python's built-ins (print, ==, sorting).
# =============================================================

class InsufficientFundsError(Exception):
    # A custom exception (previewed here, covered fully in
    # Module 5) — lets calling code catch THIS specific problem
    # instead of a generic Exception.
    pass


class BankAccount:
    def __init__(self, owner, balance=0.0):
        self.owner = owner
        self._balance = balance
        # The leading underscore `_balance` is a Python CONVENTION
        # (not enforced by the language) signaling "treat this as
        # internal — use the methods below instead of touching it
        # directly." This is encapsulation: the class controls how
        # its data can change.

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self._balance += amount
        return self._balance

    def withdraw(self, amount):
        if amount > self._balance:
            raise InsufficientFundsError(
                f"{self.owner} has {self._balance:.2f}, tried to withdraw {amount:.2f}"
            )
        self._balance -= amount
        return self._balance

    @property
    def balance(self):
        # A `property` exposes `_balance` as READ-ONLY from
        # outside — `account.balance` works like a plain attribute
        # but you can't do `account.balance = 999` to cheat it;
        # you're forced through deposit()/withdraw().
        return self._balance

    def __repr__(self):
        return f"{self.__class__.__name__}({self.owner}, balance={self._balance:.2f})"

    def __eq__(self, other):
        # Defining `__eq__` lets `account_a == account_b` compare
        # by VALUE (same owner and balance) instead of Python's
        # default of comparing by identity (same object in memory).
        if not isinstance(other, BankAccount):
            return NotImplemented
        return self.owner == other.owner and self._balance == other._balance


class SavingsAccount(BankAccount):
    def __init__(self, owner, balance=0.0, interest_rate=0.02):
        super().__init__(owner, balance)
        self.interest_rate = interest_rate

    def apply_interest(self):
        # Extra behavior only savings accounts have — checking
        # accounts (if we added one) wouldn't inherit this unless
        # they subclassed SavingsAccount too.
        interest = self._balance * self.interest_rate
        self._balance += interest
        return round(interest, 2)


acct = SavingsAccount("u1", balance=1000.0, interest_rate=0.03)
acct.deposit(250)
print(f"Interest earned: {acct.apply_interest()}")
print(acct)

try:
    acct.withdraw(5000)
except InsufficientFundsError as e:
    print(f"Withdrawal blocked: {e}")

other = SavingsAccount("u1", balance=acct.balance)
print(f"Accounts equal? {acct == other}")
```

## 05_error_handling — Error handling

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 5: Error Handling — try/except/else/finally and custom
# exceptions — with a note on every line.
# =============================================================

raw_value = "abc"
# A deliberately "bad" string — standing in for something like
# real user input that turns out not to be a valid number.

try:
    # `try` marks a block where you expect something MIGHT fail.
    # Python runs it line by line; the moment something raises an
    # exception, it jumps straight to a matching `except` below.
    value = int(raw_value)
    # `int("abc")` cannot convert "abc" to a number, so this
    # raises a `ValueError` and immediately abandons the rest of
    # the `try` block.
except ValueError:
    # Catches specifically a `ValueError` — exactly what `int()`
    # raises on bad input. Catching the SPECIFIC exception type
    # (rather than a bare `except:`) means you only swallow the
    # error you actually anticipated, not unrelated bugs.
    print("That wasn't a number.")
except (TypeError, KeyError) as e:
    # A second `except` clause, checked only if the first didn't
    # match. `(TypeError, KeyError)` catches EITHER of those two
    # types in one clause. `as e` binds the actual exception
    # object to `e` so you can inspect/print its message.
    print(f"Something else went wrong: {e}")
else:
    # `else` runs ONLY if the `try` block completed with NO
    # exception at all. It won't run here, since `int("abc")`
    # failed above and jumped to the `except ValueError` branch.
    print("No errors — this runs if try succeeded")
finally:
    # `finally` ALWAYS runs — whether the try succeeded, an
    # except caught something, or even if an exception went
    # uncaught and is about to propagate. This is where you put
    # cleanup code (closing a file, releasing a lock) that must
    # happen no matter what.
    print("Always runs — cleanup goes here")


class InsufficientManaError(Exception):
    # Defining your OWN exception type: a class that inherits
    # from the built-in `Exception`. `pass` means "no extra
    # behavior needed — just being a distinct, named type is the
    # whole point." Now code elsewhere can catch specifically
    # `InsufficientManaError` instead of a generic Exception.
    pass

mana = 10
cost = 30
# Two plain variables setting up a scenario where the player
# doesn't have enough mana to cast a spell.

try:
    if mana < cost:
        raise InsufficientManaError("Not enough mana to cast that")
        # `raise` manually triggers an exception. Here it fires
        # because 10 < 30. The string passed in becomes the
        # exception's message, retrievable via `str(e)` or an
        # f-string like below.
except InsufficientManaError as e:
    # Catching the custom exception we just defined and raised,
    # exactly like catching any built-in exception type.
    print(f"Custom error caught: {e}")
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: Order Processing with Custom Exceptions
#
# Real-world relevance: this is the actual error-handling shape
# of a checkout/order pipeline — validate input, check business
# rules (stock levels), handle multiple distinct failure types
# differently, and always log the attempt regardless of outcome
# (the same way a real system logs to a file or monitoring tool).
# =============================================================

inventory = {
    "widget": 5,
    "gadget": 0,
    "gizmo": 12,
}


class OutOfStockError(Exception):
    # A custom exception for a specific business rule violation —
    # "the item exists, but we don't have enough of it" — distinct
    # from a plain KeyError ("the item doesn't exist at all").
    pass


def place_order(item_name, quantity):
    if item_name not in inventory:
        raise KeyError(f"No such item: {item_name}")

    if inventory[item_name] < quantity:
        raise OutOfStockError(
            f"Only {inventory[item_name]} '{item_name}' left, requested {quantity}"
        )

    inventory[item_name] -= quantity
    return f"Order placed: {quantity}x {item_name}"


def process_order(item_name, quantity_raw):
    # Wraps place_order with the full error-handling toolkit —
    # this is the pattern a real order API endpoint follows.
    try:
        quantity = int(quantity_raw)
        # A bad quantity (e.g. "two" instead of 2) fails here
        # with a ValueError before we even touch inventory.

        if quantity <= 0:
            raise ValueError("Quantity must be positive")

        result = place_order(item_name, quantity)

    except ValueError as e:
        print(f"[REJECTED] Invalid quantity for '{item_name}': {e}")
    except KeyError as e:
        print(f"[REJECTED] {e}")
    except OutOfStockError as e:
        print(f"[REJECTED] {e}")
    else:
        # Only runs if place_order succeeded with no exception —
        # this is where you'd trigger a confirmation email, etc.
        print(f"[SUCCESS] {result}")
    finally:
        # Runs no matter what happened above — stands in for
        # something like writing an audit-log line for every
        # order attempt, successful or not.
        print(f"           (logged attempt: item={item_name}, qty={quantity_raw})")


process_order("widget", "3")      # succeeds
process_order("gadget", "1")      # out of stock
process_order("unicorn", "1")     # unknown item
process_order("gizmo", "two")     # bad quantity
process_order("gizmo", "-5")      # invalid quantity

print(f"Final inventory: {inventory}")
```

## 06_files_and_io — Files and io

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 6: Files, I/O, and Working with Data — open/with, json,
# and csv — with a note on every line.
# =============================================================

import json
# Standard library module for reading/writing JSON — the most
# common structured text format for config files and web APIs.

import csv
# Standard library module for reading/writing CSV (comma-
# separated values) — the common export format for spreadsheets.

from pathlib import Path
# `pathlib` is the modern way to build filesystem paths, instead
# of gluing strings together with `+` or `os.path.join`.

folder = Path(__file__).parent
# `__file__` is a special variable Python sets to the path of the
# CURRENTLY RUNNING script. `.parent` gives the directory that
# contains it. We use this so the demo files always land next to
# this script, no matter which directory you ran `python3` from.

with open(folder / "save.txt", "w") as f:
    # `open(path, "w")` opens a file for WRITING (creating it if
    # it doesn't exist, ERASING it if it does). `with ... as f`
    # is a CONTEXT MANAGER: it guarantees the file gets closed
    # automatically when the indented block ends — even if an
    # error happens partway through. `folder / "save.txt"` uses
    # pathlib's `/` operator to join a directory and filename.
    f.write("level=12\n")
    # `.write()` puts raw text into the file. Unlike `print()`, it
    # does NOT add a newline automatically — hence the explicit
    # `\n` here.

with open(folder / "save.txt") as f:
    # Opening with no mode argument defaults to "r" (read).
    for line in f:
        # Iterating directly over an open file yields it one LINE
        # at a time — memory-efficient even for huge files, since
        # it doesn't load the whole thing at once.
        print(line.strip())
        # `.strip()` removes the trailing "\n" (and any leading/
        # trailing whitespace) so printing doesn't add a blank line.

data = {"name": "u1", "level": 12}
# A plain Python dict — the in-memory shape we want to persist.

with open(folder / "save.json", "w") as f:
    json.dump(data, f)
    # `json.dump(obj, file)` serializes `data` into JSON text and
    # writes it directly into the open file handle `f`.

with open(folder / "save.json") as f:
    loaded = json.load(f)
    # `json.load(file)` does the reverse: reads JSON text from the
    # file and parses it back into a Python dict.
print(loaded)
# Prints {'name': 'u1', 'level': 12} — proving the round trip
# (dict -> file -> dict) preserved the data exactly.

with open(folder / "scores.csv", "w", newline="") as f:
    # `newline=""` is required on Python's csv module when
    # writing, to stop it from adding extra blank lines on
    # Windows-style line endings.
    writer = csv.DictWriter(f, fieldnames=["name", "score"])
    # `DictWriter` writes rows FROM dicts, matching dict keys to
    # the column names you specify in `fieldnames`.
    writer.writeheader()
    # Writes the first row: "name,score" — the column headers.
    writer.writerow({"name": "u1", "score": 95})
    # Writes one data row using the dict's values in fieldname order.

with open(folder / "scores.csv") as f:
    reader = csv.DictReader(f)
    # `DictReader` reads each row back as a dict, using the FIRST
    # row of the file as the keys automatically.
    for row in reader:
        print(row["name"], row["score"])
        # Each `row` is a dict like {"name": "u1", "score": "95"}.
        # Note: CSV values always come back as STRINGS — if you
        # need "95" as an int, you must convert it yourself.
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: JSON-Backed Contact Book
#
# Real-world relevance: this is project #3 from the guide's
# "Practical Projects" list, and it's the same pattern behind
# any small app that needs to "remember" data between runs
# without a full database — settings files, save games, local
# caches. The JSON file IS the persistence layer.
# =============================================================

import json
from pathlib import Path

CONTACTS_FILE = Path(__file__).parent / "contacts.json"


def load_contacts():
    # Guard against the very first run, when the file doesn't
    # exist yet — a real app can't assume its data file is there.
    if not CONTACTS_FILE.exists():
        return []
    with open(CONTACTS_FILE) as f:
        return json.load(f)


def save_contacts(contacts):
    with open(CONTACTS_FILE, "w") as f:
        # `indent=2` produces human-readable, pretty-printed JSON —
        # worth it for any file a person might open and inspect.
        json.dump(contacts, f, indent=2)


def add_contact(contacts, name, phone, email):
    for contact in contacts:
        if contact["name"].lower() == name.lower():
            print(f"'{name}' already exists — skipping duplicate.")
            return contacts
    contacts.append({"name": name, "phone": phone, "email": email})
    return contacts


def find_contact(contacts, name):
    # A generator expression + next(): scans for the first match
    # and returns None if nothing is found, instead of crashing.
    return next((c for c in contacts if c["name"].lower() == name.lower()), None)


contacts = load_contacts()

contacts = add_contact(contacts, "Ada Lovelace", "555-0100", "ada@example.com")
contacts = add_contact(contacts, "Grace Hopper", "555-0101", "grace@example.com")
contacts = add_contact(contacts, "Ada Lovelace", "555-9999", "duplicate@example.com")

save_contacts(contacts)
print(f"Saved {len(contacts)} contact(s) to {CONTACTS_FILE.name}")

found = find_contact(contacts, "grace hopper")
if found:
    print(f"Found: {found['name']} — {found['phone']} — {found['email']}")

missing = find_contact(contacts, "Nobody")
print(f"Search for 'Nobody': {missing}")

# Reload straight from disk to prove the data actually persisted,
# not just living in memory for this run.
reloaded = load_contacts()
print(f"Reloaded from disk: {[c['name'] for c in reloaded]}")
```

## 07_modules_and_packages — Modules and packages

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 7: Modules, Packages & the Standard Library — import
# styles and a taste of the stdlib — with a note on every line.
# =============================================================
#
# This file works together with `mymodule.py` sitting right next
# to it in this same folder — open that file too, it's the
# "reusable code" being imported here.

import mymodule
# `import mymodule` loads the WHOLE module (mymodule.py) and
# binds the name `mymodule` to it. To use anything inside it, you
# prefix with the module name: `mymodule.helper()`. Python finds
# `mymodule.py` because it sits in the same directory as this
# script — Python automatically adds a script's own folder to its
# search path.

from mymodule import helper
# `from mymodule import helper` pulls just the ONE function
# `helper` directly into this file's namespace, so you can call
# it bare as `helper()` instead of `mymodule.helper()`. Both
# import styles reference the exact same function underneath.

import random
# A STANDARD LIBRARY module — ships with Python, no `pip install`
# needed. Useful for randomness: dice rolls, shuffles, sampling.

from datetime import datetime
# Pulling just the `datetime` CLASS out of the `datetime` MODULE
# (confusingly, the module and the class share a name). This lets
# you write `datetime.now()` instead of `datetime.datetime.now()`.

print(mymodule.helper())
# Calls `helper()` via the full module-qualified path. Prints
# "I'm reusable".

print(helper())
# Calls the SAME function, but via the name we imported directly.
# Also prints "I'm reusable" — proving both import styles reach
# the identical underlying function.

print(mymodule.shout("hello"))
# Calls a second function from our own module — modules can hold
# as many functions (or classes, or constants) as you like.
# Prints "HELLO!".

print(random.randint(1, 6))
# `random.randint(a, b)` returns a random INTEGER between `a` and
# `b`, INCLUSIVE of both ends — simulating a single six-sided die
# roll.

print(datetime.now().year)
# `datetime.now()` returns the current date and time; `.year`
# pulls just the year number off of it — one line to answer
# "what year is it right now" from the standard library, no
# third-party package required.
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: Local Folder Report
#
# Real-world relevance: this is the kind of small utility script
# you'd actually reach for day-to-day — scan a folder, summarize
# what's in it. It combines OUR OWN module (mymodule.py, sitting
# right next to this script) with several standard-library
# modules the guide calls out as "worth knowing early."
# =============================================================

import os
from pathlib import Path
from datetime import datetime
from collections import Counter

import mymodule
# Reusing our own module's `shout()` just to format the report
# title — showing that project code you write is imported the
# exact same way as the standard library.

target_dir = Path(__file__).parent
# `os` and `pathlib` overlap in purpose; pathlib is the modern,
# object-oriented way to work with filesystem paths.

print(mymodule.shout(f"folder report: {target_dir.name}"))
print(f"Generated: {datetime.now():%Y-%m-%d %H:%M}")
print(f"Full path: {os.path.abspath(target_dir)}")

entries = list(target_dir.iterdir())
# `.iterdir()` lists everything directly inside the folder (files
# and subfolders), similar to `os.listdir()` but returning `Path`
# objects instead of plain strings.

file_count = sum(1 for e in entries if e.is_file())
dir_count = sum(1 for e in entries if e.is_dir())

extension_counts = Counter(e.suffix for e in entries if e.is_file())
# `Counter` (from `collections`) tallies how many times each
# value appears — here, how many files share each extension
# (".py", ".json", etc). Building this by hand would mean a
# manual dict + "if key in dict else" bookkeeping loop.

print(f"\nFiles: {file_count} | Subfolders: {dir_count}")
print("Extensions breakdown:")
for ext, count in extension_counts.most_common():
    # `.most_common()` returns items sorted by count, highest
    # first — exactly the ordering you want in a summary report.
    label = ext if ext else "(no extension)"
    print(f"  {label}: {count}")

total_size = sum(e.stat().st_size for e in entries if e.is_file())
print(f"\nTotal size of top-level files: {total_size} bytes")
```

## 08_intermediate_concepts — Intermediate concepts

### Annotated (`2_annotated.py`)

```python
# =============================================================
# MODULE 8: Intermediate Concepts — generators, decorators, type
# hints, f-strings, and unpacking — with a note on every line.
# =============================================================

def countdown(n):
    # A function containing `yield` is a GENERATOR function —
    # calling `countdown(5)` does NOT run the body immediately.
    # It returns a generator OBJECT that runs the body lazily,
    # one step at a time, only as values are requested.
    while n > 0:
        yield n
        # `yield` pauses the function and hands `n` out to
        # whoever is iterating, remembering exactly where it
        # stopped. Next time a value is requested, execution
        # resumes right after this line.
        n -= 1
        # This only runs AFTER the paused function is resumed —
        # so the countdown truly happens one step at a time,
        # rather than building [5, 4, 3, 2, 1] all at once in memory.

for i in countdown(5):
    # `for` repeatedly asks the generator for "the next value"
    # until it's exhausted (n reaches 0 and the while loop ends).
    print(i)
    # Prints 5, 4, 3, 2, 1 — one per iteration, computed on demand.


def log_call(func):
    # A DECORATOR is just a regular function that takes another
    # function (`func`) and returns a REPLACEMENT function.
    def wrapper(*args, **kwargs):
        # `wrapper` accepts anything — it doesn't need to know
        # `func`'s exact signature because `*args`/`**kwargs`
        # forward whatever was passed in.
        print(f"Calling {func.__name__}")
        # `func.__name__` reads the original function's name as
        # a string — useful for logging without hardcoding it.
        return func(*args, **kwargs)
        # Actually calls the ORIGINAL function with the original
        # arguments, and passes its return value straight through.
    return wrapper
    # `log_call` returns `wrapper` — NOT the result of calling it.

@log_call
def attack(target):
    # `@log_call` above a function definition is shorthand for:
    #   attack = log_call(attack)
    # So after this, the name `attack` actually points at
    # `wrapper` (from log_call), which wraps the original logic.
    print(f"Attacking {target}")

attack("goblin")
# Because of the decorator, this call actually runs `wrapper`,
# which first prints "Calling attack", THEN runs the real
# function body, printing "Attacking goblin".


def add(a: int, b: int) -> int:
    # TYPE HINTS: `a: int, b: int` documents the expected
    # parameter types; `-> int` documents the return type. Python
    # does NOT enforce these at runtime — you could still pass a
    # string and nothing would stop you — but editors, linters,
    # and tools like `mypy` use them to catch mistakes before you
    # ever run the code.
    return a + b

name, hp = "u1", 80
# Assigning two variables from one line — tuple unpacking.
# The right side `"u1", 80` is actually a tuple; Python unpacks
# its two values into `name` and `hp` in order.

print(f"{name} has {hp} HP ({hp/100:.0%})")
# An F-STRING: the `f` prefix lets you embed expressions directly
# inside `{ }` within the string. `{hp/100:.0%}` does two things
# at once: computes hp/100, THEN formats it as a percentage with
# 0 decimal places (`.0%`) — turning 0.8 into "80%".

first, *rest = [1, 2, 3, 4]
# STAR UNPACKING: `first` grabs the first element (1). `*rest`
# (the star means "collect everything else") gathers ALL
# remaining elements into a list: [2, 3, 4].

print(first, rest)
# Prints: 1 [2, 3, 4]

def merge(**kwargs):
    # `**kwargs` here collects any keyword arguments into a dict
    # (same mechanic as Module 3), and this function just hands
    # that dict straight back.
    return kwargs

print(merge(**{"a": 1}, **{"b": 2}))
# `**{"a": 1}` UNPACKS a dict into individual keyword arguments —
# equivalent to calling `merge(a=1, b=2)`. Doing it twice with two
# different dicts merges both into one call. Prints {'a': 1, 'b': 2}.

print(add(2, 3))
# Calls our type-hinted function normally — hints don't change
# how the call works, only what tools can check ahead of time.
# Prints 5.
```

### Practical Application (`3_practical_app.py`)

```python
# =============================================================
# PRACTICAL APP: Damage-Over-Time Combat Resolver
#
# Real-world relevance: this is the kind of small combat-math
# module you'd write in an actual game (directly relevant given
# a Godot/JS background) — but the same shape (lazy step-by-step
# processing + logging wrapper + typed helper functions) shows up
# in non-game code too: billing installments, retry backoff
# delays, animation frames. It combines every concept from this
# module: generators, decorators, type hints, f-strings, and
# unpacking. See `test_practical_app.py` next to this file for
# how you'd actually TEST this with pytest, per the guide's
# "Testing" section.
# =============================================================

from time import perf_counter


def apply_multiplier(damage: int, multiplier: float) -> int:
    # Type-hinted helper — a small, easily testable pure function.
    return round(damage * multiplier)


def damage_ticks(total_damage: int, num_ticks: int):
    # A GENERATOR: instead of computing a full list of per-tick
    # damage up front, it yields one tick's worth at a time — the
    # same lazy-evaluation shape as a real damage-over-time effect
    # applying poison/burn damage once per game frame or second.
    base = total_damage // num_ticks
    remainder = total_damage % num_ticks
    for tick in range(num_ticks):
        # Push the leftover remainder onto the final tick so the
        # ticks sum to EXACTLY total_damage, not less.
        yield base + remainder if tick == num_ticks - 1 else base


def timed(func):
    # A DECORATOR that measures and logs how long a call took —
    # a genuinely common real-world use (profiling slow code
    # paths) beyond the toy logging example in the annotated file.
    def wrapper(*args, **kwargs):
        start = perf_counter()
        result = func(*args, **kwargs)
        elapsed_ms = (perf_counter() - start) * 1000
        print(f"{func.__name__} took {elapsed_ms:.3f}ms")
        return result
    return wrapper


@timed
def resolve_attack(base_damage: int, multiplier: float, num_ticks: int) -> list[int]:
    boosted = apply_multiplier(base_damage, multiplier)
    return list(damage_ticks(boosted, num_ticks))


def combine_hit_logs(*logs: list[int]) -> list[int]:
    # STAR-UNPACKING each incoming log into one flat list, instead
    # of nested loops — `*logs` already collects every positional
    # argument into a tuple of lists; this unpacks each of THOSE.
    combined: list[int] = []
    for log in logs:
        combined = [*combined, *log]
    return combined


if __name__ == "__main__":
    # Guard so importing this file (e.g. from the test file) does
    # NOT re-run this demo — only running it directly does.
    hits = resolve_attack(base_damage=100, multiplier=1.5, num_ticks=4)
    print(f"Damage per tick: {hits}")

    crit_hits = resolve_attack(base_damage=40, multiplier=2.0, num_ticks=2)
    print(f"Crit damage per tick: {crit_hits}")

    total_log = combine_hit_logs(hits, crit_hits)
    print(f"Combined hit log: {total_log}")
    print(f"Total damage dealt: {sum(total_log)}")
```

