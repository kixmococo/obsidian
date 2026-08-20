# C# Cheatsheet: Strings, Arrays, Parsing, and Everything In Between
_A simple, detailed, overexplained guide_

---

## What this cheatsheet is about

This guide is meant to connect several beginner C# ideas that show up together all the time:

- strings
- characters
- arrays
- indexing
- length
- loops
- splitting text
- converting text into numbers
- parsing
- simple input-style data processing

These topics often appear together because real programs constantly do things like:

- read text
- split it apart
- store pieces in arrays
- convert pieces into numbers
- loop through them
- use the results

So this cheatsheet is about the whole family of those ideas working together.

---

# Part 1: What is a string?

A **string** is text.

In C#, the type is:

```csharp
string
```

## Example

```csharp
string name = "Kai";
string message = "Hello world";
```

A string is a sequence of characters.

So:

```csharp
string word = "cat";
```

really means the text contains these characters:

- `'c'`
- `'a'`
- `'t'`

---

# Part 2: Strings use double quotes

Strings use **double quotes**:

```csharp
string name = "Kai";
```

That tells C#:

> "This is text."

---

# Part 3: What is a char?

A **char** is one single character.

The type is:

```csharp
char
```

## Example

```csharp
char letter = 'A';
char symbol = '!';
```

A char uses **single quotes**, not double quotes.

## Difference

```csharp
char a = 'A';
string b = "A";
```

These are not the same.

- `'A'` = one character
- `"A"` = a string containing one character

---

# Part 4: Strings have indexes

Strings can be accessed by position.

## Example

```csharp
string word = "hello";
Console.WriteLine(word[0]);
```

## Output

```text
h
```

### Why?

Because strings use indexes, and indexes start at 0.

So `"hello"` is arranged like this:

- `word[0]` = `'h'`
- `word[1]` = `'e'`
- `word[2]` = `'l'`
- `word[3]` = `'l'`
- `word[4]` = `'o'`

---

# Part 5: String length

Strings have a `.Length` property.

## Example

```csharp
string word = "hello";
Console.WriteLine(word.Length);
```

## Output

```text
5
```

Because `"hello"` has 5 characters.

---

# Part 6: Looping through a string

Since a string is a sequence of characters, you can loop through it.

## Example

```csharp
string word = "hello";

for (int i = 0; i < word.Length; i++)
{
    Console.WriteLine(word[i]);
}
```

## Output

```text
h
e
l
l
o
```

This is common when you need to inspect text one character at a time.

---

# Part 7: Joining strings together

This is called **concatenation**.

## Example

```csharp
string firstName = "Kai";
string lastName = "Stone";

string fullName = firstName + " " + lastName;
Console.WriteLine(fullName);
```

## Output

```text
Kai Stone
```

The `+` operator can join strings together.

---

# Part 8: String interpolation

A cleaner way to build strings is interpolation.

## Example

```csharp
string name = "Kai";
int age = 20;

Console.WriteLine($"{name} is {age} years old.");
```

## Output

```text
Kai is 20 years old.
```

This is often easier to read than lots of `+`.

---

# Part 9: Common string methods

## `ToUpper()`

```csharp
string word = "hello";
Console.WriteLine(word.ToUpper());
```

Output:

```text
HELLO
```

## `ToLower()`

```csharp
string word = "HELLO";
Console.WriteLine(word.ToLower());
```

Output:

```text
hello
```

## `Trim()`

Removes extra whitespace from the start and end.

```csharp
string text = "   hello   ";
Console.WriteLine(text.Trim());
```

Output:

```text
hello
```

---

# Part 10: `Contains()`

Checks whether a string contains some text.

```csharp
string sentence = "The cat is sleeping.";

bool hasCat = sentence.Contains("cat");
Console.WriteLine(hasCat);
```

Output:

```text
True
```

---

# Part 11: `StartsWith()` and `EndsWith()`

## `StartsWith()`

