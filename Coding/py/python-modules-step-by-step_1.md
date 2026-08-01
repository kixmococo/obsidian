# Python in Modules: Beginner → Novice, Step by Step
*Work top to bottom. Each module builds on the last — don't skip. Each ends with a checkpoint; if you can't do the checkpoint without looking back, redo the module before moving on.*

---

## How to use this
- Type every code example yourself. Don't copy-paste — typing it is what builds the muscle memory.
- Do the practice exercise before reading the next module.
- Keep a `sandbox.py` file open the whole time and just run things in it as you go.
- Setup/install steps are in the earlier guide — this file assumes `python3` already runs on your machine.

---

## Module 0 — Orientation
**Goal:** know what you're actually doing when you "run Python."

- A `.py` file is a plain text file. Python **reads it top to bottom** and executes each line as it goes — no separate compile step you have to think about (it compiles to bytecode internally, but that's invisible to you day-to-day).
- Two ways to run code: the **REPL** (`python3` in a terminal, one line at a time, good for testing) and a **script** (`python3 myfile.py`, runs the whole file).
- `print()` is your window into what's happening. Use it constantly while learning — don't guess what a value is, print it.

**Checkpoint:** create `sandbox.py` containing `print("it runs")`, and run it from the terminal.

---

## Module 1 — Variables & Types
**Concepts:** variables, dynamic typing, the core built-in types.

```python
name = "u1"        # str
age = 30             # int
height = 1.8          # float
is_dev = True          # bool
nothing = None          # NoneType — "no value"

print(type(name))    # <class 'str'>
```
A variable is just a label pointing at a value. `=` doesn't mean "equals" like in math — it means "assign." Python figures out the type from the value; you never declare it.

**Practice:** make variables for a game character's `name`, `level` (int), `hp_percent` (float), `is_alive` (bool). Print each one with `type()`.

**Checkpoint:** what happens if you run `print(1 + "1")`? Try it, then explain in one sentence why it fails.

---

## Module 2 — Operators & Expressions
**Concepts:** arithmetic, comparison, logical, assignment operators.

```python
7 + 3, 7 - 3, 7 * 3, 7 / 3      # / always gives a float
7 // 3                           # floor division: 2
7 % 3                            # modulo (remainder): 1
2 ** 5                           # exponent: 32

5 == 5, 5 != 3, 5 > 3, 5 <= 5     # comparisons -> bool

True and False, True or False, not True   # logical operators

x = 5
x += 1     # same as x = x + 1
x *= 2
```
Every expression **evaluates to a value**. `5 > 3` isn't a statement, it's a value (`True`) you can store or print.

**Practice:** given `hp = 73`, write one expression that prints `True` if hp is above 50% of a max of 100, `False` otherwise.

**Checkpoint:** what's the difference between `/` and `//`? What does `10 % 3` give you and why?

---

## Module 3 — Control Flow (if / elif / else)
**Concepts:** conditionals, boolean logic, truthiness, indentation-as-syntax.

```python
hp = 40

if hp <= 0:
    print("dead")
elif hp < 50:
    print("low health")
else:
    print("healthy")
```
Indentation isn't style here — it **is** the block boundary. No braces. Everything indented under an `if` belongs to it.

**Truthy/falsy:** `0`, `""`, `[]`, `{}`, `None`, `False` are all falsy. Everything else is truthy.
```python
name = ""
if name:
    print("has a name")
else:
    print("empty")   # this runs
```

**Practice:** write a program that takes a `score` variable and prints a letter grade (A/B/C/D/F) using if/elif/else.

**Checkpoint:** why does `if []:` not run its block? What are the other falsy values?

---

## Module 4 — Loops
**Concepts:** `for`, `while`, `range`, `break`, `continue`.

```python
for i in range(5):        # 0,1,2,3,4
    print(i)

for i in range(2, 10, 2):  # start, stop, step -> 2,4,6,8
    print(i)

n = 3
while n > 0:
    print(n)
    n -= 1

for i in range(20):
    if i == 5:
        break             # stop the loop entirely
    if i % 2 == 0:
        continue          # skip to next iteration
    print(i)
```
`for` loops over something iterable (a range, a list, a string...). `while` loops as long as a condition holds — use it when you don't know the number of iterations ahead of time.

**Practice:** print all multiples of 3 between 1 and 50 using a `for` loop. Then write a `while` loop that keeps halving a number starting at 100 until it's less than 1, printing each step.

**Checkpoint:** when would you reach for `while` instead of `for`? Give one real example.

---

## Module 5 — Strings in Depth
**Concepts:** indexing, slicing, methods, f-strings.

```python
s = "Hello, u1"
s[0]           # 'H'
s[-1]          # '1' (negative index = from the end)
s[7:9]          # 'u1' (slice: start inclusive, end exclusive)
s.lower()
s.upper()
s.strip()        # remove leading/trailing whitespace
s.split(", ")     # ['Hello', 'u1']
s.replace("u1", "u2")

name, level = "u1", 12
print(f"{name} is level {level}")           # f-string — use this over % or .format()
print(f"{level:03d}")                        # 012 — zero-padded
print(f"{level/100:.1%}")                     # 12.0%
```
Strings are **immutable** — every method above returns a *new* string, it never changes `s` in place.

**Practice:** given `sentence = "the quick brown fox"`, capitalize each word and join them back with `"-"` instead of spaces. (Hint: `.split()`, a loop or comprehension, `.capitalize()`, `"-".join()`.)

**Checkpoint:** why does `s.upper()` not change `s` itself? What would you need to write to actually update `s`?

---

## Module 6 — Lists & Tuples
**Concepts:** ordered collections, mutability vs immutability.

```python
scores = [10, 20, 30]
scores.append(40)
scores.insert(0, 5)
scores.remove(20)
scores.pop()          # removes & returns last item
scores.sort()
scores.reverse()
len(scores)
30 in scores            # membership test -> bool

scores[1:3]              # slice -> new list
scores[::-1]              # reversed copy

point = (3, 4)             # tuple — immutable, use for fixed groupings
x, y = point                # unpacking
```
**List** = mutable, ordered, use for collections that change. **Tuple** = immutable, ordered, use for fixed structures (coordinates, RGB values, a function returning multiple values).

**Practice:** build a list of 5 enemy names. Add one, remove one, sort it, and print the third element using indexing.

**Checkpoint:** why would you choose a tuple over a list for a function that returns `(x, y)` coordinates?

---

## Module 7 — Dictionaries & Sets
**Concepts:** key-value storage, uniqueness, lookups.

```python
player = {"name": "u1", "level": 12, "hp": 80}
player["level"]                # 12
player["mana"] = 50             # add a key
player.get("stamina", 0)         # safe lookup, default 0 if missing
"hp" in player                    # membership check on keys
del player["mana"]

for key, value in player.items():
    print(key, value)

seen = {1, 2, 3}       # set — unordered, unique items only
seen.add(2)             # no effect, already there
seen.add(4)
3 in seen                # fast membership check
```
Dicts are the workhorse structure for anything key-labeled (a player's stats, a config, a JSON payload). Sets are for "do I have this already / give me only unique items" problems.

**Practice:** build a dict representing an inventory (`item name -> quantity`). Add an item, increase a quantity, and print every item with its count using `.items()`.

**Checkpoint:** what's the difference in what you can look up quickly in a `list` vs a `dict`? (Think about how you'd find "does X exist" in each.)

---

## Module 8 — Functions
**Concepts:** defining, parameters, defaults, return values, scope, `*args`/`**kwargs`, lambdas.

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("u1")
greet("u1", "Hey")
greet(name="u1", greeting="Yo")

def total(*args):           # collects extra positional args into a tuple
    return sum(args)

total(1, 2, 3)

def describe(**kwargs):      # collects extra keyword args into a dict
    for k, v in kwargs.items():
        print(k, v)

describe(name="u1", level=12)

square = lambda x: x * x     # anonymous one-line function
```
**Scope:** a variable made *inside* a function disappears when the function returns, and can't be seen from outside. A function should generally take what it needs as parameters and return what it produces — not reach out and grab/modify variables from outside itself.

**Practice:** write a function `calculate_damage(base, multiplier=1.0, is_critical=False)` that returns `base * multiplier * (2 if is_critical else 1)`. Call it three different ways (positional, keyword, mixed).

**Checkpoint:** what happens if you try to print a variable that was only defined inside a function, from outside that function? Try it and read the error.

---

## Module 9 — Comprehensions
**Concepts:** list/dict/set comprehensions as a compact loop+condition pattern.

```python
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
lookup = {x: x**2 for x in range(5)}
unique_lengths = {len(w) for w in ["cat", "dog", "elephant"]}
```
A comprehension is just a `for` loop (with an optional `if`) that builds a collection, written on one line. It's not a new concept — it's Modules 4 and 6 combined into shorthand. If a comprehension gets hard to read, it's fine to write it as a plain loop instead.

**Practice:** given a list of words, use a comprehension to build a list of only the words longer than 4 characters, uppercased.

**Checkpoint:** rewrite one of your comprehensions above as a regular `for` loop that builds the same result, to prove to yourself they're equivalent.

---

## Module 10 — Error Handling
**Concepts:** exceptions, `try`/`except`/`else`/`finally`, raising your own.

```python
try:
    value = int(input("Enter a number: "))
except ValueError:
    print("That wasn't a number.")
else:
    print("Got it:", value)     # runs only if no exception
finally:
    print("This always runs")    # cleanup, runs no matter what

class InsufficientManaError(Exception):
    pass

def cast(mana, cost):
    if mana < cost:
        raise InsufficientManaError("Not enough mana")
    return mana - cost
```
An exception is Python's way of saying "something went wrong, and I'm stopping normal execution to say so." `try` lets you catch that instead of crashing. Catch the **specific** exception type you expect — a bare `except:` hides real bugs from you.

**Practice:** write a function that divides two numbers and handles `ZeroDivisionError` gracefully, printing a friendly message instead of crashing.

**Checkpoint:** what's the difference between code in `else` vs code just written after the `try` block with no `except` triggered? (Answer: functionally similar here, but `else` makes clear that code depends on success — it's a readability tool.)

---

## Module 11 — Files & I/O
**Concepts:** reading/writing files, context managers, JSON.

```python
with open("save.txt", "w") as f:
    f.write("level=12\n")

with open("save.txt") as f:
    for line in f:
        print(line.strip())

import json
data = {"name": "u1", "level": 12}
with open("save.json", "w") as f:
    json.dump(data, f)

with open("save.json") as f:
    loaded = json.load(f)
```
`with` is a **context manager** — it guarantees the file gets closed even if something errors out mid-write. This is the same pattern (setup → use → guaranteed cleanup) you'll see later for database connections and locks.

**Practice:** write a program that saves a dict of player stats to a JSON file, then a second program (or a second block) that reads it back and prints each stat.

**Checkpoint:** what would happen if you forgot `with` and just did `f = open(...)` without ever closing it? Why does that matter for a program that opens hundreds of files over time?

---

## Module 12 — Modules, Packages & pip
**Concepts:** organizing code across files, `import`, virtual environments, third-party packages.

```python
# helpers.py
def double(x):
    return x * 2

# main.py
import helpers
helpers.double(5)

from helpers import double
double(5)
```
A **module** is just a `.py` file. A **package** is a folder of modules. Splitting code into files is how programs stay manageable past a few hundred lines — one file per responsibility.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install requests
```
A virtual environment keeps each project's installed packages separate, so Project A needing `requests==2.0` and Project B needing `requests==3.0` don't conflict.

**Practice:** split your Module 10 damage-calculation function out into its own `combat.py` file, and import it into a `main.py` that calls it.

**Checkpoint:** why does every serious Python project use a virtual environment instead of installing packages globally?

---

## Module 13 — OOP: Classes & Objects
**Concepts:** classes as blueprints, `__init__`, instance attributes, methods, `self`.

```python
class Character:
    def __init__(self, name, hp=100):
        self.name = name        # instance attribute
        self.hp = hp

    def take_damage(self, amount):
        self.hp -= amount
        return self.hp

    def __repr__(self):
        return f"Character({self.name}, hp={self.hp})"

hero = Character("u1")
hero.take_damage(20)
print(hero)               # uses __repr__
```
A class is a template; each `Character(...)` call builds one **instance** with its own independent `self.name`/`self.hp`. `self` is just "the specific instance this method is running on" — Python passes it automatically.

**Practice:** build a `Player` class with `name`, `hp`, `level`. Add a method `level_up()` that increases level by 1 and hp by 10.

**Checkpoint:** if you create two `Character` instances and call `take_damage` on one, does the other one's `hp` change? Why or why not?

---

## Module 14 — OOP: Inheritance & Polymorphism
**Concepts:** subclassing, `super()`, overriding, dunder methods.

```python
class Mage(Character):                 # inherits from Character
    def __init__(self, name, mana=50):
        super().__init__(name)          # runs Character.__init__ first
        self.mana = mana

    def cast(self, spell):
        self.mana -= 10
        return f"{self.name} casts {spell}"

    def take_damage(self, amount):       # overrides the parent method
        amount = amount // 2             # mages take half damage, say
        return super().take_damage(amount)
```
Inheritance means "is a kind of" — a `Mage` **is a** `Character` with extra stuff. Overriding a method lets a subclass change behavior while still being able to call the parent's original version via `super()`. This is how you avoid copy-pasting near-identical classes.

**Practice:** build a `Warrior` class that inherits from your `Player` (Module 13), adds an `armor` attribute, and overrides `level_up()` to also increase armor.

**Checkpoint:** what's the difference between a `Mage` *inheriting from* `Character` versus a `Character` just *having* a `Mage` object as one of its attributes? When would you use each?

---

## Module 15 — Iterators & Generators
**Concepts:** how `for` loops actually work under the hood, lazy evaluation with `yield`.

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for i in countdown(5):
    print(i)

gen = countdown(3)
next(gen)    # 3
next(gen)    # 2
```
A generator function looks like a normal function but uses `yield` instead of `return` — it pauses and resumes instead of running start to finish and giving back one value. Useful when you want to produce a sequence without building the whole thing in memory at once (think: processing a huge file line by line).

**Practice:** write a generator `even_numbers(limit)` that yields even numbers from 0 up to `limit`, and loop over it with a `for`.

**Checkpoint:** what's the practical difference between `def f(): return [x for x in range(1000000)]` and a generator version using `yield`? When would the difference actually matter?

---

## Module 16 — Decorators
**Concepts:** functions that wrap other functions.

```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_call
def attack(target):
    return f"Attacking {target}"

attack("goblin")
```
`@log_call` above `def attack` is shorthand for `attack = log_call(attack)`. This is a natural extension of Module 8 (functions) and Module 15 (functions that return/produce other functions) — a decorator is just a function that takes a function and returns a new, wrapped one.

**Practice:** write a `@timer` decorator that prints how long a function took to run (use `time.time()` before and after the call).

**Checkpoint:** what does `func.__name__` refer to inside `wrapper`, and why is it there instead of just hardcoding the function's name as a string?

---

## Module 17 — Standard Library Tour
**Concepts:** the tools you reach for constantly, without installing anything.

```python
import random
random.randint(1, 6)         # dice roll
random.choice(["a", "b", "c"])
random.shuffle(my_list)        # in place

import datetime
datetime.date.today()
datetime.datetime.now()

import os
from pathlib import Path
Path("data").mkdir(exist_ok=True)
list(Path(".").glob("*.py"))

import re
re.findall(r"\d+", "there are 12 apples and 5 oranges")   # ['12', '5']

from collections import Counter, defaultdict
Counter(["a", "b", "a", "c", "a"])            # Counter({'a': 3, 'b': 1, 'c': 1})
```
Every module above ships with Python — no `pip install` needed. Knowing what's in the standard library saves you from reinventing (or unnecessarily installing) things that are already there.

**Practice:** write a script that generates 10 random dice rolls (1-6), then uses `Counter` to print how many times each value came up.

**Checkpoint:** name three standard-library modules from this list and, in one sentence each, what problem they solve.

---

## Module 18 — Testing & Debugging
**Concepts:** proving your code works, finding out why it doesn't.

```python
# combat.py
def calculate_damage(base, multiplier=1.0):
    return base * multiplier

# test_combat.py
from combat import calculate_damage

def test_calculate_damage():
    assert calculate_damage(10, 2.0) == 20
    assert calculate_damage(10) == 10
```
Run with `pytest` (`pip install pytest` first) from your project root — it auto-discovers any `test_*.py` file and any function named `test_*`.

**Debugging habits:**
- `print()` liberally at first — it's not "unsophisticated," it's the fastest feedback loop while learning.
- Read the **full** traceback, bottom to top — the bottom line is the actual error, the lines above show the call chain that led there.
- `python3 -m pdb myscript.py` drops you into an interactive debugger if you want to step line by line.

**Practice:** write two more `assert`-based tests for your `calculate_damage` function covering edge cases (multiplier of 0, negative base).

**Checkpoint:** why is a program with tests more trustworthy to change later than one without, even if both "work" right now?

---

## Module 19 — Type Hints
**Concepts:** optional static typing on top of a dynamic language.

```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str, times: int = 1) -> list[str]:
    return [f"Hi {name}"] * times
