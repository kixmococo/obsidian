# C# Cheatsheet: Booleans
_A simple, detailed, overexplained guide_

---

## What this cheatsheet is about

A **boolean** is a value that can only be one of two things:

- `true`
- `false`

That is it.

A boolean is basically C#'s way of answering a **yes-or-no question**.

You can think of a boolean like a switch:

- on / off
- yes / no
- true / false
- 1-bit decision

In programming, booleans are used constantly for:

- checking conditions
- making decisions
- controlling loops
- comparing values
- deciding whether something should happen

---

# Part 1: What is a boolean?

In C#, the type for a boolean is:

```csharp
bool
```

## Example

```csharp
bool isRunning = true;
bool isDead = false;
```

### What this means

- `isRunning` stores `true`
- `isDead` stores `false`

So a boolean variable holds a truth value.

---

# Part 2: Why booleans matter

Booleans are what make decisions possible in code.

Without booleans, code would just do the same thing every time.

With booleans, code can ask questions like:

- Is the player alive?
- Is the number bigger than 10?
- Is the password correct?
- Is the user logged in?
- Should this loop continue?

That is why booleans are everywhere.

---

# Part 3: Declaring a boolean

## Example

```csharp
bool isHappy = true;
```

### Breakdown

- `bool` = the data type
- `isHappy` = the variable name
- `=` = assignment
- `true` = the value being stored

Another example:

```csharp
bool hasKey = false;
```

---

# Part 4: Printing a boolean

```csharp
bool hasSword = true;
Console.WriteLine(hasSword);
```

## Output

```text
True
```

Notice that when C# prints a boolean, it shows:

- `True`
- `False`

with capital first letters in the output.

But in code, you write:

- `true`
- `false`

all lowercase.

That difference matters.

---

# Part 5: Booleans from comparisons

A very common way to get a boolean is from a comparison.

## Example

```csharp
bool result = 5 > 3;
Console.WriteLine(result);
```

## Output

```text
True
```

Why?

Because `5 > 3` is true.

So the boolean variable stores `true`.

---

# Part 6: Comparison operators

These operators usually produce a boolean result.

## Equal to

```csharp
==
```

Example:

```csharp
5 == 5
```

This asks:

> "Is 5 equal to 5?"

Answer: `true`

---

## Not equal to

```csharp
!=
```

Example:

```csharp
5 != 3
```

This asks:

> "Is 5 not equal to 3?"

Answer: `true`

---

## Greater than

```csharp
>
```

Example:

```csharp
10 > 4
```

Answer: `true`

---

## Less than

```csharp
<
```

Example:

```csharp
2 < 7
```

Answer: `true`

---

## Greater than or equal to

```csharp
>=
```

Example:

```csharp
5 >= 5
```

Answer: `true`

Because 5 is equal to 5.

---

## Less than or equal to

```csharp
<=
```

Example:

```csharp
4 <= 9
```

Answer: `true`

---

# Part 7: Examples of comparisons making booleans

```csharp
bool a = 10 > 5;
bool b = 3 < 1;
bool c = 7 == 7;
bool d = 8 != 8;
```

### Values stored

- `a` = `true`
- `b` = `false`
- `c` = `true`
- `d` = `false`

---

# Part 8: Booleans in `if` statements

This is one of the biggest uses of booleans.

## Example

```csharp
bool isLoggedIn = true;

if (isLoggedIn)
{
    Console.WriteLine("Welcome back.");
}
```

### What this means

The `if` statement checks:

> "Is `isLoggedIn` true?"

If yes, the code inside runs.

---

# Part 9: `if` with a direct comparison

```csharp
int age = 20;

if (age >= 18)
{
    Console.WriteLine("Adult");
}
```

### What happens

The condition is:

```csharp
age >= 18
```

That creates a boolean.

Since `20 >= 18` is true, the code runs.

So an `if` statement always depends on a boolean condition.

---

# Part 10: `if` and `else`

```csharp
int health = 0;

if (health > 0)
{
    Console.WriteLine("Player is alive.");
}
else
{
    Console.WriteLine("Player is dead.");
}
```

### Why this works

- if `health > 0` is true → first block runs
- otherwise → `else` block runs

---

# Part 11: Boolean operators

These let you combine or flip boolean values.

The main ones are:

- `&&` = AND
- `||` = OR
- `!` = NOT

---

# Part 12: AND operator `&&`

AND means:

> both sides must be true

## Example

```csharp
bool hasTicket = true;
bool hasId = true;

bool canEnter = hasTicket && hasId;
Console.WriteLine(canEnter);
```

## Output

```text
True
```

Because both are true.

### Another example

```csharp
bool hasTicket = true;
bool hasId = false;

bool canEnter = hasTicket && hasId;
```

Now `canEnter` becomes `false` because both sides are not true.

---

# Part 13: OR operator `||`

OR means:

> at least one side must be true

## Example

```csharp
bool isAdmin = false;
bool isModerator = true;

bool hasAccess = isAdmin || isModerator;
Console.WriteLine(hasAccess);
```

