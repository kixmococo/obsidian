# GDScript for Absolute Beginners
*You've never written code before. That's the assumption for this whole document. Every line of code below gets broken apart piece by piece — what each symbol is doing there, and why it has to be written that way. Nothing is assumed.*

---

## Before Module 0: What even is "code"?

A computer only understands electrical signals — on and off. Nobody writes programs in on/off signals directly; that would be insane. Instead, we write instructions in a form humans can read (like `print("hello")`), and a program called a **compiler** or **interpreter** translates it into what the computer actually understands.

**GDScript** is the language. It's a list of instructions, written one per line, that Godot reads from top to bottom and carries out in order — like a recipe. A recipe says "crack the egg" then "whisk it" then "pour it in the pan," in that order, and you don't skip ahead. Code works the same way: line 1 happens, then line 2, then line 3.

A **script** is just a text file containing these instructions, saved with a `.gd` ending so Godot knows it's GDScript.

---

## Module 0: Your First Script, Explained Line by Line

```gdscript
extends Node

func _ready():
    print("this node just entered the scene tree")
```

Let's take this apart completely.

### Line 1: `extends Node`
- Godot games are built out of building blocks called **nodes** — think of a node as one "thing" in your game: a character, a sound, a timer, a button. Everything is a node.
- When you write a script, that script has to attach itself to one specific node, and it needs to say what *kind* of node it's enhancing. `extends Node` means "this script is adding behavior to a basic Node."
- Why the word "extends"? Because your script is *extending* — adding on to — the abilities that a `Node` already has by default. You're not replacing it, you're building on top of it.
- No parentheses, no colon, no indentation on this line — it's a simple one-time declaration, not an instruction that runs repeatedly.

### Line 2 (blank line)
- This does nothing. Blank lines are just there so humans can read the code more easily — the computer ignores them completely. Think of it like paragraph breaks in an essay.

### Line 3: `func _ready():`
- `func` is short for **function**. A function is a named, reusable block of instructions. You're telling Godot "here comes a group of steps, and I'm giving this group a name."
- `_ready` is the **name** of this specific function. It's not a name you made up — Godot specifically looks for a function named exactly `_ready` and automatically runs it one time, the moment this node is fully set up and ready to go. That's why it's called `_ready`.
- `()` — the parentheses — are where a function can receive **inputs** (called *parameters* or *arguments*), extra pieces of information it needs to do its job. `_ready` doesn't need any information from outside to do its job, so the parentheses are empty. You'll see functions later with things inside the parentheses, like `func take_damage(amount):` — that `amount` is a value getting handed *into* the function from whoever calls it. Think of the parentheses like a slot you can drop ingredients into before the function starts cooking.
- The `:` (colon) at the end means "everything indented underneath this line belongs to this function." It's like saying "here's what's inside this box, starting now."

### Line 4: `    print("this node just entered the scene tree")`
- The 4 spaces of indentation before `print` are not optional decoration — they are how GDScript knows this line is *inside* the `_ready` function, not just floating outside it on its own. If you un-indent it, it stops being part of `_ready`.
- `print(...)` is a function Godot gives you for free — its entire job is to display text in the Output panel at the bottom of the editor, so you (the human) can see what your program is doing while it runs.
- The text `"this node just entered the scene tree"` is called a **string** — any text wrapped in quotation marks. The quotes tell Godot "this is literal text, not an instruction, don't try to run it as code, just treat it as words."
- The parentheses around the string are, again, the "input slot" — you're handing that exact piece of text *into* the `print` function so it knows what to display.

**Checkpoint:** Why does `_ready()` have empty parentheses, but `print("...")` has text inside its parentheses? (Answer: parentheses hold whatever information a function needs from you to do its job — `print` needs to know *what* to display, so it needs an input; `_ready` doesn't need anything from you, Godot just calls it automatically.)

---

## Module 1: Variables — Explained From Zero

A **variable** is a labeled box that holds a piece of information, so you can refer back to it later by name instead of retyping the value every time.

```gdscript
var player_name = "u1"
var level = 12
```

- `var` is short for **variable**. Writing `var` is how you tell Godot "I'm creating a new labeled box right now."
- `player_name` is the **name** you're giving that box. You made this name up — it could be anything (as long as it doesn't start with a number or contain spaces). Good names describe what's inside, so future-you (or future-u1) can read the code and understand it without re-figuring it out.
- `=` here does **not** mean "equals" the way it does in math class. It means **"put this value into that box."** Read `var level = 12` out loud as "create a box called level, and put 12 inside it" — not "level equals 12."
- `"u1"` is a string (text in quotes), same as we saw with `print`. `12` has no quotes, because it's a **number**, not text — quotes specifically mean "treat this as literal words," and a number doesn't need that.

