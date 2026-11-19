Constructor  
- Special function to initialize an object when created  
- Same name as the class, no return type

Destructor  
- Special function called when an object is destroyed  
- Used to free resources, cleanup

Inheritance  
- Mechanism where a class (child) derives from another class (parent)  
- Child inherits properties and methods of parent

Polymorphism  
- Ability to treat objects of different classes through a common interface  
- Often via virtual functions and method overriding

Virtual Function  
- A function that can be overridden in derived classes  
- Enables runtime polymorphism (dynamic dispatch)

Abstract Class  
- A class with at least one pure virtual function  
- Cannot be instantiated directly, used as interface/template

Template  
- Enables writing generic functions or classes  
- Works with any data type

Exception Handling  
- Mechanism to handle runtime errors using try, catch, throw blocks

Reference  
- An alias for another variable  
- Safer alternative to pointers in many cases

Move Semantics  
- Optimize resource transfer between objects (C++11)  
- Uses rvalue references (&&) to avoid unnecessary copying

Lambda Function  
- Anonymous inline functions, useful for short snippets of code  
- Syntax example: [](int x) { return x * 2; }

Smart Pointer  
- Objects that manage pointer lifetimes automatically  
- Examples: std::unique_ptr, std::shared_ptr

Const  
- Keyword indicating immutability for variables, methods, parameters

Operator Overloading  
- Defining custom behavior for operators (+, -, etc.) on user-defined types

Friend Function/Class  
- Grants access to private members of a class from another function or class