```
Type hints don't change how Python runs (nothing enforces them at runtime) — they're documentation the editor and tools like `mypy` can check for you. Worth adopting once your functions get non-obvious; skip them for quick throwaway scripts.

**Practice:** add type hints to three functions you've already written in earlier modules.

**Checkpoint:** if you hint a parameter as `int` but pass a `str`, does Python actually stop you at runtime? Try it and see.

---

## Module 20 — Capstone: Put It All Together
**Goal:** prove you're at novice level by building something that touches most of the modules above.

Build a **small text-based RPG combat sim**:
- `Character` base class + at least one subclass (Modules 13–14)
- Stats stored in a dict or as attributes (Modules 1, 6, 7)
- A combat loop using `while` (Module 4)
- Damage calculation as its own function, with type hints (Modules 8, 19)
- Save/load character state to a JSON file (Module 11)
- Handle bad input with `try`/`except` (Module 10)
- Split into at least two files: e.g. `characters.py` and `main.py` (Module 12)
- At least one `pytest` test for your damage function (Module 18)

This isn't busywork — it's deliberately shaped close to the kind of system you'd build a piece of in an actual game. If you finish this without needing to re-read earlier modules, you're at novice.

---

## What "novice" unlocks next
Once you're comfortable with everything above, the natural next steps are: `async`/`await` for concurrency, working with a real database (`sqlite3` ships built in), building a small web API (Flask or FastAPI), and packaging a project properly (`pyproject.toml`). Those weren't included here on purpose — they build on this foundation rather than being part of it.
