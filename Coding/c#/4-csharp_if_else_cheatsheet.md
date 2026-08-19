# C# Cheatsheet: If, Else If, and Else
_A simple, detailed, overexplained guide_

---

## What this cheatsheet is about

An `if` statement is how C# makes a decision.

It lets your program ask a question and then choose what code to run.

You can think of it like this:

> "If this condition is true, do this."

And sometimes:

> "If this is false, do something else."

This is one of the most important ideas in programming because programs need to make decisions constantly.

Examples:

- If the player is alive, keep playing
- If the password is correct, let the user in
- If the number is bigger than 10, print a message
- If the user is not logged in, show the login screen

---

# Part 1: The basic `if` statement

## Example

```csharp
if (true)
{
    Console.WriteLine("This runs.");
}
```

### What this means

The condition inside the parentheses is:

```csharp
true
```

Since that is true, the code inside the braces runs.

---

# Part 2: Real example with a boolean

```csharp
bool isAlive = true;

if (isAlive)
{
    Console.WriteLine("The player is alive.");
}
```

### Breakdown

- `isAlive` is a boolean
- `if (isAlive)` asks: "Is `isAlive` true?"
- if yes, the code block runs

---

# Part 3: `if` with a comparison

```csharp
int age = 20;

if (age >= 18)
{
    Console.WriteLine("You are an adult.");
}
```

### What is happening

This condition:

```csharp
age >= 18
```

creates a boolean result.

Since `20 >= 18` is true, the message prints.

---

# Part 4: Conditions must become true or false

An `if` statement needs a condition that becomes a boolean.

That means the condition must end up as either:

- `true`
- `false`

Examples of valid conditions:

```csharp
x > 5
age == 18
isLoggedIn
!isDead
hasKey && doorUnlocked
```

All of those become true or false.

---

# Part 5: The braces

```csharp
if (x > 5)
{
    Console.WriteLine("Big number");
}
```

The braces `{ }` mark the block of code that belongs to the `if`.

If the condition is true, everything inside that block runs.

If the condition is false, the block is skipped.

---

# Part 6: If the condition is false

```csharp
int score = 40;

if (score >= 70)
{
    Console.WriteLine("You passed.");
}
```

Since `40 >= 70` is false, nothing inside the block runs.

That means this code prints nothing.

---

# Part 7: `if` and `else`

Sometimes you want one thing to happen if the condition is true, and a different thing if it is false.

That is what `else` is for.

## Example

```csharp
int score = 40;

if (score >= 70)
{
    Console.WriteLine("You passed.");
}
else
{
    Console.WriteLine("You failed.");
}
```

### What this means

- if the condition is true → first block runs
- otherwise → `else` block runs

Since `40 >= 70` is false, this prints:

```text
You failed.
```

---

# Part 8: `else` has no condition

Notice this:

```csharp
else
{
    Console.WriteLine("You failed.");
}
```

There is no condition after `else`.

That is because `else` means:

> "If the earlier `if` was false, do this."

So `else` is the fallback option.

---

# Part 9: `if`, `else if`, and `else`

Sometimes there are more than two possibilities.

That is where `else if` comes in.

## Example

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("Grade A");
}
else if (score >= 80)
{
    Console.WriteLine("Grade B");
}
else if (score >= 70)
{
    Console.WriteLine("Grade C");
}
else
{
    Console.WriteLine("Below C");
}
```

### What happens here

C# checks from top to bottom:

1. Is score at least 90?
2. If not, is score at least 80?
3. If not, is score at least 70?
4. If none matched, use `else`

Since `85 >= 80` is true, it prints:

```text
Grade B
```

---

# Part 10: Order matters

In `if / else if / else`, order is very important.

C# stops at the first true condition.

Look at this bad order:

```csharp
int score = 95;

if (score >= 70)
{
    Console.WriteLine("Passed");
}
else if (score >= 90)
{
    Console.WriteLine("Excellent");
}
```

This prints:

```text
Passed
```

Why?

Because `95 >= 70` is already true, so C# never reaches the `else if`.

So you usually put the more specific or stricter checks first.

---

# Part 11: Good ordering example

```csharp
int score = 95;

