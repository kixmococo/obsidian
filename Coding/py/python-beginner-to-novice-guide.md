# Python: Beginner → Novice Guide
*A copilot's field manual — from "what's a terminal" to writing real programs.*

---

## Table of Contents
1. [Getting Python Running (Linux, Mac, Windows)](#1-getting-python-running)
2. [Your Toolkit (Editor, REPL, Package Manager)](#2-your-toolkit)
3. [Core Language Concepts, Mapped to Python](#3-core-language-concepts)
4. [Data Structures](#4-data-structures)
5. [Functions & Scope](#5-functions--scope)
6. [Object-Oriented Programming](#6-object-oriented-programming)
7. [Error Handling](#7-error-handling)
8. [Files, I/O, and Working with Data](#8-files-io-and-working-with-data)
9. [Modules, Packages & the Standard Library](#9-modules-packages--the-standard-library)
10. [Intermediate Concepts (your "novice" ramp)](#10-intermediate-concepts)
11. [Practice Apps & Where to Drill](#11-practice-apps--where-to-drill)
12. [Tutorials, Docs & Communities](#12-tutorials-docs--communities)
13. [Practical Projects (build these)](#13-practical-projects)
14. [Cheat Sheet](#14-cheat-sheet)

---

## 1. Getting Python Running

### Linux (your likely home turf)
Most distros ship Python 3 already. Check first:
```bash
python3 --version
```
If it's missing or old, install the latest stable line (Python 3.14 as of mid-2026):

**Debian/Ubuntu:**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```
**Fedora:**
```bash
sudo dnf install python3 python3-pip
```
**Arch:**
```bash
sudo pacman -S python python-pip
```
If your distro's repo is behind, use **pyenv** to install any version without touching the system Python:
```bash
curl https://pyenv.run | bash
# add pyenv init to your .bashrc/.zshrc per its instructions, then:
pyenv install 3.14.6
pyenv global 3.14.6
```

### macOS
```bash
brew install python
```

### Windows
Download the installer from python.org — check **"Add python.exe to PATH"** during install. Or `winget install Python.Python.3.14`.

### Verify it works
```bash
python3 --version
python3 -c "print('hello, world')"
```

---

## 2. Your Toolkit

- **Editor:** VS Code (free, huge Python extension) or JetBrains PyCharm (heavier, batteries-included). Since you already work in editors for Godot/JS, VS Code will feel familiar.
- **REPL:** Just type `python3` in a terminal for an interactive shell — great for testing one-liners. `Ctrl+D` to exit.
- **Virtual environments** (critical habit — keeps each project's dependencies isolated):
```bash
python3 -m venv .venv          # create it, once per project
source .venv/bin/activate      # activate it (every session)
deactivate                     # leave it
```
- **pip** — the package installer:
```bash
pip install requests
pip install -r requirements.txt
pip freeze > requirements.txt  # snapshot what's installed
```
- **Newer alternative:** `uv` (from Astral) is a much faster drop-in replacement for pip/venv if you want to try something modern:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv venv && uv pip install requests
```

---

## 3. Core Language Concepts

### Variables & types
```python
name = "u1"            # str
age = 30                # int
height = 1.8             # float
is_dev = True            # bool
nothing = None            # NoneType
```
Python is **dynamically typed** (no declarations) but **strongly typed** (no silent `"5" + 5`).

### Operators
```python
7 // 2      # 3   floor division
7 % 2       # 1   modulo
2 ** 8      # 256 exponent
a, b = b, a  # swap, Python party trick
```

### Control flow
```python
if age >= 18:
    print("adult")
elif age >= 13:
    print("teen")
else:
    print("kid")

for i in range(5):
    print(i)

n = 5
while n > 0:
    n -= 1
```
No switch statement until 3.10's `match`:
```python
match command:
    case "move":
        ...
    case "attack" | "defend":
        ...
    case _:
        print("unknown")
```

### Truthiness & indentation
Python uses **whitespace, not braces**, to define blocks — indentation is syntax, not style. `0`, `""`, `[]`, `{}`, `None` are all falsy.

---

## 4. Data Structures

| Type | Ordered | Mutable | Example |
|---|---|---|---|
| `list` | yes | yes | `[1, 2, 3]` |
| `tuple` | yes | no | `(1, 2, 3)` |
| `dict` | yes (3.7+) | yes | `{"key": "value"}` |
| `set` | no | yes | `{1, 2, 3}` |

```python
scores = [10, 20, 30]
scores.append(40)
scores[0]           # 10
scores[-1]           # 40 (negative indexing)
scores[1:3]           # [20, 30] slicing

player = {"name": "u1", "level": 12}
player["level"] += 1
player.get("mana", 0)   # safe lookup with default

# comprehensions — Python's signature move
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
lookup = {x: x**2 for x in range(5)}
```

---

## 5. Functions & Scope

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("u1")                       # positional
greet(name="u1", greeting="Hey")   # keyword

def total(*args, **kwargs):
    print(args)     # tuple of positional extras
    print(kwargs)    # dict of keyword extras

square = lambda x: x * x   # anonymous one-liner
```
Scope: variables defined inside a function are **local** unless declared `global` or `nonlocal`. Default rule of thumb — avoid globals, pass things in and return things out.

---

## 6. Object-Oriented Programming

```python
class Character:
    species = "human"          # class attribute, shared

    def __init__(self, name, hp=100):
        self.name = name        # instance attribute
        self.hp = hp

    def take_damage(self, amount):
        self.hp -= amount
        return self.hp

    def __repr__(self):
        return f"Character({self.name}, hp={self.hp})"

class Mage(Character):          # inheritance
    def __init__(self, name, mana=50):
        super().__init__(name)
        self.mana = mana

    def cast(self, spell):
        self.mana -= 10
        return f"{self.name} casts {spell}"
```
Key ideas: **encapsulation** (attributes/methods bundled), **inheritance** (`Mage` extends `Character`), **polymorphism** (subclasses override methods), dunder methods like `__init__`, `__repr__`, `__eq__`, `__len__` let your objects plug into Python's built-in behaviors.

---

## 7. Error Handling

```python
try:
    value = int(input("Enter a number: "))
except ValueError:
    print("That wasn't a number.")
except (TypeError, KeyError) as e:
    print(f"Something else went wrong: {e}")
else:
    print("No errors — this runs if try succeeded")
finally:
    print("Always runs — cleanup goes here")

# raising your own
class InsufficientManaError(Exception):
    pass

if mana < cost:
    raise InsufficientManaError("Not enough mana to cast that")
```
Rule of thumb: catch specific exceptions, never a bare `except:` — it swallows real bugs.

---

## 8. Files, I/O, and Working with Data

```python
with open("save.txt", "w") as f:
    f.write("level=12\n")

with open("save.txt") as f:
    for line in f:
        print(line.strip())
```
`with` is a **context manager** — it guarantees the file closes even if an error happens mid-write. This pattern shows up everywhere (database connections, locks, network sockets).

```python
import json
data = {"name": "u1", "level": 12}
with open("save.json", "w") as f:
    json.dump(data, f)

with open("save.json") as f:
    loaded = json.load(f)
```

```python
import csv
with open("scores.csv") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["score"])
```

---

## 9. Modules, Packages & the Standard Library

```python
# mymodule.py
def helper():
    return "I'm reusable"

# main.py
import mymodule
mymodule.helper()

from mymodule import helper
helper()
```
A folder with an `__init__.py` (or just a folder in modern Python) becomes a **package** — a namespace of modules.

**Standard library worth knowing early:**
- `os`, `pathlib` — filesystem paths and operations
- `sys` — interpreter internals, command-line args (`sys.argv`)
- `datetime` — dates and times
- `random` — randomness (dice rolls, shuffles — handy for game logic)
- `re` — regular expressions
- `collections` — `Counter`, `defaultdict`, `deque`
- `itertools` — combinatorics, infinite iterators
- `math` — the usual suspects
- `subprocess` — run shell commands from Python
- `argparse` — build real CLI tools

**Third-party (via pip), the popular ones:**
- `requests` — HTTP calls
- `pygame` — 2D games (worth a look given your Godot/JS background — good way to compare engines' philosophies to a bare-metal Python loop)
- `numpy` / `pandas` — numeric and tabular data
- `flask` / `fastapi` — web servers
- `pytest` — testing

---

## 10. Intermediate Concepts

These are the concepts that separate "wrote a script" from "novice programmer":

**Generators** — lazy iterators, don't build the whole list in memory:
```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for i in countdown(5):
    print(i)
```

**Decorators** — functions that wrap other functions:
```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_call
def attack(target):
    print(f"Attacking {target}")
```

**Type hints** (optional, but increasingly standard):
```python
def add(a: int, b: int) -> int:
    return a + b
```

**f-strings** (formatting, use these over `%` or `.format()`):
```python
name, hp = "u1", 80
print(f"{name} has {hp} HP ({hp/100:.0%})")
```

**Unpacking & `*`/`**`:**
```python
first, *rest = [1, 2, 3, 4]
def merge(**kwargs): return kwargs
merge(**{"a": 1}, **{"b": 2})
```

**Testing** — the habit that turns hobby code into real software:
```python
# test_math_utils.py
from math_utils import add

def test_add():
    assert add(2, 3) == 5
```
Run with `pytest` from the project root.

**Environments & dependency files** — `requirements.txt` or a `pyproject.toml` so your projects are reproducible on another machine.

---

## 11. Practice Apps & Where to Drill

- **Exercism.org** — mentored exercises with real feedback, has a solid Python track
- **CodingBat / Codewars** — short, punchy problems to build reflexes
- **Advent of Code** (adventofcode.com) — yearly puzzle set, great once you know the basics; very game-dev-brain-friendly
- **LeetCode / HackerRank** — once you want interview-style algorithm practice
- **Project Euler** — math-flavored problems if that's your taste
- **PyGame Zero / pygame** — since you already build games, recreating a simple Pong or Snake in pygame is one of the fastest ways to cement functions, classes, and loops

---

## 12. Tutorials, Docs & Communities

- **Official docs:** https://docs.python.org/3/tutorial/ — genuinely well-written, start here
- **Official install page:** https://www.python.org/downloads/
- **Real Python** (realpython.com) — deep, practical, well-explained articles/tutorials for nearly every topic above
- **Automate the Boring Stuff** (automatetheboringstuff.com) — free online book, extremely beginner-friendly, practical projects
- **freeCodeCamp Python course** — long-form free video course
- **Python Discord** (pythondiscord.com) — active community, good for real-time help
- **r/learnpython** — beginner-friendly subreddit
- **PEP 8** (python.org/dev/peps/pep-0008) — the style guide; worth skimming once you're past total basics

---

## 13. Practical Projects

Ordered roughly beginner → novice:
1. **CLI to-do list** — practices files, lists, dicts, control flow
2. **Number guessing game** — practices loops, `random`, conditionals
3. **JSON-backed contact book** — practices file I/O, functions, dicts
4. **Simple text adventure** (classes for Room/Player/Item) — practices OOP, directly transferable to your Godot narrative-game instincts
5. **Pong or Snake in pygame** — practices game loop structure, state, collision logic — a nice mirror to compare against how Godot handles the same problems
6. **Web scraper with `requests` + `re`/BeautifulSoup** — practices modules, error handling, real-world messiness
7. **Small Flask/FastAPI API with a JSON "database"** — practices packages, testing, real project structure

---

## 14. Cheat Sheet

```python
# strings
s.strip(); s.split(","); s.lower(); s.replace("a","b"); "-".join(list)

# lists
lst.sort(); sorted(lst); lst.reverse(); len(lst); x in lst

# dicts
d.keys(); d.values(); d.items(); d.get(k, default)

# common patterns
[x for x in lst if cond]           # list comprehension
{k: v for k, v in pairs}           # dict comprehension
enumerate(lst)                      # index + value
zip(list1, list2)                   # pair up two lists
sorted(lst, key=lambda x: x.hp)      # custom sort
```

---

*Next step from here: pick one project from section 13, build it badly, then rebuild it well. That loop is basically the whole novice-to-intermediate arc.*