```csharp
string fileName = "report.txt";
Console.WriteLine(fileName.StartsWith("rep"));
```

## `EndsWith()`

```csharp
Console.WriteLine(fileName.EndsWith(".txt"));
```

These return booleans.

---

# Part 12: Replacing text

```csharp
string text = "I like cats";
string newText = text.Replace("cats", "dogs");

Console.WriteLine(newText);
```

Output:

```text
I like dogs
```

---

# Part 13: Splitting a string

This is one of the most important beginner tools.

`Split()` breaks a string into pieces.

## Example

```csharp
string data = "red,green,blue";
string[] parts = data.Split(',');
```

Now `parts` is an array containing:

- `"red"`
- `"green"`
- `"blue"`

### Print them

```csharp
for (int i = 0; i < parts.Length; i++)
{
    Console.WriteLine(parts[i]);
}
```

---

# Part 14: What is an array?

An **array** is a fixed-size collection of items of the same type.

## Example

```csharp
int[] numbers = { 10, 20, 30, 40 };
```

This array stores several integers.

## Another example

```csharp
string[] names = { "Kai", "Mira", "Sol" };
```

This array stores several strings.

---

# Part 15: Arrays also use indexes

Array indexes also start at 0.

```csharp
string[] names = { "Kai", "Mira", "Sol" };

Console.WriteLine(names[0]);
Console.WriteLine(names[1]);
Console.WriteLine(names[2]);
```

Output:

```text
Kai
Mira
Sol
```

---

# Part 16: Array length

Arrays also have `.Length`.

```csharp
int[] numbers = { 10, 20, 30, 40 };
Console.WriteLine(numbers.Length);
```

Output:

```text
4
```

---

# Part 17: Looping through arrays

## Using `for`

```csharp
int[] numbers = { 10, 20, 30, 40 };

for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}
```

## Using `foreach`

```csharp
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

Both are normal.

Use `for` when you need the index.  
Use `foreach` when you only need the items.

---

# Part 18: Strings split into string arrays

This is a very common pattern:

```csharp
string data = "10,20,30,40";
string[] pieces = data.Split(',');
```

Now the string was broken into an array of smaller strings:

- `"10"`
- `"20"`
- `"30"`
- `"40"`

Important:
They are still strings right now, not ints.

---

# Part 19: What is parsing?

**Parsing** usually means taking text and converting it into a more useful type.

Example:

- `"123"` as text
- becomes `123` as an integer

That conversion is parsing.

---

# Part 20: Parsing integers with `int.Parse()`

```csharp
string textNumber = "42";
int number = int.Parse(textNumber);

Console.WriteLine(number);
```

Output:

```text
42
```

Now it is a real integer, not text.

---

# Part 21: Parsing doubles with `double.Parse()`

```csharp
string textValue = "3.14";
double number = double.Parse(textValue);

Console.WriteLine(number);
```

Output:

```text
3.14
```

---

# Part 22: Why parsing matters

Sometimes data begins as text.

Examples:

- user input
- file input
- lines read from a file
- values from the internet
- comma-separated data

If you want to do math with those values, you often must parse them first.

---

# Part 23: From split strings to parsed numbers

This pattern is extremely common.

## Example

```csharp
string data = "10,20,30";
string[] parts = data.Split(',');

int first = int.Parse(parts[0]);
int second = int.Parse(parts[1]);
int third = int.Parse(parts[2]);

Console.WriteLine(first + second + third);
```

Output:

```text
60
```

---

# Part 24: Safer parsing with `TryParse`

`Parse()` is useful, but if the text is invalid, it can crash.

Safer option:

```csharp
string text = "123";
bool worked = int.TryParse(text, out int number);

