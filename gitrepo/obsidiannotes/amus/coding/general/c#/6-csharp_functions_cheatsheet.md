# C# Cheatsheet: Functions
_A simple, detailed, overexplained guide_

---

## What this cheatsheet is about

A **function** is a named block of code that does a job.

In C#, functions are usually called **methods** when they live inside a class.

People often use the words loosely like this:

- function
- method

For beginner learning, that is okay.

Main idea:

> A function is code you can reuse by calling its name.

Instead of rewriting the same logic again and again, you put that logic into a function and call it whenever needed.

Functions are used for things like:

- printing messages
- adding numbers
- checking conditions
- processing text
- moving game characters
- breaking big problems into smaller parts

---

# Part 1: Why functions exist

Imagine you want to print a welcome message many times.

Without a function, you might keep rewriting the same code.

That gets repetitive fast.

A function lets you define the code once, then reuse it.

That makes code:

- cleaner
- easier to read
- easier to fix
- easier to reuse

Computer science lore:
Functions are one of the biggest tools humans invented for fighting spaghetti code.

---

# Part 2: A simple function

```csharp
using System;

class Program
{
    static void SayHello()
    {
        Console.WriteLine("Hello!");
    }

    static void Main()
    {
        SayHello();
    }
}
```

## What happens

- `SayHello()` is the function
- `Main()` calls it
- when called, the code inside runs

Output:

```text
Hello!
```

---

# Part 3: Breaking down a function

Look at this:

```csharp
static void SayHello()
{
    Console.WriteLine("Hello!");
}
```

### Pieces

## `static`
For now, in beginner console programs, this often means:

> "This belongs to the class itself, so Main can call it directly."

## `void`
This means the function does **not return a value**.

It does something, but does not send a result back.

## `SayHello`
This is the function name.

## `()`
These parentheses hold inputs, also called **parameters**.

This one has none.

## `{ }`
This block contains the code the function runs.

---

# Part 4: Calling a function

To use a function, you **call** it by name.

```csharp
SayHello();
```

That means:

> "Run the code inside `SayHello` now."

---

# Part 5: A function can be called many times

```csharp
using System;

class Program
{
    static void SayHello()
    {
        Console.WriteLine("Hello!");
    }

    static void Main()
    {
        SayHello();
        SayHello();
        SayHello();
    }
}
```

Output:

```text
Hello!
Hello!
Hello!
```

This is why functions are useful.

You define once, use many times.

---

# Part 6: Functions with parameters

A parameter is an input the function receives.

## Example

```csharp
using System;

class Program
{
    static void SayName(string name)
    {
        Console.WriteLine("Hello, " + name);
    }

    static void Main()
    {
        SayName("Kai");
        SayName("Mira");
    }
}
```

Output:

```text
Hello, Kai
Hello, Mira
```

---

# Part 7: Breaking down parameters

```csharp
static void SayName(string name)
```

This means:

- the function takes one input
- that input must be a `string`
- inside the function, that input is called `name`

So when you call:

```csharp
SayName("Kai");
```

the string `"Kai"` gets placed into `name`.

---

# Part 8: Multiple parameters

Functions can take more than one input.

```csharp
using System;

class Program
{
    static void PrintFullName(string firstName, string lastName)
    {
        Console.WriteLine(firstName + " " + lastName);
    }

    static void Main()
    {
        PrintFullName("Kai", "Stone");
    }
}
```

Output:

```text
Kai Stone
```

---

# Part 9: Parameter order matters

This matters a lot:

```csharp
PrintFullName("Kai", "Stone");
```

The first input goes into `firstName`.  
The second input goes into `lastName`.

If you swap them:

```csharp
PrintFullName("Stone", "Kai");
```

then the result changes.

So argument order matters.

---

# Part 10: What is a return value?

Some functions do not just do something.

Some functions calculate something and **return** the answer.

Example:

```csharp
static int Add(int a, int b)
{
    return a + b;
}
```

This function returns an `int`.

---

# Part 11: `void` vs return type

## `void`
Means no value comes back.

Example:

```csharp
static void SayHello()
{
    Console.WriteLine("Hello!");
}
```

## `int`
Means the function must return an integer.

Example:

```csharp
static int Add(int a, int b)
{
    return a + b;
}
```

Other return types can be:

- `string`
- `bool`
- `double`
- `char`

and many more.

---

# Part 12: Using a returned value

```csharp
using System;

class Program
{
    static int Add(int a, int b)
    {
        return a + b;
    }

    static void Main()
    {
        int result = Add(3, 4);
        Console.WriteLine(result);
    }
}
```

Output:

```text
7
```

### What happened

- `Add(3, 4)` returns `7`
- that `7` gets stored in `result`

---

# Part 13: Returning directly into output

You do not always need a variable.

