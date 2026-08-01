# C#: Beginner → Novice Guide
*A copilot's field manual — from "what's the SDK" to writing real programs (and yes, Godot).*

---

## Table of Contents
1. [Getting C# Running (Linux, Mac, Windows)](#1-getting-c-running)
2. [Your Toolkit (Editor, REPL, Project System)](#2-your-toolkit)
3. [Core Language Concepts, Mapped to C#](#3-core-language-concepts)
4. [Data Structures & Collections](#4-data-structures--collections)
5. [Methods & Scope](#5-methods--scope)
6. [Object-Oriented Programming](#6-object-oriented-programming)
7. [Error Handling](#7-error-handling)
8. [Files, I/O, and Working with Data](#8-files-io-and-working-with-data)
9. [Namespaces, Assemblies & the Standard Library](#9-namespaces-assemblies--the-standard-library)
10. [Intermediate Concepts (your "novice" ramp)](#10-intermediate-concepts)
11. [C# in Godot](#11-c-in-godot)
12. [Practice Apps & Where to Drill](#12-practice-apps--where-to-drill)
13. [Tutorials, Docs & Communities](#13-tutorials-docs--communities)
14. [Practical Projects (build these)](#14-practical-projects)
15. [Cheat Sheet](#15-cheat-sheet)

---

## 1. Getting C# Running

C# is compiled and runs on the **.NET runtime** (current stable: **.NET 10**, an LTS release, paired with **C# 14**). Unlike Python, you don't need a separate interpreter install per OS — the .NET SDK gives you compiler, runtime, and package manager in one.

### Linux
```bash
# Ubuntu/Debian — Microsoft's official feed
wget https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt update
sudo apt install -y dotnet-sdk-10.0
```
```bash
# Fedora
sudo dnf install dotnet-sdk-10.0
```
```bash
# Arch (AUR)
yay -S dotnet-sdk
```
Or use the official install script (works on any distro):
```bash
curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel 10.0
```

### macOS
```bash
brew install --cask dotnet-sdk
```

### Windows
Download the installer from dotnet.microsoft.com, or:
```powershell
winget install Microsoft.DotNet.SDK.10
```

### Verify it works
```bash
dotnet --version
dotnet --list-sdks
```

---

## 2. Your Toolkit

- **Editor:** VS Code with the **C# Dev Kit** extension is the lightweight cross-platform option (and matches the workflow you already use for JS/Godot). **Visual Studio** (Windows only, free Community edition) is the heavyweight, batteries-included IDE if you're on Windows. **JetBrains Rider** is a strong cross-platform paid alternative.
- **REPL:** `dotnet-script` or the built-in **C# Interactive** window in Visual Studio let you test snippets without a full project. Quickest cross-platform option:
```bash
dotnet tool install -g dotnet-script
dotnet script
```
- **The project system** — C# organizes code into projects (`.csproj`) inside solutions (`.sln`), not loose scripts:
```bash
dotnet new console -o MyApp     # scaffold a new console project
cd MyApp
dotnet run                       # build + run
dotnet build                     # build only
```
- **NuGet** — the package manager (pip's equivalent):
```bash
dotnet add package Newtonsoft.Json
dotnet restore                    # pull down everything listed in the .csproj
```

---

## 3. Core Language Concepts

### Variables & types
C# is **statically typed** — the compiler checks types before your code ever runs, unlike Python.
```csharp
string name = "u1";
int age = 30;
double height = 1.8;
bool isDev = true;
object nothing = null;

var inferred = "still a string";   // var lets the compiler infer the type
```

### Operators
```csharp
7 / 2;        // 3 — integer division truncates
7 % 2;        // 1 — modulo
Math.Pow(2, 8); // 256 — no ** operator, use Math.Pow
(a, b) = (b, a); // tuple swap
```

### Control flow
```csharp
if (age >= 18)
{
    Console.WriteLine("adult");
}
else if (age >= 13)
{
    Console.WriteLine("teen");
}
else
{
    Console.WriteLine("kid");
}

for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);
}

int n = 5;
while (n > 0)
{
    n--;
}

foreach (var item in someList)
{
    Console.WriteLine(item);
}
```
Modern pattern matching (`switch` expressions, C# 8+):
```csharp
string result = command switch
{
    "move" => "moving",
    "attack" or "defend" => "combat",
    _ => "unknown"
};
```

### Braces & semicolons
Unlike Python's whitespace rules, C# uses `{ }` to define blocks and `;` to end statements — indentation is just for readability, the compiler doesn't care about it.

---

## 4. Data Structures & Collections

| Type | Ordered | Mutable | Example |
|---|---|---|---|
| `List<T>` | yes | yes | `new List<int> { 1, 2, 3 }` |
| `T[]` (array) | yes | fixed size | `new int[] { 1, 2, 3 }` |
| `Dictionary<K,V>` | insertion order (not guaranteed) | yes | `new Dictionary<string,int>()` |
| `HashSet<T>` | no | yes | `new HashSet<int> { 1, 2, 3 }` |
| `Tuple` / `(T1,T2)` | yes | no | `(1, "a")` |

```csharp
var scores = new List<int> { 10, 20, 30 };
scores.Add(40);
scores[0];              // 10
scores[^1];              // 40 — index from end (C# 8+)
scores.GetRange(1, 2);    // [20, 30]

var player = new Dictionary<string, object>
{
    ["name"] = "u1",
    ["level"] = 12
};
player["level"] = (int)player["level"] + 1;
player.TryGetValue("mana", out var mana); // safe lookup

// LINQ — C#'s answer to comprehensions
var squares = Enumerable.Range(0, 10).Select(x => x * x).ToList();
var evens = Enumerable.Range(0, 20).Where(x => x % 2 == 0).ToList();
var lookup = Enumerable.Range(0, 5).ToDictionary(x => x, x => x * x);
```

---

## 5. Methods & Scope

```csharp
static string Greet(string name, string greeting = "Hello")
{
    return $"{greeting}, {name}!";
}

Greet("u1");                             // positional
Greet(name: "u1", greeting: "Hey");        // named argument

static int Total(params int[] numbers)
{
    return numbers.Sum();
}

Func<int, int> square = x => x * x;   // lambda expression
```
Scope: variables declared inside a method are local to it. C# has no `global`/`nonlocal` equivalent needed — fields on a class are the usual way to share state, accessed through `this`.

---

## 6. Object-Oriented Programming

C# is OOP by design far more than Python is — everything lives inside a class or struct.

```csharp
public class Character
{
    public static string Species = "human";  // shared across instances

    public string Name { get; set; }          // auto-property
    public int Hp { get; private set; }        // settable only inside the class

    public Character(string name, int hp = 100)
    {
        Name = name;
        Hp = hp;
    }

    public int TakeDamage(int amount)
    {
        Hp -= amount;
        return Hp;
    }

    public override string ToString() => $"Character({Name}, hp={Hp})";
}

public class Mage : Character           // inheritance
{
    public int Mana { get; private set; }

    public Mage(string name, int mana = 50) : base(name)
    {
        Mana = mana;
    }

    public string Cast(string spell)
    {
        Mana -= 10;
        return $"{Name} casts {spell}";
    }
}
```
Key ideas: **properties** (`get`/`set`) replace bare public fields as the idiomatic way to expose state, **interfaces** (`interface IDamageable { void TakeDamage(int amount); }`) define contracts without inheritance, `override`/`virtual` control polymorphism explicitly (unlike Python, where any method can be overridden freely).

---

## 7. Error Handling

```csharp
try
{
    int value = int.Parse(Console.ReadLine());
}
catch (FormatException)
{
    Console.WriteLine("That wasn't a number.");
}
catch (Exception e) when (e is ArgumentException or InvalidOperationException)
{
    Console.WriteLine($"Something else went wrong: {e.Message}");
}
finally
{
    Console.WriteLine("Always runs — cleanup goes here");
}

// custom exceptions
public class InsufficientManaException : Exception
{
    public InsufficientManaException(string message) : base(message) { }
}

if (mana < cost)
{
    throw new InsufficientManaException("Not enough mana to cast that");
}
```
Same rule as any language: catch specific exception types, don't blanket-catch `Exception` unless you're at a true top-level boundary.

---

## 8. Files, I/O, and Working with Data

```csharp
File.WriteAllText("save.txt", "level=12\n");

foreach (var line in File.ReadLines("save.txt"))
{
    Console.WriteLine(line.Trim());
}

using (var writer = new StreamWriter("save.txt"))
{
    writer.WriteLine("level=12");
} // Dispose() called automatically at the closing brace
```
`using` is C#'s context-manager equivalent — anything implementing `IDisposable` gets cleaned up automatically, the direct parallel to Python's `with`.

```csharp
using System.Text.Json;

var data = new Dictionary<string, object> { ["name"] = "u1", ["level"] = 12 };
File.WriteAllText("save.json", JsonSerializer.Serialize(data));

var loaded = JsonSerializer.Deserialize<Dictionary<string, object>>(
    File.ReadAllText("save.json"));
```

```csharp
using System.Globalization;
using CsvHelper; // NuGet package

using var reader = new StreamReader("scores.csv");
using var csv = new CsvHelper.CsvReader(reader, CultureInfo.InvariantCulture);
var records = csv.GetRecords<dynamic>();
```

---

## 9. Namespaces, Assemblies & the Standard Library

```csharp
// MyHelpers.cs
namespace MyApp.Helpers
{
    public static class Helper
    {
        public static string DoThing() => "I'm reusable";
    }
}

// Program.cs
using MyApp.Helpers;
Helper.DoThing();
```
A compiled project produces an **assembly** (`.dll`/`.exe`) — C#'s rough equivalent of a Python package, but pre-compiled rather than source-distributed.

**Base Class Library (BCL) worth knowing early:**
- `System.IO` — files and paths
- `System` — core types, `Console`, `Math`
- `System.Collections.Generic` — `List<T>`, `Dictionary<K,V>`, `Queue<T>`, `Stack<T>`
- `System.Linq` — query/transform collections declaratively
- `System.Text.Json` — JSON serialization
- `System.Threading.Tasks` — async/parallel work
- `System.Text.RegularExpressions` — regex

**Third-party (via NuGet), popular ones:**
- `Newtonsoft.Json` — the long-standing JSON library, predates `System.Text.Json`
- `Dapper` / `Entity Framework Core` — database access
- `xUnit` / `NUnit` — testing
- `Serilog` — structured logging
- `Godot.NET.Sdk` — you already have this one through Godot's C# support

---

## 10. Intermediate Concepts

**Async/await** — C#'s answer to non-blocking work, used constantly in real apps:
```csharp
public async Task<string> FetchDataAsync()
{
    using var client = new HttpClient();
    return await client.GetStringAsync("https://api.example.com/data");
}
```

**LINQ** — declarative queries over any collection, the single most distinctive C# feature:
```csharp
var topScores = players
    .Where(p => p.Level > 10)
    .OrderByDescending(p => p.Score)
    .Take(5)
    .ToList();
```

**Generics** — write one method/class that works across types safely:
```csharp
public T FindMax<T>(List<T> items) where T : IComparable<T>
{
    return items.Max();
}
```

**Nullable reference types** (on by default in modern projects) — the compiler warns you when something might be `null`:
```csharp
string? maybeNull = GetName();   // ? marks it as nullable
string safe = maybeNull ?? "default";  // null-coalescing
```

**Records** — concise immutable data types (C# 9+), handy for simple data-holding classes:
```csharp
public record Point(int X, int Y);
var p1 = new Point(1, 2);
var p2 = p1 with { X = 5 };  // copy with one field changed
```

**Events & delegates** — C#'s built-in observer pattern:
```csharp
public class Character
{
    public event Action<int>? OnDamaged;

    public void TakeDamage(int amount)
    {
        OnDamaged?.Invoke(amount);
    }
}
```

**Testing** — the habit that turns hobby code into real software:
```csharp
// using xUnit
public class MathUtilsTests
{
    [Fact]
    public void Add_ReturnsSum()
    {
        Assert.Equal(5, MathUtils.Add(2, 3));
    }
}
```
Run with `dotnet test`.

---

## 11. C# in Godot

Since you're already building the Godot chess-boss-fight project, a few things that matter specifically there:

- Godot uses C# via **Godot.NET.Sdk** — attach a script by inheriting from a Godot node type: `public partial class Player : CharacterBody2D`.
- **`[Export]`** exposes a field/property to the Godot editor's Inspector, the C# equivalent of GDScript's `@export`:
```csharp
public partial class Player : CharacterBody2D
{
    [Export] public float Speed = 300f;

    public override void _Ready() { }
    public override void _PhysicsProcess(double delta) { }
}
```
- Signals map to C# **events**; `[Signal]` delegate declarations let Godot's signal system talk to native C# event handlers.
- Godot's own collection/vector types (`Vector2`, `Vector3`, `Godot.Collections.Array`) sit alongside the BCL ones — mixing `System.Collections.Generic.List<T>` and `Godot.Collections.Array` is common and fine, just know which one a given Godot API expects.
- Because Godot is also scriptable in GDScript, C# in Godot is a genuinely good way to feel the static-vs-dynamic-typing tradeoff directly, node for node, against code you've likely already written.

---

## 12. Practice Apps & Where to Drill

- **Exercism.org** — has a solid C# track with mentored feedback
- **Codewars** — short problems, C# is a supported language
- **Advent of Code** — yearly puzzle set, works great in a `dotnet script` or console-app loop
- **LeetCode / HackerRank** — algorithm drilling, C# supported
- **Project Euler** — math-flavored problems

---

## 13. Tutorials, Docs & Communities

- **Official docs:** https://learn.microsoft.com/en-us/dotnet/csharp/ — Microsoft's official tour and reference
- **.NET downloads:** https://dotnet.microsoft.com/download
- **C# language tour:** https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/
- **Godot C# docs:** https://docs.godotengine.org/en/stable/tutorials/scripting/c_sharp/index.html — directly relevant to your current project
- **/r/csharp** and **/r/dotnet** — active, beginner-tolerant subreddits
- **C# Discord communities** — searchable, several active servers
- Microsoft's official **coding conventions** doc — worth a skim once past total basics, since C# style (PascalCase for methods/properties, camelCase for locals) differs meaningfully from Python's PEP 8

---

## 14. Practical Projects

Ordered roughly beginner → novice:
1. **Console to-do list** — practices collections, control flow, file I/O
2. **Number guessing game** — practices loops, `Random`, conditionals
3. **JSON-backed contact book** — practices `System.Text.Json`, classes, collections
4. **Simple text adventure** (classes for `Room`/`Player`/`Item`) — practices OOP, directly transferable to your Godot narrative-game instincts
5. **A small Godot scene scripted entirely in C#** — practices `[Export]`, signals/events, `_Process`/`_PhysicsProcess` — a direct bridge from this guide into your actual project
6. **Console app calling a public HTTP API** (`HttpClient` + `System.Text.Json`) — practices async/await, error handling, real-world messiness
7. **Minimal ASP.NET Core API with an in-memory "database"** — practices project structure, testing, dependency injection basics

---

## 15. Cheat Sheet

```csharp
// strings
s.Trim(); s.Split(','); s.ToLower(); s.Replace("a","b"); string.Join("-", list);

// lists
list.Sort(); list.OrderBy(x => x); list.Reverse(); list.Count; list.Contains(x);

// dictionaries
dict.Keys; dict.Values; dict.TryGetValue(k, out var v);

// common LINQ patterns
list.Where(x => cond).ToList();          // filter (comprehension equivalent)
list.Select((x, i) => (x, i));            // index + value, like enumerate
list1.Zip(list2, (a, b) => (a, b));        // pair up two lists
list.OrderByDescending(x => x.Hp).ToList(); // custom sort
```

---

*Next step from here: pick one project from section 14 — ideally #5, since it plugs straight into the Godot project you're already building — then rebuild it once you understand why the first version was clunky. That loop is basically the whole novice-to-intermediate arc, same as it was for Python.*
