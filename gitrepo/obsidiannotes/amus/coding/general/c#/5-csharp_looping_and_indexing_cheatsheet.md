# C# Cheatsheet: Looping and Indexing
_A simple, detailed, overexplained guide_

---

## What this cheatsheet is about

This guide focuses on two ideas that appear together constantly in C#:

- **looping**
- **indexing**

These two ideas are close friends in programming.

Why?

Because a lot of real code does this:

1. start at the first item
2. move through items one by one
3. use an index to know where you are
4. stop when you reach the end

That pattern shows up in:

- arrays
- strings
- lists
- file data
- menus
- game grids
- text processing

So this cheatsheet is about how looping and indexing work together.

---

# Part 1: What is looping?

A **loop** repeats code.

Instead of writing the same code many times, you use a loop to do it again and again.

## Example

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine("Hello");
}
```

This prints `"Hello"` 5 times.

So looping means:

> "Repeat this block of code."

---

# Part 2: What is indexing?

An **index** is a position number for an item inside something like:

- an array
- a string

In C#, indexes usually start at **0**, not 1.

That is a huge rule.

## Example with an array

```csharp
string[] names = { "Kai", "Mira", "Sol" };
```

The indexes are:

- `names[0]` = `"Kai"`
- `names[1]` = `"Mira"`
- `names[2]` = `"Sol"`

---

# Part 3: Why indexing starts at 0

Programming languages often count positions from 0 because it maps cleanly to memory offsets.

Computer science lore version:

Index 0 means:

> "zero steps away from the start"

Index 1 means:

> "one step away from the start"

That is why 0-based indexing became the norm in many languages.

You do not need to love it yet, but you do need to respect it.

---

# Part 4: Looping and indexing together

These ideas often combine like this:

```csharp
string[] names = { "Kai", "Mira", "Sol" };

for (int i = 0; i < names.Length; i++)
{
    Console.WriteLine(names[i]);
}
```

### What is happening

- `i` starts at 0
- `i` is used as the index
- `names[i]` means "the current item"
- loop continues while `i < names.Length`

This is one of the most important beginner patterns in programming.

---

# Part 5: Breaking down the classic loop

```csharp
for (int i = 0; i < names.Length; i++)
```

This has 3 parts:

## 1. Start

```csharp
int i = 0
```

Start the counter at 0.

Why 0?

Because indexing starts at 0.

## 2. Condition

```csharp
i < names.Length
```

Keep looping while `i` is less than the number of items.

## 3. Change

```csharp
i++
```

After each pass, add 1.

So the index moves like this:

- 0
- 1
- 2
- ...

until it reaches the end.

---

# Part 6: Array indexing example

```csharp
int[] numbers = { 10, 20, 30, 40 };

Console.WriteLine(numbers[0]);
Console.WriteLine(numbers[1]);
Console.WriteLine(numbers[2]);
Console.WriteLine(numbers[3]);
```

## Output

```text
10
20
30
40
```

---

# Part 7: String indexing example

Strings also use indexes.

```csharp
string word = "cat";

Console.WriteLine(word[0]);
Console.WriteLine(word[1]);
Console.WriteLine(word[2]);
```

## Output

```text
c
a
t
```

So a string is like a sequence of characters you can access by index.

---

# Part 8: `.Length` is your best friend

Arrays and strings both have `.Length`.

## Example with array

```csharp
int[] numbers = { 10, 20, 30, 40 };
Console.WriteLine(numbers.Length);
```

Output:

```text
4
```

## Example with string

```csharp
string word = "hello";
Console.WriteLine(word.Length);
```

Output:

```text
5
```

`.Length` tells you how many items there are.

That is why loops often use it.

---

# Part 9: Why `i < Length` matters

This is correct:

```csharp
for (int i = 0; i < numbers.Length; i++)
```

This is usually wrong:

```csharp
for (int i = 0; i <= numbers.Length; i++)
```

Why wrong?

If `numbers.Length` is 4, valid indexes are:

- 0
- 1
- 2
- 3

But `4` is not a valid index.

So `<=` goes one step too far.

This is called an **off-by-one error**.

That is one of the oldest beginner bugs in programming history.

---

# Part 10: Visualizing indexes

Example:

```csharp
string[] items = { "A", "B", "C", "D" };
```

Visual map:

- index `0` → `"A"`
- index `1` → `"B"`
- index `2` → `"C"`
- index `3` → `"D"`

Length is `4`, but the last valid index is `3`.

That is a very important rule:

## Last valid index

```csharp
items.Length - 1
```

---

# Part 11: Accessing the last item

```csharp
string[] items = { "A", "B", "C", "D" };

