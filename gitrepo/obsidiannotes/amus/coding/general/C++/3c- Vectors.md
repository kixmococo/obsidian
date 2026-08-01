```c
#include <iostream>
#include <string>
#include <vector>

class Player {
public:
    std::string name;
    int health;

    Player(std::string n, int h) : name(n), health(h) {}

    void display() {
        std::cout << name << " (HP: " << health << ")\n";
    }
};

int main() {
    std::vector<Player> players;
    int numPlayers;

    std::cout << "How many players? ";
    std::cin >> numPlayers;
    std::cin.ignore(); // Clear newline

    for (int i = 0; i < numPlayers; ++i) {
        std::string name;
        int health;

        std::cout << "Enter name for player " << (i + 1) << ": ";
        std::getline(std::cin, name);

        std::cout << "Enter health for player " << (i + 1) << ": ";
        std::cin >> health;
        std::cin.ignore();

        players.emplace_back(name, health); // Create & add Player object
    }

    std::cout << "\nPlayers created:\n";
    for (auto& p : players) {
        p.display();
    }

    return 0;
}
Explanation:
numPlayers is from user input.

Loop runs that many times, each time:

Prompt for name and health.

Create a Player object with that data.

Store the object in a std::vector<Player>.

Finally, display all players.
```