### Why do some variables have extra text like `: String`?
```gdscript
var player_name: String = "u1"
var level: int = 12
```
- The `: String` and `: int` are called **type hints** — they tell Godot exactly what *kind* of information is allowed to go in that box, forever. `String` means text. `int` means a whole number (no decimals). `float` means a number that can have decimals, like `1.8`. `bool` means the box can only ever hold `true` or `false`.
- Why bother? Because it stops mistakes early. If you write `var level: int = 12` and later accidentally try to put text into it (`level = "twelve"`), Godot will immediately warn you something's wrong — *before* you even run your game — instead of the mistake silently causing weird bugs five steps later that are much harder to track down.
- You don't have to use type hints (the version without `: int` still works), but it's a genuinely good habit, so it's used in every module from here on.

**Checkpoint:** In your own words, what's the difference between the *name* of a variable and the *value* stored inside it? Pick one variable you've seen and point to which part is which.

---

## Module 2: Doing Math and Comparing Things

```gdscript
var hp = 73
var is_healthy = hp > 50
```

- `hp > 50` is called an **expression** — a little calculation that produces a result. `>` means "is greater than." This whole expression produces either `true` or `false` — it's asking a yes/no question and the answer becomes the value.
- Because the answer to that question (`true` or `false`) is itself a value, you can store it in a variable, same as any other value — that's what `var is_healthy = hp > 50` is doing. It's not running `hp > 50` and printing it; it's saving the *answer* to that question in a box named `is_healthy`, so you can check it later.

### The math symbols and why some behave oddly
```gdscript
7 + 3     # 10, addition
7 - 3     # 4, subtraction
7 * 3     # 21, multiplication (a plain x isn't used because it looks like the letter x)
7 / 3     # 2  <-- NOT 2.33! Read below.
7.0 / 3    # 2.333... this time it works as expected
7 % 3       # 1, this is called "modulo" — it gives you the LEFTOVER after division
```
- Why does `7 / 3` give `2` and not `2.33`? Because both `7` and `3` are whole numbers (`int`s) with no decimal point. When GDScript divides two whole numbers, it assumes you want a whole number back, so it throws away anything after the decimal point. If you want the precise decimal answer, at least one of the numbers needs a decimal point in it (`7.0` instead of `7`) — that tells GDScript "I want a `float` (decimal number) answer."
- `%` (modulo) is confusing the first time you see it. It doesn't mean percent here — it means "divide, and tell me what's left over." `7 % 3` = 1, because 3 goes into 7 twice (6), with 1 left over. This comes up constantly for things like "is this number even?" (`number % 2 == 0`).

### The `#` symbol
- Anything after a `#` on a line is a **comment** — a note for humans, completely ignored by Godot. It doesn't run, it doesn't do anything, it's purely there so you (or someone reading your code later) can leave yourself explanations. Use it generously while learning.

**Checkpoint:** Why does `5 / 2` not give you `2.5`? What single character would you add, and where, to fix it?

---

## Module 3: if / elif / else — Making Decisions

```gdscript
var hp = 40

if hp <= 0:
    print("dead")
elif hp < 50:
    print("low health")
else:
    print("healthy")
```

Read this exactly like a flowchart, top to bottom:

- `if hp <= 0:` — asks a yes/no question: "is hp less than or equal to 0?" If the answer is yes, everything indented underneath runs, and Godot **skips the rest of this whole if/elif/else group entirely** — it does not keep checking the other conditions.
- If the answer to the `if` was no, Godot moves to `elif hp < 50:` — "elif" is short for "else, if" — meaning "okay, the first question was false, so let's ask a *different* question instead." Same deal: if this is true, run what's indented underneath, then skip the rest.
- `else:` is the catch-all — "if none of the questions above were true, do this instead." Notice `else` has no question after it — it doesn't need one, it's the leftover case.
- Again, the colon `:` after each line means "here comes an indented block that belongs to this condition," and the indentation is what visually (and functionally) groups those lines together.
- Why `<=` and not just `<`? `<=` means "less than **or equal to**." If you only wrote `<`, then `hp` being exactly `0` wouldn't count as "dead" — it would fall through to being checked against 50 instead, which is probably not what you want for a character at exactly 0 hp.

