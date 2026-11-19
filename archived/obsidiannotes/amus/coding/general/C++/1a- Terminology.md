In C++, these are not _commands_ but rather **language elements** or **syntax constructs** such as:

- **Keywords** (e.g., `int`, `class`, `return`)
    
- **Operators** (e.g., `+`, `-`, `++`)
    
- **Functions** (e.g., `main()`, `sqrt()`)
    
- **Statements** (e.g., `if`, `for`)
    
- **Declarations** (e.g., variable and function declarations)
    
- **Expressions** (combinations of variables, operators, and function calls)
    

So, “commands” is not the correct term; use **keywords**, **operators**, **functions**, or more generally **syntax elements** depending on context.

Function  
- A named block of code that performs a task  
- Can take inputs (parameters) and return outputs  
- Example:  
  int add(int a, int b) { return a + b; }

Loop  
- Repeats code multiple times based on a condition  
- Types: for, while, do-while  
- Example (for loop):  
  for(int i = 0; i < 5; i++) { /* code */ }

Class  
- Blueprint or template for creating objects  
- Defines properties (variables) and behaviors (functions)  
- Example:  
  class Dog {  
    public:  
      void bark() { /* code */ }  
  };

Object  
- An instance of a class, a concrete entity based on the class  
- Holds its own data defined by the class  
- Example:  
  Dog myDog;  
  myDog.bark();

Variable  
- Named storage for data  
- Has a type (int, double, etc.)  
- Example:  
  int age = 25;

Statements
are statements sometimes referred to as conditionals
	ChatGPT said:
Yes, but only **some statements** are called **conditionals**—specifically those that make decisions based on conditions, like:
- `if`
    
- `else if`
    
- `else`
    
- `switch`
More generally, **statements** include many things: conditionals, loops, declarations, expressions, etc. So conditionals are a subset of statements.

Conditional  
- Executes code only if a condition is true  
- Uses if, else if, else statements  
- Example:  
  if(age > 18) { /* code */ } else { /* code */ }

Pointer  
- Stores the memory address of another variable  
- Used for dynamic memory and advanced operations  
- Example:  
  int* ptr = &age;

Namespace  
- A way to group code and avoid name conflicts  
- Example: std::cout (std is a namespace)

Header File  
- File containing declarations for functions/classes  
- Included with #include directive  