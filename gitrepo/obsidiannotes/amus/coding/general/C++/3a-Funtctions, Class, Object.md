 a **function**, a **class**, and an **object** all together:

cpp in

```
#include <iostream>
#include <string>

// A simple function outside the class
```c
int add(int x, int y) {
    return x + y;
}

// Define a class
class Player {
public:
    std::string name;
    int health;

    // Class method to display info
    void display() {
        std::cout << "Player: " << name << ", Health: " << health << "\n";
    }
};

int main() {
    // Use the function
    int sum = add(3, 4);
    std::cout << "Sum: " << sum << "\n";

    // Create an object of the class
    Player p1;
    p1.name = "Alex";
    p1.health = 100;

    // Call the object's method
    p1.display();

    return 0;
}
```


cpp out

- `add` is a **function** doing addition.
    
- `Player` is a **class** blueprint with data and a method.
    
- `p1` is an **object** (instance) of `Player`.
    
- `p1.display()` calls the method on that object.
  
  remember;
  for (initialization; condition; increment) {
    // loop body
}
  ---