```csharp
Console.WriteLine(Add(10, 20));
```

That works because the function returns a value.

---

# Part 14: Returning strings

```csharp
static string GetGreeting(string name)
{
    return "Hello, " + name;
}
```

Use it like this:

```csharp
string message = GetGreeting("Kai");
Console.WriteLine(message);
```

---

# Part 15: Returning booleans

Functions can also return true/false answers.

```csharp
static bool IsEven(int number)
{
    return number % 2 == 0;
}
```

Use it like this:

```csharp
bool result = IsEven(4);
Console.WriteLine(result);
```

Output:

```text
True
```

---

# Part 16: A function can help organize your program

Instead of putting everything inside `Main`, you can split jobs into functions.

Example:

```csharp
using System;

class Program
{
    static void ShowTitle()
    {
        Console.WriteLine("=== My Program ===");
    }

    static int Add(int a, int b)
    {
        return a + b;
    }

    static void Main()
    {
        ShowTitle();
        int total = Add(5, 7);
        Console.WriteLine(total);
    }
}
```

This is much cleaner than shoving everything into one giant block.

---

# Part 17: Functions help break big problems into small parts

Example problem:
Make a tiny game system.

Possible functions:

- `ShowMenu()`
- `CreatePlayer()`
- `AttackEnemy()`
- `HealPlayer()`
- `CheckWin()`

That is how functions help structure large code.

---

# Part 18: Function names should describe actions

Functions usually do something, so their names should sound like actions.

Good names:

- `PrintScore()`
- `AddNumbers()`
- `GetGreeting()`
- `IsEven()`
- `ShowMenu()`

Less good names:

- `Thing()`
- `DoStuff()`
- `X()`

Good names are part of good C# etiquette.

---

# Part 19: Parameters vs arguments

These words are related but slightly different.

## Parameter
The variable listed in the function definition.

```csharp
static void SayName(string name)
```

Here, `name` is a parameter.

## Argument
The actual value you pass in during the call.

```csharp
SayName("Kai");
```

Here, `"Kai"` is the argument.

A lot of beginners mix these up, which is normal.

---

# Part 20: Local variables inside functions

Variables created inside a function usually belong only to that function.

```csharp
static void ShowNumber()
{
    int x = 10;
    Console.WriteLine(x);
}
```

Here, `x` only exists inside `ShowNumber()`.

This is called **scope**.

---

# Part 21: Scope

**Scope** means where a variable can be used.

Example:

```csharp
static void Test()
{
    int x = 5;
    Console.WriteLine(x);
}
```

You can use `x` inside `Test()`, but not outside it.

That is function scope.

Computer science lore:
Scope is one of the walls that keeps variables from running around the whole program like feral goblins.

---

# Part 22: A function can call another function

```csharp
using System;

class Program
{
    static void SayHello()
    {
        Console.WriteLine("Hello!");
    }

    static void StartProgram()
    {
        SayHello();
        Console.WriteLine("Program started.");
    }

    static void Main()
    {
        StartProgram();
    }
}
```

Functions can work together.

That is very common.

---

# Part 23: Functions with calculations

```csharp
static int Square(int number)
{
    return number * number;
}
```

Use it:

```csharp
int answer = Square(6);
Console.WriteLine(answer);
```

Output:

```text
36
```

---

# Part 24: Functions with conditions

```csharp
static bool IsAdult(int age)
{
    if (age >= 18)
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

This works, but can be shorter:

```csharp
static bool IsAdult(int age)
{
    return age >= 18;
}
```

Both versions do the same thing.

---

# Part 25: Early return

Sometimes a function returns early when a condition is met.

```csharp
static string GetGrade(int score)
{
    if (score >= 90)
    {
        return "A";
    }

    if (score >= 80)
    {
        return "B";
    }

    return "Below B";
}
```

As soon as `return` happens, the function stops.

That is very important.

---

# Part 26: Why `return` matters

`return` does two jobs:

1. sends a value back
2. ends the function immediately

So this:

```csharp
return 5;
```

means:

- give back `5`
- stop the function right now

---

# Part 27: Common beginner mistakes

## Mistake 1: Forgetting parentheses when calling

Wrong:

```csharp
SayHello;
```

Right:

```csharp
SayHello();
```

---

## Mistake 2: Returning nothing from a non-void function

Wrong:

```csharp
static int Add(int a, int b)
{
}
```

If the return type is `int`, the function must return an int.

Correct:

```csharp
static int Add(int a, int b)
{
    return a + b;
}
```

---

## Mistake 3: Trying to store a void result

Wrong idea:

```csharp
int x = SayHello();
```

That fails because `SayHello()` returns nothing.

---

## Mistake 4: Passing the wrong type

If the function expects an int:

```csharp
static int Square(int number)
```

then this is okay:

```csharp
Square(5);
```

but this is wrong:

```csharp
Square("hello");
```

because `"hello"` is a string, not an int.

---

# Part 28: Functions can make loops cleaner

Instead of doing everything in one big loop, you can move work into functions.

Example:

```csharp
static void PrintNumber(int number)
{
    Console.WriteLine("Number: " + number);
}
```

Then:

```csharp
for (int i = 0; i < 5; i++)
{
    PrintNumber(i);
}
```

This makes code easier to read.

---

# Part 29: Functions can process strings

```csharp
static string MakeLoud(string text)
{
    return text.ToUpper();
}
```

Use it:

```csharp
Console.WriteLine(MakeLoud("hello"));
```

Output:

```text
HELLO
```

---

# Part 30: Functions can process arrays

```csharp
static int SumArray(int[] numbers)
{
    int total = 0;

    for (int i = 0; i < numbers.Length; i++)
    {
        total += numbers[i];
    }

    return total;
}
```

Use it:

```csharp
int[] nums = { 1, 2, 3, 4 };
Console.WriteLine(SumArray(nums));
```

Output:

```text
10
```

---

# Part 31: A common beginner structure

A lot of beginner console apps follow this style:

```csharp
using System;

