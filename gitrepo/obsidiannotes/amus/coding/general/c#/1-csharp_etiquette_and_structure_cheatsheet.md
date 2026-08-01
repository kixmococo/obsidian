# C# Quick Cheatsheet: Etiquette and How Programs Are Usually Made
_A simple guide to common C# style, habits, and structure_

---

## What this cheatsheet is about

This is a quick guide to:

- common C# etiquette
- how C# code is usually organized
- how small and medium C# programs are commonly built
- habits that make code easier to read and maintain

This is less about strict syntax rules and more about:

> "What does normal, clean, respectable C# usually look like?"

---

# Part 1: Name things clearly

Good code names should explain what they are.

## Good

```csharp
int playerHealth = 100;
string firstName = "Kai";
bool isAlive = true;
```

## Bad

```csharp
int x = 100;
string s = "Kai";
bool thing = true;
```

## Rule

- variables should describe the data
- methods should describe the action
- classes should describe the thing

This matters because code is read far more often than it is written.

---

# Part 2: Common C# naming style

C# has common naming conventions that many developers follow.

## Classes and methods: PascalCase

Use **PascalCase** for:

- class names
- method names
- property names

```csharp
class PlayerCharacter
{
    public int Health { get; set; }

    public void AttackEnemy()
    {
    }
}
```

PascalCase means each word starts with a capital letter.

Examples:

- `Player`
- `EnemyManager`
- `AttackEnemy`
- `ShowStatus`

---

## Variables and parameters: camelCase

Use **camelCase** for:

- local variables
- method parameters

```csharp
int playerHealth = 100;
string userName = "u1";

static void PrintMessage(string messageText)
{
    Console.WriteLine(messageText);
}
```

camelCase means:

- first word starts lowercase
- later words start uppercase

Examples:

- `playerHealth`
- `userName`
- `damageAmount`

---

## Private fields: often `_camelCase`

A common C# style for private fields is:

```csharp
private int _health;
private string _name;
```

This is common because it makes fields easy to recognize.

Not every codebase uses this, but many do.

---

# Part 3: Use braces even when optional

C# sometimes allows short code like this:

```csharp
if (isAlive)
    Console.WriteLine("Alive");
```

But cleaner style is usually this:

```csharp
if (isAlive)
{
    Console.WriteLine("Alive");
}
```

## Why this is better

- easier to read
- easier to expand later
- safer when editing
- reduces bugs caused by missing braces

Good etiquette usually means using braces consistently.

---

# Part 4: Indent cleanly

Most C# code uses 4 spaces per block level.

## Good

```csharp
if (isAlive)
{
    Console.WriteLine("Alive");
}
```

## Bad

```csharp
if (isAlive){
Console.WriteLine("Alive");
}
```

Indentation helps other humans understand the structure of your code instantly.

Programming lore:
bad indentation is one of the oldest ways humans have made code harder than it needed to be.

---

# Part 5: Keep methods focused

A method should usually do one main job.

## Good

```csharp
static int Add(int a, int b)
{
    return a + b;
}
```

This method has one clear purpose.

## Bad idea

A single method that:
- reads files
- prints menus
- attacks enemies
- validates passwords
- saves data
- logs errors

That becomes a chaos pile fast.

## Good habit

Ask:

> "What is this method's one job?"

This connects to the **single responsibility** idea in software design.

---

# Part 6: Comment wisely

Comments should explain things that are not obvious.

## Bad comment

```csharp
int age = 5; // set age to 5
```

That comment adds no value.

## Better comment

```csharp
// We subtract 1 because array indexes start at 0.
int lastIndex = names.Length - 1;
```

Good comments often explain:

- why something exists
- why something is weird
- why a choice was made
- why a workaround is needed

---

# Part 7: Avoid magic numbers

A magic number is a raw number in code with no explanation.

## Less clear

```csharp
if (health < 25)
{
    Console.WriteLine("Low health");
}
```

Why 25? The reader has to guess.

## Better

```csharp
const int LowHealthThreshold = 25;

if (health < LowHealthThreshold)
{
    Console.WriteLine("Low health");
}
```

Now the meaning is clear.

This also makes later changes easier.

---

# Part 8: Prefer readable code over clever code

Readable code wins most of the time.

## Harder to read

```csharp
Console.WriteLine(x > 0 ? y < 3 ? "A" : "B" : "C");
```

## Easier to read

```csharp
if (x > 0)
{
    if (y < 3)
    {
        Console.WriteLine("A");
    }
    else
    {
        Console.WriteLine("B");
    }
}
else
{
    Console.WriteLine("C");
}
```

Especially while learning, clear beats fancy.

---

# Part 9: Basic shape of a simple C# program

