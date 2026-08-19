# C# Quick Cheat Sheet

## 1. Basic Program

```csharp
Console.WriteLine("Hello, world!");
```

## 2. Variables

```csharp
int age = 25;
double price = 19.99;
char grade = 'A';
string name = "u1";
bool isReady = true;

const double PI = 3.14159;
```

## 3. Type Casting

```csharp
double x = 9.8;
int y = (int)x;

string numText = "42";
int num = int.Parse(numText);

string maybeNum = "123";
bool worked = int.TryParse(maybeNum, out int result);
```

## 4. Input / Output

```csharp
Console.Write("Enter your name: ");
string userName = Console.ReadLine();

Console.WriteLine("Hello " + userName);
Console.WriteLine($"Hello {userName}");
```

## 5. Operators

```csharp
int a = 10;
int b = 3;

Console.WriteLine(a + b);
Console.WriteLine(a - b);
Console.WriteLine(a * b);
Console.WriteLine(a / b);
Console.WriteLine(a % b);

Console.WriteLine(a > b);
Console.WriteLine(a == b);
Console.WriteLine(a != b);

Console.WriteLine(true && false);
Console.WriteLine(true || false);
Console.WriteLine(!true);
```

## 6. If / Else

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("A");
}
else if (score >= 80)
{
    Console.WriteLine("B");
}
else
{
    Console.WriteLine("Needs work");
}
```

## 7. Switch

```csharp
int day = 2;

switch (day)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    default:
        Console.WriteLine("Unknown");
        break;
}
```

## 8. Loops

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);
}

int count = 0;
while (count < 3)
{
    Console.WriteLine(count);
    count++;
}

int n = 0;
do
{
    Console.WriteLine(n);
    n++;
} while (n < 2);
```

## 9. Arrays

```csharp
int[] numbers = { 10, 20, 30 };

Console.WriteLine(numbers[0]);
Console.WriteLine(numbers.Length);

for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}
```

## 10. Foreach

```csharp
string[] names = { "Alice", "Bob", "Charlie" };

foreach (string person in names)
{
    Console.WriteLine(person);
}
```

## 11. Lists

```csharp
using System.Collections.Generic;

List<string> items = new List<string>();
items.Add("Sword");
items.Add("Shield");
items.Remove("Sword");

foreach (string item in items)
{
    Console.WriteLine(item);
}
```

## 12. Methods

```csharp
static void SayHello()
{
    Console.WriteLine("Hello!");
}

static int Add(int num1, int num2)
{
    return num1 + num2;
}

SayHello();
int sum = Add(3, 4);
Console.WriteLine(sum);
```

## 13. Strings

```csharp
string text = "Hello";

Console.WriteLine(text.Length);
Console.WriteLine(text.ToUpper());
Console.WriteLine(text.ToLower());
Console.WriteLine(text.Contains("ell"));
Console.WriteLine(text[0]);
```

## 14. String Interpolation

```csharp
string firstName = "u1";
int level = 7;

Console.WriteLine($"{firstName} is level {level}");
```

## 15. Exceptions

```csharp
try
{
    int z = int.Parse("not a number");
}
catch (Exception ex)
{
    Console.WriteLine("Error: " + ex.Message);
}
```

## 16. Classes and Objects

```csharp
class Dog
{
    public string Name;
    public int Age;

    public void Bark()
    {
        Console.WriteLine("Woof!");
    }
}

Dog myDog = new Dog();
myDog.Name = "Rex";
myDog.Age = 3;
myDog.Bark();
```

## 17. Constructor

```csharp
class Cat
{
    public string Name;

    public Cat(string name)
    {
        Name = name;
    }
}

Cat myCat = new Cat("Milo");
Console.WriteLine(myCat.Name);
```

## 18. Properties

```csharp
class Player
{
    public string Name { get; set; }
    public int Health { get; set; }
}

Player p = new Player();
p.Name = "Knight";
p.Health = 100;
```

## 19. Random

```csharp
Random rng = new Random();
int randomNumber = rng.Next(1, 11);
```

## 20. Null

```csharp
string maybeName = null;

if (maybeName == null)
{
    Console.WriteLine("No name yet");
}
```

## 21. Files

```csharp
using System.IO;

File.WriteAllText("test.txt", "Hello file!");
string fileText = File.ReadAllText("test.txt");
Console.WriteLine(fileText);
```

## Fast Mental Model

- `int`, `double`, `string`, `bool` = common types
- `if` / `else` = decisions
- loops = repeat work
- arrays / lists = store many values
- methods = reusable actions
- classes = make your own types

## Tiny CS Lore

- C# is a compiled, high-level, object-oriented language.
- It usually runs on .NET, which is why you use `dotnet run`, `dotnet build`, and `dotnet publish`.
- It is in the same general-purpose app language family as Java, but with Microsoft/.NET flavor.