class Program
{
    static void ShowTitle()
    {
        Console.WriteLine("=== App ===");
    }

    static int Multiply(int a, int b)
    {
        return a * b;
    }

    static void Main()
    {
        ShowTitle();
        int result = Multiply(3, 4);
        Console.WriteLine(result);
    }
}
```

This is a healthy structure because:

- `Main` stays small
- logic is broken into functions
- each function has a clear job

---

# Part 32: Function design trick

Ask two questions:

## What job does this code do?
That might become a function.

## Will I need this again?
If yes, that is another clue it should become a function.

This is one of the most useful beginner design habits.

---

# Part 33: Quick recap of the big ideas

## Function
Reusable named code.

## Call
Using the function.

## Parameter
Input variable in the function definition.

## Argument
Actual value passed during the call.

## Return value
The result sent back by the function.

## `void`
No value returned.

## Scope
Where a variable exists and can be used.

---

# Part 34: Final sample program that demonstrates functions

```csharp
using System;

class Program
{
    static void ShowTitle()
    {
        Console.WriteLine("=== Function Demo Program ===");
    }

    static void SayHello(string name)
    {
        Console.WriteLine("Hello, " + name + "!");
    }

    static int Add(int a, int b)
    {
        return a + b;
    }

    static int Square(int number)
    {
        return number * number;
    }

    static bool IsEven(int number)
    {
        return number % 2 == 0;
    }

    static string MakeLoud(string text)
    {
        return text.ToUpper();
    }

    static int SumArray(int[] numbers)
    {
        int total = 0;

        for (int i = 0; i < numbers.Length; i++)
        {
            total += numbers[i];
        }

        return total;
    }

    static void Main()
    {
        ShowTitle();

        Console.WriteLine();
        Console.WriteLine("=== VOID FUNCTION ===");
        SayHello("Kai");
        SayHello("Mira");

        Console.WriteLine();
        Console.WriteLine("=== FUNCTION RETURNING INT ===");
        int total = Add(5, 7);
        Console.WriteLine("5 + 7 = " + total);

        Console.WriteLine();
        Console.WriteLine("=== FUNCTION RETURNING ANOTHER INT ===");
        int squared = Square(6);
        Console.WriteLine("6 squared = " + squared);

        Console.WriteLine();
        Console.WriteLine("=== FUNCTION RETURNING BOOL ===");
        bool evenResult = IsEven(8);
        Console.WriteLine("Is 8 even? " + evenResult);

        Console.WriteLine();
        Console.WriteLine("=== FUNCTION RETURNING STRING ===");
        string loudText = MakeLoud("hello world");
        Console.WriteLine(loudText);

        Console.WriteLine();
        Console.WriteLine("=== FUNCTION USING AN ARRAY ===");
        int[] nums = { 10, 20, 30, 40 };
        int arrayTotal = SumArray(nums);
        Console.WriteLine("Array total = " + arrayTotal);

        Console.WriteLine();
        Console.WriteLine("=== CALLING A FUNCTION INSIDE WRITELINE ===");
        Console.WriteLine("3 + 4 = " + Add(3, 4));
        Console.WriteLine("Is 5 even? " + IsEven(5));
    }
}
```

---

# Part 35: What the final program teaches

This one program shows:

- a `void` function
- functions with parameters
- functions with multiple parameters
- functions returning `int`
- functions returning `bool`
- functions returning `string`
- calling functions many times
- functions using arrays
- loops inside functions
- cleaner program structure

---

# Part 36: One sentence memory hook

**A function is reusable named code that can take inputs, do work, and optionally return a result.**
