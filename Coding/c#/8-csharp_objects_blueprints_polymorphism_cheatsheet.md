# C# Cheatsheet: Objects, Blueprints, and Polymorphism
_A very simple, very detailed guide_

---

## What this cheatsheet is trying to teach

In C#, these ideas are tightly connected:

- **Class** = the **blueprint**
- **Object** = the **real thing built from the blueprint**
- **Polymorphism** = treating different objects through one shared type, while each object still does its own version of the work

Think of it like this:

- A **class** is the design for a car
- An **object** is an actual car built from that design
- **Polymorphism** is being able to say, "All of these are vehicles, so start moving," and each one moves in its own way

---

# Part 1: What is a class?

A **class** is a template, design, or blueprint.

It tells C#:

- what data something has
- what actions something can do

A class by itself is **not yet a real object**.  
It is only the plan.

## Example

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
```

### What this means

We made a class named `Dog`.

Inside it:

- `Name` is data
- `Age` is data
- `Bark()` is behavior

So the blueprint says:

> "Any Dog object will have a name, an age, and the ability to bark."

---

# Part 2: What is an object?

An **object** is a real instance of a class.

That means:

- the class is the blueprint
- the object is the actual thing created from that blueprint

## Example

```csharp
Dog myDog = new Dog();
```

### Breakdown

- `Dog` = the class type
- `myDog` = the variable holding the object
- `new Dog()` = creates a real Dog object in memory

Now `myDog` is a real object.

You can give it values:

```csharp
myDog.Name = "Rex";
myDog.Age = 4;
```

And call its method:

```csharp
myDog.Bark();
```

---

# Part 3: Full simple example of class and object

```csharp
using System;

class Dog
{
    public string Name;
    public int Age;

    public void Bark()
    {
        Console.WriteLine(Name + " says: Woof!");
    }
}

class Program
{
    static void Main()
    {
        Dog myDog = new Dog();

        myDog.Name = "Rex";
        myDog.Age = 4;

        Console.WriteLine("Dog name: " + myDog.Name);
        Console.WriteLine("Dog age: " + myDog.Age);
        myDog.Bark();
    }
}
```

## What happens here

1. We define a `Dog` blueprint
2. We create one real dog object called `myDog`
3. We store data in that object
4. We make that object do something

---

# Part 4: A class is a blueprint, but many objects can come from it

One blueprint can make many objects.

## Example

```csharp
Dog dog1 = new Dog();
dog1.Name = "Rex";
dog1.Age = 4;

Dog dog2 = new Dog();
dog2.Name = "Milo";
dog2.Age = 2;
```

Now we have:

- one `Dog` class
- two different `Dog` objects

That is important.

A class is **one design**.  
Objects are **many actual instances**.

---

# Part 5: Fields, properties, and methods

When learning objects, you will often see these three things.

## 1. Fields

Fields are variables inside a class.

```csharp
public string Name;
public int Age;
```

These store data.

## 2. Properties

Properties are a safer and more common way to expose data.

```csharp
public string Name { get; set; }
public int Age { get; set; }
```

These also store data, but in a more controlled C# way.

## 3. Methods

Methods are actions an object can do.

```csharp
public void Bark()
{
    Console.WriteLine("Woof!");
}
```

---

# Part 6: Better version using properties

This is more modern C# style:

```csharp
using System;

class Dog
{
    public string Name { get; set; }
    public int Age { get; set; }

    public void Bark()
    {
        Console.WriteLine(Name + " says: Woof!");
    }
}

class Program
{
    static void Main()
    {
        Dog myDog = new Dog();
        myDog.Name = "Rex";
        myDog.Age = 4;

        myDog.Bark();
    }
}
```

---

# Part 7: Constructors

A **constructor** is special code that runs when an object is created.

This is useful because it lets you set up the object immediately.

## Example

```csharp
class Dog
{
    public string Name { get; set; }
    public int Age { get; set; }

    public Dog(string name, int age)
    {
        Name = name;
        Age = age;
    }