## Output

```text
True
```

Because at least one side is true.

---

# Part 14: NOT operator `!`

NOT flips a boolean.

- `true` becomes `false`
- `false` becomes `true`

## Example

```csharp
bool lightsOn = true;
Console.WriteLine(!lightsOn);
```

## Output

```text
False
```

Because NOT reverses it.

---

# Part 15: Boolean truth table idea

These are worth memorizing.

## AND `&&`

- `true && true` = `true`
- `true && false` = `false`
- `false && true` = `false`
- `false && false` = `false`

## OR `||`

- `true || true` = `true`
- `true || false` = `true`
- `false || true` = `true`
- `false || false` = `false`

## NOT `!`

- `!true` = `false`
- `!false` = `true`

---

# Part 16: Combined boolean conditions

```csharp
int age = 25;
bool hasPass = true;

if (age >= 18 && hasPass)
{
    Console.WriteLine("You may enter.");
}
```

### What this means

This asks two things:

- Is age at least 18?
- Does the person have a pass?

Both must be true because of `&&`.

---

# Part 17: Another combined example

```csharp
bool isWeekend = false;
bool isHoliday = true;

if (isWeekend || isHoliday)
{
    Console.WriteLine("You do not need to work.");
}
```

This works because only one side has to be true with `||`.

---

# Part 18: Using `!` in conditions

```csharp
bool isBanned = false;

if (!isBanned)
{
    Console.WriteLine("User is allowed in.");
}
```

### What this means

`!isBanned` means:

> "NOT banned"

So if `isBanned` is false, then `!isBanned` becomes true.

---

# Part 19: Boolean variables should usually sound like questions or states

Good boolean variable names often sound like yes/no ideas.

## Good examples

```csharp
bool isAlive = true;
bool hasAmmo = false;
bool canJump = true;
bool isVisible = false;
bool needsReload = true;
```

These are good because they read naturally.

Example:

```csharp
if (isAlive)
```

reads like:

> "if isAlive"

That makes sense.

---

# Part 20: Comparing strings gives booleans too

```csharp
string password = "cat";
bool isCorrect = password == "cat";
Console.WriteLine(isCorrect);
```

## Output

```text
True
```

So comparisons are not just for numbers.  
They can also be used with strings.

---

# Part 21: Comparing chars gives booleans too

```csharp
char grade = 'A';
bool isA = grade == 'A';
Console.WriteLine(isA);
```

## Output

```text
True
```

---

# Part 22: Booleans control loops too

## Example with `while`

```csharp
bool gameRunning = true;
int turns = 0;

while (gameRunning)
{
    Console.WriteLine("Game is running.");
    turns++;

    if (turns == 3)
    {
        gameRunning = false;
    }
}
```

### What happens

- loop starts because `gameRunning` is true
- after 3 turns, it becomes false
- the loop stops

So booleans often act like control switches.

---

# Part 23: Boolean expressions

A **boolean expression** is any expression that becomes true or false.

## Examples

```csharp
5 > 3
age >= 18
name == "Kai"
hasKey && doorUnlocked
!isDead
```

All of these evaluate to either `true` or `false`.

That means all of them are boolean expressions.

---

# Part 24: Assigning a boolean from an expression

```csharp
int score = 95;
bool passed = score >= 70;
Console.WriteLine(passed);
```

## Output

```text
True
```

Because `95 >= 70` is true.

This is common and useful.

---

# Part 25: Using a boolean directly vs comparing to `true`

Sometimes beginners write this:

```csharp
if (isReady == true)
{
    Console.WriteLine("Ready");
}
```

That works, but usually this is cleaner:

```csharp
if (isReady)
{
    Console.WriteLine("Ready");
}
```

These mean the same thing.

For false, beginners may write:

```csharp
if (isReady == false)
```

But a cleaner version is:

```csharp
if (!isReady)
```

---

# Part 26: Example of clean boolean style

```csharp
bool hasFuel = true;

if (hasFuel)
{
    Console.WriteLine("The car can move.");
}

if (!hasFuel)
{
    Console.WriteLine("The car cannot move.");
}
```

---

# Part 27: Common beginner mistakes

## Mistake 1: Using `=` instead of `==`

Wrong idea:

```csharp
if (x = 5)
```

Why wrong?

- `=` means assign
- `==` means compare

Correct:

```csharp
if (x == 5)
```

---

## Mistake 2: Forgetting that booleans are only true or false

A boolean is not:

- 7
- "hello"
- 3.14

It must be:

- `true`
- `false`

---

## Mistake 3: Making conditions more complicated than needed

Example:

```csharp
if (isOpen == true)
```

Works, but cleaner is:

```csharp
if (isOpen)
```

---

## Mistake 4: Confusing `!` with `!=`

These are different.

## `!`
NOT

Example:

```csharp
!isAlive
```

## `!=`
not equal to

Example:

```csharp
score != 100
```

Very different jobs.

---

# Part 28: Boolean examples with numbers