Console.WriteLine(worked);
Console.WriteLine(number);
```

Output:

```text
True
123
```

If parsing fails:

```csharp
string text = "abc";
bool worked = int.TryParse(text, out int number);
```

Then:

- `worked` becomes `false`
- `number` becomes `0`

So `TryParse` is safer.

---

# Part 25: Common numeric conversions

## Text to int

```csharp
int x = int.Parse("50");
```

## Text to double

```csharp
double y = double.Parse("2.5");
```

## Number to string

```csharp
int score = 100;
string text = score.ToString();
```

Now `text` is `"100"`.

---

# Part 26: Processing a comma-separated line

This is a very realistic beginner task.

## Example

```csharp
string line = "Kai,20,175";
string[] parts = line.Split(',');

string name = parts[0];
int age = int.Parse(parts[1]);
int height = int.Parse(parts[2]);

Console.WriteLine(name);
Console.WriteLine(age);
Console.WriteLine(height);
```

This is common in file reading and simple data work.

---

# Part 27: Array of parsed numbers from text

```csharp
string data = "5 10 15 20";
string[] pieces = data.Split(' ');

int[] numbers = new int[pieces.Length];

for (int i = 0; i < pieces.Length; i++)
{
    numbers[i] = int.Parse(pieces[i]);
}
```

Now `numbers` contains real ints.

This is a very important pattern:
- split text
- loop through pieces
- parse each piece
- store the parsed values

---

# Part 28: Summing parsed numbers

```csharp
string data = "5 10 15 20";
string[] pieces = data.Split(' ');

int total = 0;

for (int i = 0; i < pieces.Length; i++)
{
    int number = int.Parse(pieces[i]);
    total += number;
}

Console.WriteLine(total);
```

Output:

```text
50
```

---

# Part 29: Empty spaces and `Trim()`

Sometimes text has extra spaces.

```csharp
string data = "   42   ";
int number = int.Parse(data.Trim());

Console.WriteLine(number);
```

`Trim()` helps clean up text before parsing.

---

# Part 30: Strings, arrays, and parsing often form a pipeline

A useful mental model is:

## Step 1
Get text

## Step 2
Clean text

## Step 3
Split text

## Step 4
Loop through pieces

## Step 5
Parse pieces

## Step 6
Use the final values

That is a very common beginner-to-intermediate data workflow.

---

# Part 31: Accessing array elements safely

Bad:

```csharp
string[] names = { "Kai", "Mira" };
Console.WriteLine(names[2]);
```

This crashes because index 2 does not exist.

Valid indexes are:

- `0`
- `1`

So always remember:

```csharp
i < names.Length
```

not

```csharp
i <= names.Length
```

---

# Part 32: Common beginner mistakes

## Mistake 1: Forgetting indexes start at 0

If an array has length 4, valid indexes are:

- 0
- 1
- 2
- 3

not 4.

---

## Mistake 2: Using `.Length` wrong in loops

Wrong:

```csharp
for (int i = 0; i <= numbers.Length; i++)
```

Right:

```csharp
for (int i = 0; i < numbers.Length; i++)
```

---

## Mistake 3: Thinking split values are already numbers

```csharp
string data = "10,20,30";
string[] parts = data.Split(',');
```

These are strings, not ints.

You still need parsing.

---

## Mistake 4: Using `Parse()` on invalid text

```csharp
int x = int.Parse("hello");
```

That fails.

Use `TryParse()` if the text may be bad.

---

## Mistake 5: Confusing `char` and `string`

```csharp
char letter = 'A';
string text = "A";
```

These are different types.

---

# Part 33: A very common pattern with words

```csharp
string sentence = "the quick brown fox";
string[] words = sentence.Split(' ');

foreach (string word in words)
{
    Console.WriteLine(word);
}
```

Output:

```text
the
quick
brown
fox
```

This is common for text processing.

---

# Part 34: A very common pattern with numbers

```csharp
string input = "1,2,3,4,5";
string[] parts = input.Split(',');

int total = 0;

foreach (string part in parts)
{
    total += int.Parse(part);
}