    public void Bark()
    {
        Console.WriteLine(Name + " says: Woof!");
    }
}
```

Then create objects like this:

```csharp
Dog dog1 = new Dog("Rex", 4);
Dog dog2 = new Dog("Milo", 2);
```

### Why this is nice

Instead of:

```csharp
Dog dog1 = new Dog();
dog1.Name = "Rex";
dog1.Age = 4;
```

you can do it all at creation time.

---

# Part 8: The big picture so far

So far:

- **Class** = blueprint
- **Object** = thing made from blueprint
- **Fields/properties** = data inside the object
- **Methods** = actions the object can do
- **Constructor** = setup code for the object when it is created

That is the foundation of object-oriented programming.

---

# Part 9: What is inheritance?

Inheritance means one class can build on another class.

It is a way of saying:

> "This new thing is a more specific version of that older thing."

## Example idea

- `Animal` = general blueprint
- `Dog` = specific kind of animal
- `Cat` = specific kind of animal

So `Dog` and `Cat` can inherit from `Animal`.

---

# Part 10: Simple inheritance example

```csharp
using System;

class Animal
{
    public string Name { get; set; }

    public void Eat()
    {
        Console.WriteLine(Name + " is eating.");
    }
}

class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine(Name + " says: Woof!");
    }
}

class Cat : Animal
{
    public void Meow()
    {
        Console.WriteLine(Name + " says: Meow!");
    }
}
```

## What `: Animal` means

This means:

- `Dog` inherits from `Animal`
- `Cat` inherits from `Animal`

So both `Dog` and `Cat` get:

- `Name`
- `Eat()`

That means you do **not** need to rewrite that code in every child class.

---

# Part 11: Using inherited classes

```csharp
class Program
{
    static void Main()
    {
        Dog dog = new Dog();
        dog.Name = "Rex";
        dog.Eat();
        dog.Bark();

        Cat cat = new Cat();
        cat.Name = "Luna";
        cat.Eat();
        cat.Meow();
    }
}
```

## Why inheritance matters

Without inheritance, you might repeat the same code in many classes.

Inheritance helps you:

- reuse code
- organize related types
- build general-to-specific relationships

---

# Part 12: What is polymorphism?

This is the part people often think sounds scary.

It is not scary.

**Polymorphism** means:

> one shared type, many forms

That means different objects can be treated as the same general type, but still behave differently.

## Easy everyday idea

Imagine:

- Dog
- Cat
- Bird

All of them are animals.

So you can store them as `Animal`.

But when you tell each one to make a sound:

- Dog says "Woof"
- Cat says "Meow"
- Bird says "Tweet"

Same command idea. Different real behavior.

That is polymorphism.

---

# Part 13: Why polymorphism exists

Polymorphism helps you write code that is:

- more flexible
- easier to expand
- less repetitive

Instead of writing separate logic for every exact type, you can write logic for the general type.

---

# Part 14: The key requirement for runtime polymorphism

The parent class must allow a method to be replaced.

In C#, this usually means using:

- `virtual` in the base class
- `override` in child classes

---

# Part 15: First polymorphism example

```csharp
using System;

class Animal
{
    public string Name { get; set; }

    public virtual void MakeSound()
    {
        Console.WriteLine(Name + " makes a sound.");
    }
}

class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Woof!");
    }
}

class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Meow!");
    }
}
```

---

# Part 16: Why `virtual` and `override` matter

## `virtual`

This tells C#:

> "This method in the parent class is allowed to be replaced by child classes."

```csharp
public virtual void MakeSound()
```

## `override`

This tells C#:

> "This child class is replacing that parent method with its own version."

```csharp
public override void MakeSound()
```

---

# Part 17: Using polymorphism

```csharp
class Program
{
    static void Main()
    {
        Animal a1 = new Dog();
        a1.Name = "Rex";

        Animal a2 = new Cat();
        a2.Name = "Luna";

        a1.MakeSound();
        a2.MakeSound();
    }
}
```

## This is the important part

Look carefully:

```csharp
Animal a1 = new Dog();
Animal a2 = new Cat();
```

The variables are typed as `Animal`, but the real objects are:

- `Dog`
- `Cat`

So when `MakeSound()` runs, C# looks at the **real object type**, not just the variable type.

That is runtime polymorphism.

---

# Part 18: What actually happens in memory conceptually

When you write:

```csharp
Animal a1 = new Dog();
```

You are saying:

- "This variable can hold an Animal reference"
- "I am placing a Dog object into it"

Because a Dog **is an** Animal, this is allowed.

Then if the method is virtual/overridden, C# uses the Dog version.

That is why polymorphism feels like:

> "Use the parent reference, but get the child behavior."

---

# Part 19: A list of many object types using one parent type

This is where polymorphism gets really useful.

```csharp
using System;
using System.Collections.Generic;