Console.WriteLine(items[items.Length - 1]);
```

## Output

```text
D
```

Why?

Because:

- `items.Length` is 4
- `4 - 1` is 3
- `items[3]` is the last item

---

# Part 12: Looping through arrays with `for`

```csharp
int[] scores = { 90, 75, 88, 100 };

for (int i = 0; i < scores.Length; i++)
{
    Console.WriteLine("Index " + i + " has value " + scores[i]);
}
```

This is useful because you get both:

- the index
- the value

---

# Part 13: Looping through strings with `for`

```csharp
string word = "hello";

for (int i = 0; i < word.Length; i++)
{
    Console.WriteLine("Index " + i + " has char " + word[i]);
}
```

This is useful for character-by-character processing.

---

# Part 14: `foreach` vs indexed `for`

## `foreach`

```csharp
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

This gives you the item directly.

## `for`

```csharp
for (int i = 0; i < names.Length; i++)
{
    Console.WriteLine(names[i]);
}
```

This gives you the index and lets you access by position.

## Rule of thumb

Use `foreach` when:
- you only care about items

Use `for` when:
- you need the index
- you need position control
- you may want to look ahead or behind

---

# Part 15: Using indexes to compare nearby items

This is one reason indexed loops matter.

```csharp
int[] numbers = { 5, 10, 15, 20 };

for (int i = 1; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i] - numbers[i - 1]);
}
```

This compares each item to the one before it.

That kind of task needs indexing.

---

# Part 16: Looping backward

You can also move from the end to the start.

```csharp
string[] names = { "Kai", "Mira", "Sol" };

for (int i = names.Length - 1; i >= 0; i--)
{
    Console.WriteLine(names[i]);
}
```

## Output

```text
Sol
Mira
Kai
```

This starts at the last valid index and moves backward.

---

# Part 17: Nested looping and indexing

Sometimes you loop inside another loop.

Example with a simple grid idea:

```csharp
for (int row = 0; row < 3; row++)
{
    for (int col = 0; col < 4; col++)
    {
        Console.WriteLine("Row " + row + ", Col " + col);
    }
}
```

This is common for:

- maps
- boards
- tables
- 2D game worlds

---

# Part 18: Indexing and assignment

Indexes are not just for reading.

They can also be used to change values.

## Example

```csharp
int[] numbers = { 10, 20, 30 };

numbers[1] = 999;

Console.WriteLine(numbers[1]);
```

## Output

```text
999
```

So indexing lets you get or set items.

---

# Part 19: Filling an array with a loop

```csharp
int[] values = new int[5];

for (int i = 0; i < values.Length; i++)
{
    values[i] = i * 10;
}
```

Now the array becomes:

- `0`
- `10`
- `20`
- `30`
- `40`

This is a very common pattern.

---

# Part 20: Counting characters in a string

```csharp
string text = "banana";
int countA = 0;

for (int i = 0; i < text.Length; i++)
{
    if (text[i] == 'a')
    {
        countA++;
    }
}

Console.WriteLine(countA);
```

## Output

```text
3
```

This is a classic example of looping and indexing working together.

---

# Part 21: Summing an array

```csharp
int[] numbers = { 5, 10, 15, 20 };
int total = 0;

for (int i = 0; i < numbers.Length; i++)
{
    total += numbers[i];
}

Console.WriteLine(total);
```

## Output

```text
50
```

---

# Part 22: Common beginner mistakes

## Mistake 1: Starting at 1 instead of 0

Wrong:

```csharp
for (int i = 1; i < names.Length; i++)
```

This skips the first item at index 0.

Sometimes that is intentional, but beginners often do it by accident.

---

## Mistake 2: Using `<= Length`

Wrong:

```csharp
for (int i = 0; i <= names.Length; i++)
```

This goes too far.

Right:

```csharp
for (int i = 0; i < names.Length; i++)
```

---

## Mistake 3: Forgetting the last valid index is `Length - 1`

If length is 5, last valid index is 4.

Not 5.

---

## Mistake 4: Mixing up item and index

In this loop:

```csharp
for (int i = 0; i < names.Length; i++)
```

`i` is the index, not the value itself.

The value is:

```csharp
names[i]
```