Console.WriteLine(total);
```

Output:

```text
15
```

---

# Part 35: Quick recap of the core relationship

## String
A piece of text.

## Char
One character.

## Array
A collection of same-type items.

## Split
Break one string into many smaller strings.

## Parse
Convert text into a number or another useful type.

## Loop
Process the pieces one by one.

These ideas are best learned together because they constantly work together.

---

# Part 36: Final sample program that combines all of it

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== BASIC STRING INFO ===");
        string sentence = "Kai 10 20 30";

        Console.WriteLine("Sentence: " + sentence);
        Console.WriteLine("Length: " + sentence.Length);
        Console.WriteLine("First character: " + sentence[0]);

        Console.WriteLine();
        Console.WriteLine("=== LOOP THROUGH STRING CHARACTERS ===");
        for (int i = 0; i < sentence.Length; i++)
        {
            Console.WriteLine("Index " + i + ": " + sentence[i]);
        }

        Console.WriteLine();
        Console.WriteLine("=== SPLIT STRING INTO PARTS ===");
        string[] parts = sentence.Split(' ');

        for (int i = 0; i < parts.Length; i++)
        {
            Console.WriteLine("Part " + i + ": " + parts[i]);
        }

        Console.WriteLine();
        Console.WriteLine("=== SAVE FIRST PART AS NAME ===");
        string name = parts[0];
        Console.WriteLine("Name: " + name);

        Console.WriteLine();
        Console.WriteLine("=== PARSE REMAINING PARTS AS INTS ===");
        int[] numbers = new int[parts.Length - 1];

        for (int i = 1; i < parts.Length; i++)
        {
            numbers[i - 1] = int.Parse(parts[i]);
        }

        foreach (int number in numbers)
        {
            Console.WriteLine("Parsed number: " + number);
        }

        Console.WriteLine();
        Console.WriteLine("=== ADD THE PARSED NUMBERS ===");
        int total = 0;

        foreach (int number in numbers)
        {
            total += number;
        }

        Console.WriteLine("Total: " + total);

        Console.WriteLine();
        Console.WriteLine("=== STRING METHODS ===");
        string messy = "   hello world   ";
        Console.WriteLine("Original: [" + messy + "]");
        Console.WriteLine("Trimmed: [" + messy.Trim() + "]");
        Console.WriteLine("Upper: " + messy.Trim().ToUpper());
        Console.WriteLine("Contains 'world': " + messy.Contains("world"));

        Console.WriteLine();
        Console.WriteLine("=== TRY PARSE EXAMPLE ===");
        string goodText = "123";
        string badText = "abc";

        bool goodWorked = int.TryParse(goodText, out int goodNumber);
        bool badWorked = int.TryParse(badText, out int badNumber);

        Console.WriteLine("Good parse worked: " + goodWorked);
        Console.WriteLine("Good number: " + goodNumber);

        Console.WriteLine("Bad parse worked: " + badWorked);
        Console.WriteLine("Bad number: " + badNumber);

        Console.WriteLine();
        Console.WriteLine("=== ARRAY INDEXING ===");
        string[] names = { "Kai", "Mira", "Sol" };

        for (int i = 0; i < names.Length; i++)
        {
            Console.WriteLine("names[" + i + "] = " + names[i]);
        }

        Console.WriteLine();
        Console.WriteLine("=== INTERPOLATION ===");
        Console.WriteLine($"{name} has {numbers.Length} parsed numbers with a total of {total}.");
    }
}
```

---

# Part 37: What the final program teaches

This one program shows:

- strings
- chars through string indexing
- `.Length`
- looping through strings
- splitting text
- arrays
- indexing arrays
- parsing ints
- `TryParse`
- trimming text
- uppercasing text
- checking text with `Contains`
- interpolation
- summing parsed values

---

# Part 38: One sentence memory hook

**In C#, text often starts as a string, gets split into arrays, gets parsed into real values, and then gets processed with loops.**