**Checkpoint:** If `hp` is exactly `50`, which branch runs — the `elif`, or the `else`? Trace through the question being asked and answer it yourself before checking (`hp < 50` — is 50 less than 50? No. So it falls to `else`).

---

## Module 4: Loops — Doing Something Over and Over

```gdscript
for i in range(5):
    print(i)
```

- `range(5)` produces a list of numbers: `0, 1, 2, 3, 4` (it starts at 0, and stops *before* reaching 5 — this trips everyone up at first, but it's consistent once you know it).
- `for i in range(5):` means "one at a time, take each number out of that list, temporarily call it `i`, and run the indented block below using that value of `i`." So this line runs 5 separate times, each time with `i` being a different number.
- `i` isn't a special word — it's just a variable name, same as `player_name` was earlier. Programmers commonly use short names like `i` for loop counters out of habit, but you could rename it to `number` and it would work identically.
- The indented `print(i)` below runs once per pass through the loop, printing whatever `i` currently is: first `0`, then `1`, then `2`, then `3`, then `4`.

```gdscript
var n = 3
while n > 0:
    print(n)
    n -= 1
```

- `while n > 0:` means "keep repeating the indented block below for as long as this question keeps answering yes." Unlike `for`, which runs a fixed number of times, `while` keeps going until its condition becomes false.
- `n -= 1` means "take whatever is currently in the box named `n`, subtract 1 from it, and put the result back in that same box." It's shorthand for `n = n - 1`. Without this line, `n` would stay `3` forever, the question `n > 0` would always be true, and your game would freeze forever repeating this loop — this is called an **infinite loop**, and it's one of the most common beginner mistakes. Always double check a `while` loop has some line inside it that eventually makes the condition become false.

**Checkpoint:** What would happen if you deleted the line `n -= 1` from the example above? Why specifically would that be a problem?

---

## Module 5: Text (Strings) in Depth

```gdscript
var s = "Hello, u1"
print(s[0])
```

- `s[0]` — the square brackets with a number inside are how you grab one specific character out of a string, by its position. Positions start counting at **0**, not 1 — so `s[0]` is `"H"` (the very first letter), and `s[1]` would be `"e"`. This "start counting at 0" rule is the same reason `range(5)` starts at 0 — it's consistent across the whole language, not a coincidence.

```gdscript
print("%s is level %d" % [name, level])
```
This one looks strange at first, so let's break it into pieces:
- `"%s is level %d"` is a string with two placeholders inside it. `%s` means "a piece of text goes here." `%d` means "a whole number goes here." Think of them as blanks to fill in, like a fill-in-the-blank sentence.
- The `%` right after the closing quote (outside the string) means "now fill in those blanks."
- `[name, level]` is a list of the actual values to drop into the blanks, in order — `name` fills the first blank (`%s`), `level` fills the second (`%d`). The square brackets here create what's called an **array** (covered fully next module) — for now, just know it's an ordered list of values.

**Checkpoint:** In `s[0]`, why does that give you the *first* character instead of the second? What rule from earlier in this guide also uses that same "start at 0" idea?

---

## Module 6: Arrays — Lists of Things

```gdscript
var scores = [10, 20, 30]
scores.append(40)
print(scores[0])
```

- `[10, 20, 30]` — square brackets containing values separated by commas create an **array**: an ordered list, all stored under one variable name, that you can add to, remove from, and look through.
- `scores.append(40)` — the `.` (dot) after `scores` means "reach inside this array and use one of the built-in tools it comes with." `append` is one such tool — its whole job is "add this new value to the end of the list." The `40` inside the parentheses is, again, the input being handed to that tool — same "parentheses are an input slot" idea from functions.
- `scores[0]` grabs the first item in the array (remember: counting starts at 0), which is `10`.
- Why the dot? Because `scores` isn't just a plain value like a number — it's a more complex object that comes bundled with its own set of built-in actions (`append`, `sort`, `size`, and more). The dot is how you say "look inside this thing and use one of its built-in abilities."

**Checkpoint:** If `scores = [10, 20, 30]`, what does `scores[2]` give you? Walk through the counting-from-0 rule to figure it out before checking (it's `30` — position 0 is `10`, position 1 is `20`, position 2 is `30`).

---

## Module 7: Dictionaries — Labeled Lists

```gdscript
var player = {"name": "u1", "level": 12}
print(player["level"])
```

- `{ }` (curly braces) create a **dictionary** — instead of positions like an array (`0, 1, 2...`), every value has its own custom text **key** you choose, and you look things up by that key instead of by position.
- `"name": "u1"` means "the key `name` points to the value `u1`." The colon separates the key (left side) from its value (right side), same visual idea as a real dictionary: word, then definition.
- `player["level"]` looks up the value stored under the key `"level"`, which gives you `12`. This uses the same square-bracket syntax as arrays, but instead of a position number inside, you put the key (as a string) instead.

**Checkpoint:** What's the practical difference between looking something up in an array (`scores[0]`) versus a dictionary (`player["level"]`)? Why might a dictionary be better for something like player stats, where each value has an obvious name?

---

## Module 8: Functions — The Full Explanation of Parentheses

You've seen functions since Module 0. Now let's fully unpack why they're built the way they are.

```gdscript
func calculate_damage(base: int, multiplier: float = 1.0) -> int:
    return int(base * multiplier)
```

- `func calculate_damage` — same as before: `func` announces a new function, `calculate_damage` is the name you're giving it.
- **The parentheses `(base: int, multiplier: float = 1.0)` are the function's input list.** This is the full answer to your question about parentheses: a function is a reusable set of instructions, but often it needs specific pieces of information to actually do its job — those pieces of information are called **parameters**, and they're declared inside the parentheses.
  - `base: int` says "this function requires one input, I'll call it `base` while I'm working with it inside this function, and it must be a whole number."
  - `multiplier: float = 1.0` declares a second input, `multiplier`, which must be a decimal number — and `= 1.0` gives it a **default value**. That means if whoever calls this function doesn't bother providing a multiplier, it'll automatically use `1.0` instead of causing an error.
  - Why have parameters at all instead of just typing the numbers straight into the function every time? Because it makes the function **reusable**. Without parameters, you'd need a separate function for every possible combination of base damage and multiplier. With parameters, one function handles all of them — you just hand it different values each time you call it.
- `-> int` after the closing parenthesis says "when this function finishes, it will hand back a whole number as its result." This is called the **return type**.
- `return int(base * multiplier)` — the `return` keyword is what actually hands a value back out of the function to whoever called it. `int(...)` here converts whatever comes out of `base * multiplier` into a whole number, in case multiplying created a decimal (e.g. `10 * 1.5 = 15.0`, and `int(15.0)` gives you plain `15`).

### How you'd actually use this function
```gdscript
var dmg = calculate_damage(10, 2.0)
```
- This line **calls** the function — it's saying "run `calculate_damage`, and specifically, use `10` as the `base` and `2.0` as the `multiplier`."
- Inside the function, while it's running, `base` temporarily equals `10` and `multiplier` temporarily equals `2.0` — those names only exist and only mean something while this specific function call is happening.
- Whatever `calculate_damage` returns gets stored in the new variable `dmg`, because of the `=`, same rule as every other variable assignment you've seen.

**Checkpoint:** Why does `calculate_damage(10, 2.0)` work without writing `base:` and `multiplier:` again when calling it? (Answer: those labels are only needed once, when you *define* the function and its expected inputs. When you *call* it, you just hand over the values in the same order the parameters were declared.)

---

## Module 9: `.map()`, `.filter()` — Transforming Whole Arrays at Once

```gdscript
var numbers = [1, 2, 3, 4, 5]
var squares = numbers.map(func(x): return x * x)
```

This is the densest-looking line so far — take it slow.

- `numbers.map(...)` — again, the dot means "use a built-in tool that arrays come with." `map`'s job specifically is: "go through every single item in this array, run some function on each one, and give me back a brand new array full of the results."
- `func(x): return x * x` — this is a function, exactly like the ones from Module 8, except it has no name. This is called an **anonymous function** or **lambda**. It's written inline, right here, purely because it's small and only used in this one spot — not worth giving it a full name and writing it elsewhere.
- `x` here is that function's one parameter — same concept as `base` earlier, just a made-up name for "whatever single item from the array is currently being processed."
- So the whole line reads as: "for every number in `numbers`, square it, and collect all those squared results into a new array called `squares`."

**Checkpoint:** Could you rewrite this same logic using a `for` loop from Module 4 instead of `.map()`? Try writing it both ways and compare — they do the exact same thing, just structured differently.

---

## Module 10: Error Handling — Why There's No try/except Here

If you've seen other languages mention "try/catch" or "exceptions," GDScript deliberately doesn't work that way. Here's the reasoning, then the actual pattern.

```gdscript
func divide(a: float, b: float) -> float:
    if b == 0:
        push_error("Division by zero attempted")
        return 0.0
    return a / b
```

- Instead of trying something and hoping it doesn't crash, GDScript wants you to **check for the problem before it happens.** `if b == 0:` is literally asking "is the thing that would break this about to happen?" *before* attempting the division.
- `==` (two equals signs) means "is equal to," used for asking a question. This is different from a single `=`, which means "store this value." Mixing these two up is one of the most common beginner mistakes in almost every programming language — `=` assigns, `==` asks.
- `push_error("...")` doesn't stop your game — it just writes a clearly-marked error message to Godot's Debugger panel, so you (the developer) notice something went wrong, while the game keeps running instead of crashing entirely.
- `return 0.0` then exits the function early, handing back a safe fallback value (zero) instead of ever attempting the division that would have caused a problem.
- If `b` isn't `0`, none of that code runs, and Godot falls through to the final line, `return a / b`, which does the actual division safely.

**Checkpoint:** Why is checking `if b == 0:` *before* dividing better than just letting the division happen and dealing with whatever error comes out of it?

---

## Module 11: Saving to a File

```gdscript
var file = FileAccess.open("user://save.txt", FileAccess.WRITE)
file.store_string("level=12\n")
file.close()
```

- `FileAccess` is a built-in Godot tool specifically for reading and writing files. `.open(...)` is one of its built-in actions — its job is to open a file and hand you back a working connection to it, which gets stored in the `file` variable.
- `"user://save.txt"` — the text being handed in as the first input tells `open` *where* the file should live. `user://` is a special Godot shortcut meaning "the folder on the player's actual computer where it's safe to save game data" (as opposed to `res://`, which means "files bundled inside the game itself, which shouldn't be changed while playing").
- `FileAccess.WRITE` is the second input — it tells `open` *what you intend to do* with this file: write new data into it (there's also `FileAccess.READ` for reading a file instead).
- `file.store_string("level=12\n")` uses the `file` connection you just opened to actually write text into it. The `\n` inside the string is a special two-character code meaning "start a new line here" — you can't just press Enter inside a string, so `\n` is the stand-in for that.
- `file.close()` closes the connection to the file, telling the computer you're done with it. This matters because leaving files open unnecessarily can cause your changes not to actually save, or can lock the file so nothing else can use it.

**Checkpoint:** What do you think would happen if you deleted the `file.close()` line — would `store_string` still have written to the file? (In practice, Godot often saves it anyway when the variable goes away, but explicitly closing it is the safe, intentional habit — don't rely on the automatic cleanup.)

---

## Module 12–20: A Note Before You Continue

Everything from here (organizing code across files, classes, inheritance, signals, nodes/scenes, randomness, testing, and the capstone project) builds directly on the concepts explained above — variables, functions and their parentheses, if/else, loops, arrays, and dictionaries. Rather than re-explaining every symbol again at this level of detail for every remaining topic, the original **GDScript in Modules** guide covers those next steps well — go back to that file for Modules 12 through 20, and re-read this document any time a piece of syntax there looks unfamiliar. The pattern for figuring out any new line of code is always the same three questions:

1. **What is this word or symbol's job?** (`var` makes a box, `func` makes a reusable set of steps, `if` asks a question)
2. **What's inside the parentheses, and why?** (it's an input the function needs to do its job)
3. **What's indented underneath, and why?** (it's the block of instructions that belongs to the line above it)

Ask those three questions on any line you don't understand, and you'll almost always be able to work it out yourself.

---

## Quick Reference: Symbols You Now Understand

| Symbol | Meaning |
|---|---|
| `var` | creates a new labeled box (variable) |
| `=` | puts a value into a box (assignment) |
| `==` | asks "are these equal?" (comparison) |
| `func name():` | defines a reusable set of instructions |
| `( )` after a function | the inputs that function needs |
| `-> type` | what the function hands back when done |
| `:` at the end of a line | "an indented block follows, belonging to this line" |
| indentation | groups lines together as belonging to the thing above |
| `#` | a comment — a note for humans, ignored by the computer |
| `[ ]` with commas | an array — an ordered list |
| `{ }` with `key: value` pairs | a dictionary — a labeled lookup table |
| `.` after a variable | "use one of this thing's built-in abilities" |
| `\n` inside a string | "start a new line" (can't just press Enter in a string) |