That difference matters a lot.

---

# Part 23: Reading the pattern in plain English

```csharp
for (int i = 0; i < items.Length; i++)
{
    Console.WriteLine(items[i]);
}
```

Plain English:

- start at the first index
- keep going while still inside the collection
- move one step each time
- print the item at the current index

If you can read that pattern comfortably, you are getting stronger.

---

# Part 24: A useful mental picture

Imagine an array like houses on a street:

```csharp
string[] houses = { "Red", "Blue", "Green" };
```

The indexes are house numbers:

- house 0 = Red
- house 1 = Blue
- house 2 = Green

A loop with `i` is like walking down the street one house at a time.

That is basically what indexed looping is.

---

# Part 25: A common pattern with strings and words

```csharp
string sentence = "the quick brown fox";
string[] words = sentence.Split(' ');

for (int i = 0; i < words.Length; i++)
{
    Console.WriteLine("Word " + i + ": " + words[i]);
}
```

Here:

- `Split()` creates an array
- the loop walks through the array
- `i` tells you which word you are on

---

# Part 26: Sample program that combines looping and indexing

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== ARRAY INDEXING ===");
        string[] names = { "Kai", "Mira", "Sol", "Juno" };

        Console.WriteLine("First item: " + names[0]);
        Console.WriteLine("Last item: " + names[names.Length - 1]);

        Console.WriteLine();
        Console.WriteLine("=== LOOP FORWARD THROUGH ARRAY ===");
        for (int i = 0; i < names.Length; i++)
        {
            Console.WriteLine("Index " + i + " -> " + names[i]);
        }

        Console.WriteLine();
        Console.WriteLine("=== LOOP BACKWARD THROUGH ARRAY ===");
        for (int i = names.Length - 1; i >= 0; i--)
        {
            Console.WriteLine("Index " + i + " -> " + names[i]);
        }

        Console.WriteLine();
        Console.WriteLine("=== CHANGE AN ITEM USING INDEX ===");
        names[1] = "Rin";

        for (int i = 0; i < names.Length; i++)
        {
            Console.WriteLine("Index " + i + " -> " + names[i]);
        }

        Console.WriteLine();
        Console.WriteLine("=== STRING INDEXING ===");
        string word = "banana";

        Console.WriteLine("First char: " + word[0]);
        Console.WriteLine("Last char: " + word[word.Length - 1]);

        Console.WriteLine();
        Console.WriteLine("=== LOOP THROUGH STRING CHARACTERS ===");
        for (int i = 0; i < word.Length; i++)
        {
            Console.WriteLine("Index " + i + " -> " + word[i]);
        }

        Console.WriteLine();
        Console.WriteLine("=== COUNT A CHAR IN A STRING ===");
        int countA = 0;

        for (int i = 0; i < word.Length; i++)
        {
            if (word[i] == 'a')
            {
                countA++;
            }
        }

        Console.WriteLine("Number of 'a' characters: " + countA);

        Console.WriteLine();
        Console.WriteLine("=== SUM AN ARRAY ===");
        int[] numbers = { 5, 10, 15, 20 };
        int total = 0;

        for (int i = 0; i < numbers.Length; i++)
        {
            total += numbers[i];
            Console.WriteLine("Index " + i + " has value " + numbers[i]);
        }

        Console.WriteLine("Total: " + total);

        Console.WriteLine();
        Console.WriteLine("=== FILL AN ARRAY WITH A LOOP ===");
        int[] values = new int[5];

        for (int i = 0; i < values.Length; i++)
        {
            values[i] = i * 100;
        }

        for (int i = 0; i < values.Length; i++)
        {
            Console.WriteLine("values[" + i + "] = " + values[i]);
        }
    }
}
```

---

# Part 27: What the final program teaches

This one program shows:

- array indexing
- string indexing
- first and last item access
- forward loops
- backward loops
- changing items by index
- counting characters
- summing values
- filling arrays with loops
- using `.Length` correctly

---

# Part 28: Final recap

## Looping
Repeating code.

## Indexing
Using position numbers to access items.

## Core rule
Indexes usually start at 0.

## Safe loop pattern
```csharp
for (int i = 0; i < items.Length; i++)
```

## Last valid index
```csharp
items.Length - 1
```

## Big idea
Looping moves through data.  
Indexing tells you where you are in that data.

---

# Part 29: One sentence memory hook

**Looping is how you move through data, and indexing is how you know which exact item you are touching.**