```csharp
int x = 10;
int y = 20;

bool a = x < y;
bool b = x > y;
bool c = x == 10;
bool d = y != 15;
```

### Results

- `a` = true
- `b` = false
- `c` = true
- `d` = true

---

# Part 29: Boolean examples with multiple conditions

```csharp
int age = 19;
bool hasId = true;
bool hasMoney = false;

bool canBuy = age >= 18 && hasId && hasMoney;
Console.WriteLine(canBuy);
```

## Output

```text
False
```

Why false?

Because even though age and ID are okay, `hasMoney` is false.

With AND, everything must be true.

---

# Part 30: Booleans in real-world style game logic

```csharp
bool isAlive = true;
bool hasSword = true;
bool enemyNearby = false;

if (isAlive && hasSword && enemyNearby)
{
    Console.WriteLine("Attack!");
}
else
{
    Console.WriteLine("Cannot attack right now.");
}
```

The result here would be:

```text
Cannot attack right now.
```

Because `enemyNearby` is false.

---

# Part 31: Simple mental model

If you ever get confused, remember:

A boolean answers a yes/no question.

Examples:

- Is the door open?
- Is the player alive?
- Is this number bigger?
- Can the user log in?

If it is a yes/no idea, it is probably boolean logic.

---

# Part 32: Sample program that demonstrates booleans

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== BASIC BOOLEAN VALUES ===");
        bool isAlive = true;
        bool hasKey = false;

        Console.WriteLine("isAlive: " + isAlive);
        Console.WriteLine("hasKey: " + hasKey);

        Console.WriteLine();
        Console.WriteLine("=== BOOLEANS FROM COMPARISONS ===");
        int x = 10;
        int y = 5;

        bool isGreater = x > y;
        bool isEqual = x == y;
        bool isNotEqual = x != y;

        Console.WriteLine("x > y: " + isGreater);
        Console.WriteLine("x == y: " + isEqual);
        Console.WriteLine("x != y: " + isNotEqual);

        Console.WriteLine();
        Console.WriteLine("=== IF STATEMENT WITH BOOLEAN ===");
        if (isAlive)
        {
            Console.WriteLine("The player is alive.");
        }

        if (!hasKey)
        {
            Console.WriteLine("The player does not have the key.");
        }

        Console.WriteLine();
        Console.WriteLine("=== BOOLEAN OPERATORS ===");
        bool hasTicket = true;
        bool hasId = true;
        bool canEnter = hasTicket && hasId;

        Console.WriteLine("canEnter with AND: " + canEnter);

        bool isAdmin = false;
        bool isModerator = true;
        bool hasAccess = isAdmin || isModerator;

        Console.WriteLine("hasAccess with OR: " + hasAccess);

        bool lightsOn = true;
        Console.WriteLine("NOT lightsOn: " + !lightsOn);

        Console.WriteLine();
        Console.WriteLine("=== COMBINED CONDITIONS ===");
        int age = 20;
        bool hasPass = true;

        if (age >= 18 && hasPass)
        {
            Console.WriteLine("User may enter the event.");
        }
        else
        {
            Console.WriteLine("User may not enter the event.");
        }

        Console.WriteLine();
        Console.WriteLine("=== BOOLEAN CONTROLLING A LOOP ===");
        bool gameRunning = true;
        int turns = 0;

        while (gameRunning)
        {
            Console.WriteLine("Game turn: " + turns);
            turns++;

            if (turns == 3)
            {
                gameRunning = false;
            }
        }

        Console.WriteLine("Game ended.");

        Console.WriteLine();
        Console.WriteLine("=== STRING AND CHAR COMPARISONS ===");
        string password = "cat";
        bool correctPassword = password == "cat";
        Console.WriteLine("Password correct: " + correctPassword);

        char grade = 'A';
        bool isA = grade == 'A';
        Console.WriteLine("Grade is A: " + isA);

        Console.WriteLine();
        Console.WriteLine("=== FINAL GAME STYLE EXAMPLE ===");
        bool playerAlive = true;
        bool playerHasSword = true;
        bool enemyNearby = true;

        if (playerAlive && playerHasSword && enemyNearby)
        {
            Console.WriteLine("Attack the enemy!");
        }
        else
        {
            Console.WriteLine("Conditions are not met for attacking.");
        }
    }
}
```

---

# Part 33: What the final program teaches

This one program shows:

- plain boolean variables
- booleans from comparisons
- booleans in `if` statements
- `&&`
- `||`
- `!`
- combined conditions
- booleans controlling loops
- string comparison
- char comparison
- a game-style boolean decision

---

# Part 34: Final recap

## Boolean type:
```csharp
bool
```

## Boolean values:
```csharp
true
false
```

## Comparison operators:
- `==`
- `!=`
- `>`
- `<`
- `>=`
- `<=`

## Boolean operators:
- `&&` = AND
- `||` = OR
- `!` = NOT

## Main purpose:
Booleans let your program make decisions.

---

# Part 35: One sentence memory hook

**A boolean is a yes-or-no value in C#, and it powers conditions, comparisons, decisions, and control flow.**