class Animal
{
    public string Name { get; set; }

    public virtual void MakeSound()
    {
        Console.WriteLine(Name + " makes a sound.");
    }
}

class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Woof!");
    }
}

class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Meow!");
    }
}

class Bird : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Tweet!");
    }
}

class Program
{
    static void Main()
    {
        List<Animal> animals = new List<Animal>();

        animals.Add(new Dog { Name = "Rex" });
        animals.Add(new Cat { Name = "Luna" });
        animals.Add(new Bird { Name = "Sky" });

        foreach (Animal animal in animals)
        {
            animal.MakeSound();
        }
    }
}
```

## Why this is powerful

The loop does not need to know:

- if the object is Dog
- if the object is Cat
- if the object is Bird

It just knows:

> "This is an Animal, and Animals can MakeSound()."

Each object handles the details itself.

That is one of the main powers of object-oriented design.

---

# Part 20: Base class vs child class

## Base class

Also called:

- parent class
- super class

Example:

```csharp
class Animal
```

## Child class

Also called:

- derived class
- subclass

Example:

```csharp
class Dog : Animal
```

---

# Part 21: What if you do not use `virtual` and `override`?

Then true runtime polymorphism for that method does not happen the same way.

Example:

```csharp
class Animal
{
    public void MakeSound()
    {
        Console.WriteLine("Animal sound");
    }
}

class Dog : Animal
{
    public void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}
```

This is not the same as overriding in proper polymorphism.  
This hides the method rather than participating in true override behavior.

For beginner learning, the safest path is:

- base class method = `virtual`
- child class method = `override`

---

# Part 22: Another way polymorphism appears: abstract classes

Sometimes the parent class is not meant to be used directly.

It only exists to define a shared idea.

In that case, you can use an **abstract class**.

## Example

```csharp
abstract class Animal
{
    public string Name { get; set; }

    public abstract void MakeSound();
}
```

This means:

- you cannot create a plain `Animal` object directly
- every child class must provide its own `MakeSound()` implementation

## Child classes

```csharp
class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Woof!");
    }
}

class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Meow!");
    }
}
```

---

# Part 23: Why abstract classes are useful

They are useful when the base idea is too general to be a real thing.

For example:

- you might not want a generic `Animal`
- but you do want `Dog`, `Cat`, and `Bird`

So the abstract class says:

> "All child classes must follow this shared contract."

---

# Part 24: Another polymorphism tool: interfaces

An **interface** is a contract.

It says:

> "Any class that promises to implement me must provide these methods."

## Example

```csharp
interface IMovable
{
    void Move();
}
```

Now any class that implements `IMovable` must define `Move()`.

```csharp
class Car : IMovable
{
    public void Move()
    {
        Console.WriteLine("Car drives forward.");
    }
}

class Robot : IMovable
{
    public void Move()
    {
        Console.WriteLine("Robot walks forward.");
    }
}
```

---

# Part 25: Interface polymorphism example

```csharp
using System;
using System.Collections.Generic;

interface IMovable
{
    void Move();
}

class Car : IMovable
{
    public void Move()
    {
        Console.WriteLine("Car drives forward.");
    }
}

class Robot : IMovable
{
    public void Move()
    {
        Console.WriteLine("Robot walks forward.");
    }
}

class Person : IMovable
{
    public void Move()
    {
        Console.WriteLine("Person walks forward.");
    }
}

