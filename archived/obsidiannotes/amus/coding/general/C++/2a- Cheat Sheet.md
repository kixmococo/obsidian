
### 1. **cppreference.com**

- The _de facto_ C++ standard library reference online.
    
- Covers core language features, standard library functions, containers, algorithms, and more.
    
- Very detailed, with examples and explanations.
    
- Link: https://en.cppreference.com/w/
    

---

### 2. **cplusplus.com**

- Beginner-friendly and well-organized tutorials and references.
    
- Good for quick lookups and learning standard libraries.
    
- Link: http://www.cplusplus.com/
# basics

int        - whole number type (e.g., 1, 42, -5)
double     - decimal number type (e.g., 3.14, -0.5)
char       - single character (e.g., 'a', 'Z')
string     - sequence of characters (text)
bool       - true or false value

variable   - a named storage for data (e.g., int age = 20;)
function   - a block of code performing a task (e.g., void sayHi() { ... })
class      - blueprint for objects, defines properties and behaviors
object     - instance of a class (an actual thing created from class)

main()     - the starting point of a C++ program
return     - exit a function and optionally send a value back

if / else  - conditional statements (decision making)
for loop   - repeats code a set number of times
while loop - repeats code while a condition is true

#include   - includes libraries/code (e.g., #include <iostream>)
std::cout  - used to print output to screen
std::cin   - used to get input from user

;          - ends a statement
{}         - groups code blocks (like in functions or loops)

# arithmetic

+    Addition             (a + b)
-    Subtraction          (a - b)
*    Multiplication       (a * b)
/    Division             (a / b)  — integer division if both are int
%    Modulus (remainder)  (a % b)  — only for integers

+=   Addition assignment  (a += b)  means a = a + b
-=   Subtraction assignment (a -= b)
*=   Multiplication assignment (a *= b)
/=   Division assignment  (a /= b)
%=   Modulus assignment   (a %= b)

++   Increment by 1  
--   Decrement by 1  

// Examples:
int a = 5, b = 2;
int sum = a + b;       // 7
int diff = a - b;      // 3
int prod = a * b;      // 10
int div = a / b;       // 2 (integer division)
int mod = a % b;       // 1 (remainder)

a += 3;   // a = 8 now
b--;      // b = 1 now

# advanced arithmetic

// Advanced Math Functions (from <cmath>)
```
#include <cmath>
double sqrt(double x);      // square root
double pow(double base, double exp);  // power
double abs(double x);       // absolute value
double ceil(double x);      // round up
double floor(double x);     // round down
double fmod(double x, double y);  // floating-point remainder
```
// Random Number Generation (C++11+)
```
#include <random>

// Create random engine (Mersenne Twister)
std::mt19937 rng(std::random_device{}());

// Uniform integer distribution [min, max]
std::uniform_int_distribution<int> distInt(min, max);
int randomInt = distInt(rng);

// Uniform real distribution [min, max)
std::uniform_real_distribution<double> distReal(min, max);
double randomReal = distReal(rng);
```
// Example:
```c
#include <iostream>
#include <random>
#include <cmath>

int main() {
    std::mt19937 rng(std::random_device{}());
    std::uniform_int_distribution<int> distInt(1, 10);
    int r = distInt(rng);  // random int from 1 to 10

    std::cout << "Random int: " << r << "\n";
    std::cout << "Square root of " << r << " is " << sqrt(r) << "\n";
    return 0;
}
```
std::cout
- `std::cout` → sends output to the console
    
- `std::cin` → gets input from the console
    
- `<<` → output operator (send data out)
    
- `>>` → input operator (bring data in)
    
- For multi-word strings, use `std::getline(std::cin, name)` instead of `std::cin >> name`

example
```
#include <iostream>

int main() {
    int a, b;

    std::cout << "Enter first number: ";
    std::cin >> a;

    std::cout << "Enter second number: ";
    std::cin >> b;

    std::cout << "Addition: " << (a + b) << "\n";
    std::cout << "Subtraction: " << (a - b) << "\n";
    std::cout << "Multiplication: " << (a * b) << "\n";
    std::cout << "Division: " << (a / b) << "\n";
    std::cout << "Modulus: " << (a % b) << "\n";
    
```

  