if (score >= 90)
{
    Console.WriteLine("Excellent");
}
else if (score >= 70)
{
    Console.WriteLine("Passed");
}
else
{
    Console.WriteLine("Failed");
}
```

Now the result makes more sense.

---

# Part 12: Nested `if` statements

A nested `if` means an `if` inside another `if`.

## Example

```csharp
bool isLoggedIn = true;
bool isAdmin = true;

if (isLoggedIn)
{
    Console.WriteLine("User is logged in.");

    if (isAdmin)
    {
        Console.WriteLine("User is also an admin.");
    }
}
```

### What happens

- first C# checks if the user is logged in
- only if that is true does it check the inner `if`

---

# Part 13: Using boolean operators in conditions

You can combine conditions with:

- `&&` = AND
- `||` = OR
- `!` = NOT

---

# Part 14: `&&` in `if`

```csharp
bool hasTicket = true;
bool hasId = true;

if (hasTicket && hasId)
{
    Console.WriteLine("You may enter.");
}
```

This means both conditions must be true.

---

# Part 15: `||` in `if`

```csharp
bool isAdmin = false;
bool isModerator = true;

if (isAdmin || isModerator)
{
    Console.WriteLine("You have staff access.");
}
```

This means at least one side must be true.

---

# Part 16: `!` in `if`

```csharp
bool isBanned = false;

if (!isBanned)
{
    Console.WriteLine("User is allowed in.");
}
```

`!isBanned` means:

> "NOT banned"

So if `isBanned` is false, `!isBanned` becomes true.

---

# Part 17: Common examples of `if`

## Number check

```csharp
int number = 12;

if (number > 10)
{
    Console.WriteLine("Number is bigger than 10.");
}
```

## String check

```csharp
string password = "cat";

if (password == "cat")
{
    Console.WriteLine("Correct password.");
}
```

## Character check

```csharp
char grade = 'A';

if (grade == 'A')
{
    Console.WriteLine("Top grade.");
}
```

---

# Part 18: `if` only runs once when reached

This is important.

An `if` statement is **not** a loop.

It does not repeat by itself.

It just checks once when the program reaches it.

Example:

```csharp
int x = 5;

if (x == 5)
{
    Console.WriteLine("x is 5");
}
```

This checks one time and moves on.

---

# Part 19: `if` vs loop

## `if`
Decision

> "Should I run this code right now?"

## Loop
Repetition

> "Should I keep running this code again and again?"

So:

- `if` = branching
- loop = iteration

Both are control flow tools, but they do different jobs.

---

# Part 20: Common beginner mistakes

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

## Mistake 2: Forgetting braces in bigger blocks

This is okay for one line:

```csharp
if (x > 5)
    Console.WriteLine("Big");
```

But beginners should usually use braces because it is clearer and safer:

```csharp
if (x > 5)
{
    Console.WriteLine("Big");
}
```

---

## Mistake 3: Bad ordering in `else if`

Put the more specific checks first.

Bad:

```csharp
if (score >= 70)
{
    Console.WriteLine("Passed");
}
else if (score >= 90)
{
    Console.WriteLine("Excellent");
}
```

Good:

```csharp
if (score >= 90)
{
    Console.WriteLine("Excellent");
}
else if (score >= 70)
{
    Console.WriteLine("Passed");
}
```

---

## Mistake 4: Overcomplicating boolean checks

This works:

```csharp
if (isReady == true)
```

But cleaner is:

```csharp
if (isReady)
```

This works:

```csharp
if (isReady == false)
```

But cleaner is:

```csharp
if (!isReady)
```

---

# Part 21: Real-world style examples

## Example: game health

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

## Example: door logic

```csharp
bool hasKey = true;
bool doorLocked = true;