class Program
{
    static void Main()
    {
        List<IMovable> movers = new List<IMovable>();

        movers.Add(new Car());
        movers.Add(new Robot());
        movers.Add(new Person());

        foreach (IMovable mover in movers)
        {
            mover.Move();
        }
    }
}
```

## Why this matters

These types are not all from the same parent class in the example.

But they all agree to the same contract: `IMovable`.

So we can still use polymorphism.

---

# Part 26: Class inheritance polymorphism vs interface polymorphism

## Class inheritance polymorphism

Uses a parent class and child classes.

Example:

- `Animal`
- `Dog`
- `Cat`

## Interface polymorphism

Uses a shared contract.

Example:

- `IMovable`
- `Car`
- `Robot`
- `Person`

---

# Part 27: The simplest mental model

If you get lost, remember this:

## Objects
A real created thing in memory.

## Class
The blueprint used to create objects.

## Inheritance
One blueprint builds on another blueprint.

## Polymorphism
Many different objects can be used through one shared type.

---

# Part 28: Common beginner confusion

## Confusion 1: "Is the class the object?"
No.

- class = design
- object = actual instance created from the design

## Confusion 2: "Why store a Dog inside an Animal variable?"
Because a Dog **is an** Animal.

That lets you write general code.

## Confusion 3: "Why not just use Dog everywhere?"
Because real programs often need to work with many related types together.

Polymorphism makes that easy.

## Confusion 4: "Does the variable type decide everything?"
Not always.

For overridden methods in polymorphism, the real object type matters.

---

# Part 29: A very plain-English analogy

Imagine a game with enemy blueprints.

## Blueprint level
You design:

- Enemy
- Zombie
- Skeleton
- Slime

## Object level
Then during the game you create:

- one Zombie named Rotjaw
- one Skeleton named Bones
- one Slime named Goo

Each of those is an object.

## Polymorphism level
You keep them all in a list of `Enemy`.

Then you call:

```csharp
enemy.Attack();
```

Each enemy attacks in its own way.

That is polymorphism.

---

# Part 30: Example with a game style model

```csharp
using System;
using System.Collections.Generic;

class Enemy
{
    public string Name { get; set; }

    public virtual void Attack()
    {
        Console.WriteLine(Name + " attacks in a generic way.");
    }
}

class Zombie : Enemy
{
    public override void Attack()
    {
        Console.WriteLine(Name + " bites slowly.");
    }
}

class Skeleton : Enemy
{
    public override void Attack()
    {
        Console.WriteLine(Name + " swings a rusty sword.");
    }
}

class Slime : Enemy
{
    public override void Attack()
    {
        Console.WriteLine(Name + " bounces and splashes acid.");
    }
}
```

Using them:

```csharp
List<Enemy> enemies = new List<Enemy>()
{
    new Zombie { Name = "Rotjaw" },
    new Skeleton { Name = "Bones" },
    new Slime { Name = "Goo" }
};

foreach (Enemy enemy in enemies)
{
    enemy.Attack();
}
```

---

# Part 31: Why this is better than a giant if-statement

Without polymorphism, some beginners try this:

```csharp
if (enemyType == "Zombie")
{
    Console.WriteLine("Zombie bites slowly.");
}
else if (enemyType == "Skeleton")
{
    Console.WriteLine("Skeleton swings a sword.");
}
else if (enemyType == "Slime")
{
    Console.WriteLine("Slime splashes acid.");
}
```

This works for tiny programs, but it gets ugly fast.

Polymorphism is cleaner because each class owns its own behavior.

That means:

- easier to add new types
- less giant branching logic
- better organization

---

# Part 32: Super important keywords recap

## `class`
Creates a blueprint.

## `new`
Creates an object from a class.

## `:`
Used for inheritance or interface implementation.

Examples:

```csharp
class Dog : Animal
class Car : IMovable
```

## `virtual`
Lets a base class method be replaced.

## `override`
Replaces that method in the child class.

## `abstract`
Makes a class or method incomplete on purpose, forcing child classes to complete it.

## `interface`
Defines a contract that classes agree to follow.

---

# Part 33: Tiny side-by-side examples

## Regular object creation

```csharp
Dog dog = new Dog();
```

## Inheritance

```csharp
class Dog : Animal
```

## Polymorphic reference

```csharp
Animal pet = new Dog();
```

## Virtual method

```csharp
public virtual void MakeSound()
```

## Override method

```csharp
public override void MakeSound()
```

---

# Part 34: Mini pattern to memorize

Here is the classic pattern:

```csharp
class BaseType
{
    public virtual void DoThing()
    {
        Console.WriteLine("Base version");
    }
}

class ChildType : BaseType
{
    public override void DoThing()
    {
        Console.WriteLine("Child version");
    }
}
```

Then:

```csharp
BaseType item = new ChildType();
item.DoThing();
```

Output:

```text
Child version
```

That is the pattern beginners should memorize.

---

# Part 35: When to use what

## Use a normal class when
You want a blueprint for real objects.

## Use inheritance when
You have an "is-a" relationship.

Examples:

- Dog is an Animal
- Car is a Vehicle

## Use polymorphism when
You want to treat different objects through one shared type.

## Use an abstract class when
The base type is only a concept and should not be directly created.

## Use an interface when
You want a shared capability/contract across possibly unrelated classes.

---

# Part 36: Final sample program that demonstrates all of it together

This program shows:

- a blueprint class
- objects created from that blueprint
- inheritance
- polymorphism with `virtual` and `override`
- interface polymorphism
- multiple objects in lists

```csharp
using System;
using System.Collections.Generic;

