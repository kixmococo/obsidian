Move Constructor & Move Assignment  
- Special functions to “move” resources instead of copying (performance boost)  
- Use rvalue references (Type&&)

RAII (Resource Acquisition Is Initialization)  
- Manages resource lifetimes by tying them to object lifetimes  
- Ensures automatic cleanup (e.g., file handles, memory)

Template Metaprogramming  
- Using templates to compute values or generate code at compile-time  
- Enables highly generic and optimized code

SFINAE (Substitution Failure Is Not An Error)  
- Template feature for selective function/template instantiation  
- Used in advanced template programming

CRTP (Curiously Recurring Template Pattern)  
- Technique where a class inherits from a template instantiation of itself  
- Enables static polymorphism

Type Traits  
- Compile-time tools to inspect or modify types  
- Found in `<type_traits>` library

Concepts (C++20)  
- Constraints on template parameters to improve error messages and code clarity

Coroutine  
- Functions that can suspend and resume execution  
- Useful for asynchronous programming

Memory Management (Custom Allocators)  
- Writing custom memory allocation strategies for performance tuning

Multithreading & Concurrency  
- Using threads, mutexes, atomic operations for parallelism and thread safety

Expression Templates  
- Technique to optimize complex expressions by delaying evaluation

Linkage & Visibility  
- Control over symbol visibility across translation units and libraries

ABI (Application Binary Interface)  
- Rules for binary compatibility between compiled code modules

Undefined Behavior  
- Code behavior not defined by the standard, often leads to bugs/security issues  
- Critical to understand and avoid

Template Specialization  
- Customizing template behavior for specific types or values

Type Erasure  
- Technique to hide concrete types behind a uniform interface

Move-only Types  
- Types that can be moved but not copied (e.g., std::unique_ptr)

Custom Operators & User-defined Literals  
- Defining new operators or literals for domain-specific uses