if (hasKey && doorLocked)
{
    Console.WriteLine("You unlock the door.");
}
else
{
    Console.WriteLine("You cannot unlock the door.");
}
```

## Example: age category

```csharp
int age = 15;

if (age >= 18)
{
    Console.WriteLine("Adult");
}
else if (age >= 13)
{
    Console.WriteLine("Teen");
}
else
{
    Console.WriteLine("Child");
}
```

---

# Part 22: Very simple mental model

If you ever get lost, remember:

An `if` statement asks a yes-or-no question.

If the answer is yes, it runs a block.

If the answer is no, it skips it or goes to `else`.

---

# Part 23: Sample program that demonstrates `if`, `else if`, and `else`

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== BASIC IF ===");
        bool isAlive = true;

        if (isAlive)
        {
            Console.WriteLine("The player is alive.");
        }

        Console.WriteLine();
        Console.WriteLine("=== IF WITH COMPARISON ===");
        int age = 20;

        if (age >= 18)
        {
            Console.WriteLine("User is an adult.");
        }

        Console.WriteLine();
        Console.WriteLine("=== IF AND ELSE ===");
        int health = 0;

        if (health > 0)
        {
            Console.WriteLine("Player is alive.");
        }
        else
        {
            Console.WriteLine("Player is dead.");
        }

        Console.WriteLine();
        Console.WriteLine("=== IF, ELSE IF, ELSE ===");
        int score = 85;

        if (score >= 90)
        {
            Console.WriteLine("Grade A");
        }
        else if (score >= 80)
        {
            Console.WriteLine("Grade B");
        }
        else if (score >= 70)
        {
            Console.WriteLine("Grade C");
        }
        else
        {
            Console.WriteLine("Below C");
        }

        Console.WriteLine();
        Console.WriteLine("=== COMBINED CONDITIONS ===");
        bool hasTicket = true;
        bool hasId = true;

        if (hasTicket && hasId)
        {
            Console.WriteLine("You may enter the event.");
        }
        else
        {
            Console.WriteLine("You may not enter the event.");
        }

        Console.WriteLine();
        Console.WriteLine("=== OR CONDITION ===");
        bool isAdmin = false;
        bool isModerator = true;

        if (isAdmin || isModerator)
        {
            Console.WriteLine("User has staff access.");
        }
        else
        {
            Console.WriteLine("User does not have staff access.");
        }

        Console.WriteLine();
        Console.WriteLine("=== NOT CONDITION ===");
        bool isBanned = false;

        if (!isBanned)
        {
            Console.WriteLine("User is allowed in.");
        }
        else
        {
            Console.WriteLine("User is banned.");
        }

        Console.WriteLine();
        Console.WriteLine("=== NESTED IF ===");
        bool isLoggedIn = true;
        bool isAdminUser = true;

        if (isLoggedIn)
        {
            Console.WriteLine("User is logged in.");

            if (isAdminUser)
            {
                Console.WriteLine("User is also an admin.");
            }
            else
            {
                Console.WriteLine("User is not an admin.");
            }
        }
        else
        {
            Console.WriteLine("User is not logged in.");
        }

        Console.WriteLine();
        Console.WriteLine("=== STRING CHECK ===");
        string password = "cat";

        if (password == "cat")
        {
            Console.WriteLine("Correct password.");
        }
        else
        {
            Console.WriteLine("Wrong password.");
        }
    }
}
```

---

# Part 24: What the final program teaches

This one program shows:

- a basic `if`
- `if` with comparisons
- `if` and `else`
- `if`, `else if`, and `else`
- `&&`
- `||`
- `!`
- nested `if`
- number checks
- boolean checks
- string checks

---

# Part 25: Final recap

## `if`
Runs code if the condition is true.

## `else`
Runs if the earlier `if` was false.

## `else if`
Lets you check more conditions in order.

## Important rule
C# checks top to bottom and stops at the first true match.

## Big idea
`if` statements are for decision-making, not repetition.

---

# Part 26: One sentence memory hook

**Use `if` to make a decision, `else if` to test more possibilities, and `else` for the fallback case.**