// Interface = contract
interface IMovable
{
    void Move();
}

// Base class = general blueprint
class Animal : IMovable
{
    // Property shared by all animals
    public string Name { get; set; }

    // Constructor to set up the object when created
    public Animal(string name)
    {
        Name = name;
    }

    // Virtual method = child classes can replace this behavior
    public virtual void MakeSound()
    {
        Console.WriteLine(Name + " makes a generic animal sound.");
    }

    // Another virtual method
    public virtual void Describe()
    {
        Console.WriteLine(Name + " is some kind of animal.");
    }

    // Interface method
    public virtual void Move()
    {
        Console.WriteLine(Name + " moves in a general way.");
    }
}

// Child class
class Dog : Animal
{
    public Dog(string name) : base(name)
    {
    }

    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Woof!");
    }

    public override void Describe()
    {
        Console.WriteLine(Name + " is a dog.");
    }

    public override void Move()
    {
        Console.WriteLine(Name + " runs on four legs.");
    }
}

// Child class
class Cat : Animal
{
    public Cat(string name) : base(name)
    {
    }

    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Meow!");
    }

    public override void Describe()
    {
        Console.WriteLine(Name + " is a cat.");
    }

    public override void Move()
    {
        Console.WriteLine(Name + " sneaks quietly.");
    }
}

// Child class
class Bird : Animal
{
    public Bird(string name) : base(name)
    {
    }

    public override void MakeSound()
    {
        Console.WriteLine(Name + " says: Tweet!");
    }

    public override void Describe()
    {
        Console.WriteLine(Name + " is a bird.");
    }

    public override void Move()
    {
        Console.WriteLine(Name + " flies through the air.");
    }
}

// Another class using the same interface
class Robot : IMovable
{
    public string Id { get; set; }

    public Robot(string id)
    {
        Id = id;
    }

    public void Move()
    {
        Console.WriteLine("Robot " + Id + " rolls forward on metal wheels.");
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine("=== OBJECTS FROM BLUEPRINTS ===");
        Dog dog = new Dog("Rex");
        Cat cat = new Cat("Luna");
        Bird bird = new Bird("Sky");

        dog.Describe();
        dog.MakeSound();
        dog.Move();

        Console.WriteLine();

        cat.Describe();
        cat.MakeSound();
        cat.Move();

        Console.WriteLine();

        bird.Describe();
        bird.MakeSound();
        bird.Move();

        Console.WriteLine();
        Console.WriteLine("=== POLYMORPHISM THROUGH BASE CLASS ===");

        List<Animal> animals = new List<Animal>();
        animals.Add(dog);
        animals.Add(cat);
        animals.Add(bird);

        foreach (Animal animal in animals)
        {
            animal.Describe();
            animal.MakeSound();
            animal.Move();
            Console.WriteLine();
        }

        Console.WriteLine("=== INTERFACE POLYMORPHISM ===");

        Robot robot = new Robot("RX-99");

        List<IMovable> movers = new List<IMovable>();
        movers.Add(dog);
        movers.Add(cat);
        movers.Add(bird);
        movers.Add(robot);

        foreach (IMovable mover in movers)
        {
            mover.Move();
        }
    }
}
```

---

# Part 37: What this final program proves

## 1. Blueprint
`Animal`, `Dog`, `Cat`, `Bird`, and `Robot` are class blueprints.

## 2. Objects
These are objects:

```csharp
Dog dog = new Dog("Rex");
Cat cat = new Cat("Luna");
Bird bird = new Bird("Sky");
Robot robot = new Robot("RX-99");
```

## 3. Inheritance
These inherit from `Animal`:

- `Dog`
- `Cat`
- `Bird`

## 4. Polymorphism through base class
This list:

```csharp
List<Animal> animals
```

holds different child objects as the same base type.

## 5. Polymorphism through interface
This list:

```csharp
List<IMovable> movers
```

holds different kinds of things that all implement `Move()`.

---

# Part 38: Super short final recap

## Class
Blueprint.

## Object
Real created instance.

## Inheritance
One class builds on another.

## Polymorphism
Use one shared type, get many real behaviors.

---

# Part 39: One sentence memory hook

**A class is the blueprint, an object is the built thing, and polymorphism lets many built things respond differently through one shared type.**
