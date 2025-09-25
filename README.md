#include <SFML/Graphics.hpp>
#include <vector>
#include <iostream>
#include <cstdlib>

const int CELL_SIZE = 50;
const int WIDTH = 800;
const int HEIGHT = 600;

class Player {
public:
    sf::RectangleShape shape;
    float speed;

    Player() {
        shape.setSize(sf::Vector2f(CELL_SIZE, CELL_SIZE));
        shape.setFillColor(sf::Color::Blue);
        shape.setPosition(100, 100);
        speed = 5.0f;
    }

    void move(const sf::Vector2f& direction) {
        shape.move(direction * speed);
    }
};

class Enemy {
public:
    sf::RectangleShape shape;

    Enemy(float x, float y) {
        shape.setSize(sf::Vector2f(CELL_SIZE, CELL_SIZE));
        shape.setFillColor(sf::Color::Red);
        shape.setPosition(x, y);
    }
};

class Maze {
public:
    std::vector<std::string> layout;

    Maze() {
        layout = {
            "1111111111111111",
            "1000000000000011",
            "1111111111111011",
            "1000000000000011",
            "1111111111111011",
            "1000000000000011",
            "1111111111111111"
        };
    }

    void draw(sf::RenderWindow& window) {
        for (int y = 0; y < layout.size(); ++y) {
            for (int x = 0; x < layout[y].size(); ++x) {
                if (layout[y][x] == '1') {
                    sf::RectangleShape wall(sf::Vector2f(CELL_SIZE, CELL_SIZE));
                    wall.setFillColor(sf::Color::White);
                    wall.setPosition(x * CELL_SIZE, y * CELL_SIZE);
                    window.draw(wall);
                }
            }
        }
    }

    bool isWall(float x, float y) {
        int gridX = static_cast<int>(x / CELL_SIZE);
        int gridY = static_cast<int>(y / CELL_SIZE);
        return layout[gridY][gridX] == '1';
    }
};

void spawnEnemies(std::vector<Enemy>& enemies) {
    enemies.clear();
    for (int i = 0; i < 5; ++i) {
        float x = (rand() % (WIDTH / CELL_SIZE)) * CELL_SIZE;
        float y = (rand() % (HEIGHT / CELL_SIZE)) * CELL_SIZE;
        enemies.emplace_back(x, y);
    }
}

int main() {
    sf::RenderWindow window(sf::VideoMode(WIDTH, HEIGHT), "Maze Game");
    Player player;
    Maze maze;
    std::vector<Enemy> enemies;
    spawnEnemies(enemies);

    sf::View view = window.getDefaultView();

    while (window.isOpen()) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        // Движение игрока
        sf::Vector2f direction(0.f, 0.f);
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left)) direction.x -= 1.f;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right)) direction.x += 1.f;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up)) direction.y -= 1.f;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down)) direction.y += 1.f;

        // Проверка столкновения с стенами
        if (!maze.isWall(player.shape.getPosition().x + direction.x * player.speed, player.shape.getPosition().y))
            player.move(sf::Vector2f(direction.x, 0));
        if (!maze.isWall(player.shape.getPosition().x, player.shape.getPosition().y + direction.y * player.speed))
            player.move(sf::Vector2f(0, direction.y));

        // Обновление камеры
        view.setCenter(player.shape.getPosition().x + CELL_SIZE / 2, player.shape.getPosition().y + CELL_SIZE / 2);
        
        // Обработка атаки
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Space)) {
            for (auto it = enemies.begin(); it != enemies.end();) {
                                if (player.shape.getGlobalBounds().intersects(it->shape.getGlobalBounds())) {
                    it = enemies.erase(it); // Удаляем врага при атаке
                } else {
                    ++it;
                }
            }
        }

        // Отрисовка
        window.clear();
        maze.draw(window);
        window.draw(player.shape);
        for (const auto& enemy : enemies) {
            window.draw(enemy.shape);
        }
        window.setView(view);
        window.display();
    }

    return 0;
}