A beginner C# console app often looks like this:

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello, world!");
    }
}
```

## Breakdown

- `using System;` gives access to basic built-in tools
- `class Program` defines a class
- `static void Main()` is the entry point
- the program starts in `Main`

---

# Part 10: How programs are usually built

A lot of C# programs are built in a general flow like this:

## Step 1: Decide the purpose

Examples:

- calculator
- text parser
- file tool
- game
- web API
- desktop app

## Step 2: Break the problem into parts

Example for a game:

- player
- enemy
- combat
- inventory
- saving
- UI

## Step 3: Turn important nouns into classes

Examples:

```csharp
class Player
class Enemy
class Item
class Inventory
```

## Step 4: Turn important verbs into methods

Examples:

```csharp
Attack()
Heal()
Move()
Save()
Load()
```

## Step 5: Connect everything together

Usually in:

- `Main`
- a game loop
- a manager class
- a controller class
- a service layer

This is a very common design habit.

---

# Part 11: Common beginner structure

Small console apps often follow this style:

```csharp
using System;

class Program
{
    static void Main()
    {
        ShowIntro();
        int result = AddNumbers(3, 4);
        Console.WriteLine(result);
    }

    static void ShowIntro()
    {
        Console.WriteLine("Welcome.");
    }

    static int AddNumbers(int a, int b)
    {
        return a + b;
    }
}
```

## Why this is good

- `Main` stays simple
- logic is split into small methods
- easier to read
- easier to debug

---

# Part 12: As projects grow, code gets split into files

Tiny programs may live in one file.

Normal projects usually split code into multiple files.

## Example layout

```text
Program.cs
Player.cs
Enemy.cs
Inventory.cs
Game.cs
```

## Common habit

- one major class per file
- file name matches class name

Examples:

- `Player.cs` contains `class Player`
- `Enemy.cs` contains `class Enemy`

This is normal C# etiquette.

---

# Part 13: Programs are often object-based

C# strongly encourages object-oriented structure.

That means programs often revolve around:

- classes
- objects
- properties
- methods

## Example

```csharp
class Player
{
    public string Name { get; set; }
    public int Health { get; set; }

    public void Attack()
    {
        Console.WriteLine(Name + " attacks.");
    }
}
```

Then:

```csharp
Player hero = new Player();
hero.Name = "Kai";
hero.Health = 100;
hero.Attack();
```

This is a very common C# pattern.

---

# Part 14: Programs are usually built in layers

A good program often separates responsibilities.

Examples of rough layers:

- input
- logic
- data
- output / UI

## Simple meaning

- input gets information
- logic decides what happens
- data stores state
- output shows results

This separation keeps code from becoming one giant tangled knot.

---

# Part 15: Common kinds of C# projects

## Console app

Runs in terminal.

Good for:

- learning
- utilities
- scripts
- tools
- practice

## Class library

Reusable code used by other programs.

## Web app / API

Often built with ASP.NET Core.

## Desktop app

Often built with WinForms, WPF, or MAUI.

## Game

Often built with Unity or custom tools.

Different project types, same main ideas:

- classes
- methods
- logic
- data
- control flow

---

# Part 16: A common C# mindset

A lot of C# code is written with this mindset:

- define the data
- define the behavior
- group related behavior into classes
- keep methods small
- name things clearly
- separate concerns
- keep the structure predictable

That is a big part of normal C# culture.

---

# Part 17: Good beginner habits

## Do these

- use clear names
- use consistent formatting
- keep methods small
- break problems into pieces
- test often
- fix warnings early
- make code readable
- organize classes cleanly

## Avoid these

- giant `Main` methods
- giant classes that do everything
- random variable names
- inconsistent formatting
- copy-paste chaos
- deeply nested spaghetti logic

---

# Part 18: Example of a more normal small C# program

```csharp
using System;

class Player
{
    public string Name { get; set; }
    public int Health { get; set; }

    public Player(string name, int health)
    {
        Name = name;
        Health = health;
    }

    public void ShowStatus()
    {
        Console.WriteLine(Name + " has " + Health + " health.");
    }

    public void TakeDamage(int damage)
    {
        Health -= damage;

        if (Health < 0)
        {
            Health = 0;
        }
    }
}

class Program
{
    static void Main()
    {
        Player hero = new Player("Kai", 100);

        hero.ShowStatus();
        hero.TakeDamage(25);
        hero.ShowStatus();
    }
}
```

## Why this looks normal

- there is a class for the thing
- there are methods for actions
- the object stores its own data
- `Main` just drives the program
- logic lives with the object it belongs to

---

# Part 19: A practical design trick

A very common way to design programs is this:

## Ask: what are the nouns?

Those often become classes.

Examples:

- Player
- Enemy
- User
- Order
- FileReader

## Ask: what are the verbs?

Those often become methods.

Examples:

- Attack()
- Save()
- Load()
- Print()
- Validate()

This is a very useful beginner design habit.

---

# Part 20: General etiquette recap

## Naming
- PascalCase for classes, methods, and properties
- camelCase for local variables and parameters
- often `_camelCase` for private fields

## Style
- use braces
- indent cleanly
- keep formatting consistent

## Design
- keep methods focused
- split code into files
- keep classes from doing too much
- prefer readable code
- avoid unexplained numbers
- separate responsibilities

---

# Part 21: One sentence memory hook

**Good C# code is usually organized, readable, object-based, and broken into small clear parts.**
