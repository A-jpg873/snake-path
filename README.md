#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <iostream>

using namespace sf;

// Размеры клетки и поля
const int cellSize = 40;
const int cellNumber = 20;
const int windowSize = cellSize * cellNumber;

// Структура для сегмента змейки
struct SnakeSegment {
    int x, y;
    SnakeSegment(int _x, int _y) : x(_x), y(_y) {}
};

enum GameState {
    MAIN_MENU,
    GAME,
    SKIN_MENU
};

GameState gameState = MAIN_MENU;

class Button {
public:
    // Предположим, что у вас есть такие члены для кнопки
    RectangleShape shape;
    Text text;
    
    // Конструктор
    Button(float x, float y, float width, float height, Font& font, const std::string& textStr) {
        shape.setPosition(x, y);
        shape.setSize(Vector2f(width, height));
        shape.setFillColor(Color::Green);  // Изначальный цвет кнопки
        text.setFont(font);
        text.setString(textStr);
        text.setCharacterSize(20);
        text.setFillColor(Color::White);
        text.setPosition(x + 10, y + 10); // Позиция текста внутри кнопки
    }

    // Метод для установки цвета кнопки
    void setColor(Color color) {
        shape.setFillColor(color);
    }

    // Метод для отрисовки кнопки
    void draw(RenderWindow& window) {
        window.draw(shape);
        window.draw(text);
    }

    // Метод для проверки клика
    bool isClicked(Vector2i mousePos) {
        return shape.getGlobalBounds().contains(Vector2f(mousePos));
    }
};



// Класс змейки
class Snake {
public:
    std::vector<SnakeSegment> body;
    Vector2i direction = {1, 0};
    bool newBlock = false;

    // Текстуры для головы, хвоста и тела
    Texture headTextures[4];
    Texture tailTextures[4];
    Texture bodyHorizontal, bodyVertical;
    Texture bodyTL, bodyTR, bodyBL, bodyBR;

    Sprite head, tail, bodySegment;

    // Звук для поедания
    SoundBuffer crunchBuffer;
    Sound crunchSound;

    // Счёт и рекорд
    int score = 0;
    int record = 0;

    Snake() {
        reset();

        // Загрузка текстур для головы
        if (!headTextures[0].loadFromFile("Graphics/head_up.png"))
            std::cerr << "Error loading head_up.png" << std::endl;
        if (!headTextures[1].loadFromFile("Graphics/head_down.png"))
            std::cerr << "Error loading head_down.png" << std::endl;
        if (!headTextures[2].loadFromFile("Graphics/head_left.png"))
            std::cerr << "Error loading head_left.png" << std::endl;
        if (!headTextures[3].loadFromFile("Graphics/head_right.png"))
            std::cerr << "Error loading head_right.png" << std::endl;

        // Загрузка текстур для хвоста
        if (!tailTextures[0].loadFromFile("Graphics/tail_up.png"))
            std::cerr << "Error loading tail_up.png" << std::endl;
        if (!tailTextures[1].loadFromFile("Graphics/tail_down.png"))
            std::cerr << "Error loading tail_down.png" << std::endl;
        if (!tailTextures[2].loadFromFile("Graphics/tail_left.png"))
            std::cerr << "Error loading tail_left.png" << std::endl;
        if (!tailTextures[3].loadFromFile("Graphics/tail_right.png"))
            std::cerr << "Error loading tail_right.png" << std::endl;

        // Загрузка текстур для тела
        if (!bodyHorizontal.loadFromFile("Graphics/body_horizontal.png"))
            std::cerr << "Error loading body_horizontal.png" << std::endl;
        if (!bodyVertical.loadFromFile("Graphics/body_vertical.png"))
            std::cerr << "Error loading body_vertical.png" << std::endl;
        if (!bodyTL.loadFromFile("Graphics/body_tl.png"))
            std::cerr << "Error loading body_tl.png" << std::endl;
        if (!bodyTR.loadFromFile("Graphics/body_tr.png"))
            std::cerr << "Error loading body_tr.png" << std::endl;
        if (!bodyBL.loadFromFile("Graphics/body_bl.png"))
            std::cerr << "Error loading body_bl.png" << std::endl;
        if (!bodyBR.loadFromFile("Graphics/body_br.png"))
            std::cerr << "Error loading body_br.png" << std::endl;

        // Загрузка звука
        if (!crunchBuffer.loadFromFile("Sound/crunch.wav"))
            std::cerr << "Error loading crunch.wav" << std::endl;
        crunchSound.setBuffer(crunchBuffer);
    }

    void reset() {
        body.clear();
        body.push_back(SnakeSegment(5, 10));
        body.push_back(SnakeSegment(4, 10));
        body.push_back(SnakeSegment(3, 10));
        direction = {1, 0};
        newBlock = false;
        score = 0;
    }

    void playCrunchSound() {
        crunchSound.play();
    }

    void move() {
    if (newBlock) {
        body.insert(body.begin(), SnakeSegment(body[0].x + direction.x, body[0].y + direction.y));
        newBlock = false;
    } else {
        body.pop_back();
        body.insert(body.begin(), SnakeSegment(body[0].x + direction.x, body[0].y + direction.y));
    }

    // Проверка на столкновение со стенами
    if (body[0].x < 0 || body[0].x >= cellNumber || body[0].y < 0 || body[0].y >= cellNumber) {
        reset();
    }

    // Проверка на столкновение с собой
    for (size_t i = 1; i < body.size(); ++i) {
        if (body[0].x == body[i].x && body[0].y == body[i].y) {
            reset();
        }
    }
}

void move(bool &isGameOver) {
    if (newBlock) {
        body.insert(body.begin(), SnakeSegment(body[0].x + direction.x, body[0].y + direction.y));
        newBlock = false;
    } else {
        body.pop_back();
        body.insert(body.begin(), SnakeSegment(body[0].x + direction.x, body[0].y + direction.y));
    }

    // Проверка на столкновение со стенами
    if (body[0].x < 0 || body[0].x >= cellNumber || body[0].y < 0 || body[0].y >= cellNumber) {
        isGameOver = true;
    }

    // Проверка на столкновение с собой
    for (size_t i = 1; i < body.size(); ++i) {
        if (body[0].x == body[i].x && body[0].y == body[i].y) {
            isGameOver = true;
        }
    }
}


    void draw(RenderWindow &window) {
        // Отрисовка головы
        updateHeadTexture();
        head.setPosition(body[0].x * cellSize, body[0].y * cellSize);
        window.draw(head);

        // Отрисовка тела
        for (size_t i = 1; i < body.size() - 1; ++i) {
            int x = body[i].x * cellSize;
            int y = body[i].y * cellSize;
            bodySegment.setPosition(x, y);
            
            Vector2i prev = {body[i - 1].x - body[i].x, body[i - 1].y - body[i].y};
            Vector2i next = {body[i + 1].x - body[i].x, body[i + 1].y - body[i].y};

            if (prev.x == 0 && next.x == 0)
                bodySegment.setTexture(bodyVertical);
            else if (prev.y == 0 && next.y == 0)
                bodySegment.setTexture(bodyHorizontal);
            else if ((prev.x == 1 && next.y == 1) || (prev.y == 1 && next.x == 1))
                bodySegment.setTexture(bodyTL);
            else if ((prev.x == -1 && next.y == 1) || (prev.y == 1 && next.x == -1))
                bodySegment.setTexture(bodyTR);
            else if ((prev.x == 1 && next.y == -1) || (prev.y == -1 && next.x == 1))
                bodySegment.setTexture(bodyBL);
            else if ((prev.x == -1 && next.y == -1) || (prev.y == -1 && next.x == -1))
                bodySegment.setTexture(bodyBR);

            window.draw(bodySegment);
        }

        // Отрисовка хвоста
        updateTailTexture();
        tail.setPosition(body.back().x * cellSize, body.back().y * cellSize);
        window.draw(tail);
    }

    void updateHeadTexture() {
        if (direction == Vector2i(0, -1))
            head.setTexture(headTextures[0]);
        else if (direction == Vector2i(0, 1))
            head.setTexture(headTextures[1]);
        else if (direction == Vector2i(-1, 0))
            head.setTexture(headTextures[2]);
        else if (direction == Vector2i(1, 0))
            head.setTexture(headTextures[3]);
    }

    void updateTailTexture() {
        Vector2i tailDir = {body[body.size() - 2].x - body.back().x, body[body.size() - 2].y - body.back().y};
        if (tailDir == Vector2i(0, -1))
            tail.setTexture(tailTextures[0]);
        else if (tailDir == Vector2i(0, 1))
            tail.setTexture(tailTextures[1]);
        else if (tailDir == Vector2i(-1, 0))
            tail.setTexture(tailTextures[2]);
        else if (tailDir == Vector2i(1, 0))
            tail.setTexture(tailTextures[3]);
    }
};

    /// Класс для фрукта
    class Fruit {
    public:
    Vector2i position;
    Texture appleTexture;
    Sprite apple;

    Fruit() {
        appleTexture.loadFromFile("Graphics/apple.png");
        apple.setTexture(appleTexture);
        randomize();
    }

    void randomize() {
        position.x = rand() % cellNumber;
        position.y = rand() % cellNumber;
    }

    void draw(RenderWindow &window) {
        apple.setPosition(position.x * cellSize, position.y * cellSize);
        window.draw(apple);
    }
};

int main() {
    srand(time(nullptr));

    RenderWindow window(VideoMode(windowSize, windowSize), "Snake Game");

    Font font;
    if (!font.loadFromFile("Font/PoetsenOne-Regular.ttf")) {
        std::cerr << "Error loading font!" << std::endl;
        return -1;
    }

    // Тексты
    Text pressToStartText;
    pressToStartText.setFont(font);
    pressToStartText.setString("       Press to Start");
    pressToStartText.setCharacterSize(40);
    pressToStartText.setFillColor(Color::White);
    pressToStartText.setPosition(windowSize / 4, windowSize / 3);

    Text gameOverText;
    gameOverText.setFont(font);
    gameOverText.setString("            Game Over\n       Press to restart");
    gameOverText.setCharacterSize(40);
    gameOverText.setFillColor(Color::White);
    gameOverText.setPosition(windowSize / 4, windowSize / 3);

    // Текст для рекорда и счёта
    Text scoreText;
    scoreText.setFont(font);
    scoreText.setCharacterSize(30);
    scoreText.setFillColor(Color::Black);
    scoreText.setPosition(10, 10);

    Text recordText;
    recordText.setFont(font);
    recordText.setCharacterSize(30);
    recordText.setFillColor(Color::Red);
    recordText.setPosition(150, 10);

    // Игровые объекты
    Snake snake;
    Fruit fruit;

    // Кнопки главного меню
    Button playButton(windowSize / 4, windowSize / 3, windowSize / 2, 50, font, "                                         Play");
    Button skinButton(windowSize / 4, windowSize / 2, windowSize / 2, 50, font, "                                         Skins");
    Button exitButton(windowSize / 4, windowSize / 1.5, windowSize / 2, 50, font, "   // Кнопки меню скинов
    Button backToMenuButton(windowSize / 4, windowSize / 1.5, windowSize / 2, 50, font, "Back to Menu");

    // Кнопка для возврата в главное меню с экрана Game Over
    Button backToMainMenuFromGameOver(windowSize / 4, windowSize / 1.5, windowSize / 2, 50, font, "                        Back to Main Menu");

    // Состояние игры
    GameState gameState = MAIN_MENU;
    bool waitingToStart = false; // Флаг, чтобы ждать начала игры

    Clock clock;
    float timer = 0, delay = 0.1f;

    bool isGameOver = false;

    Texture gameBackground;
    if (!gameBackground.loadFromFile("Graphics/grean800.png")) {
        std::cerr << "Error loading background!" << std::endl;
        return -1;
    }

    while (window.isOpen()) {
        float time = clock.restart().asSeconds();
        timer += time;

        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed)
                window.close();

            if (gameState == MAIN_MENU) {
                // Проверка нажатий кнопок главного меню
                if (event.type == Event::MouseButtonPressed) {
                    Vector2i mousePos = Mouse::getPosition(window);
                    if (playButton.isClicked(mousePos)) {
                        gameState = GAME; // Переход в игру
                        waitingToStart = true; // Ждем нажатие, чтобы начать
                    } else if (skinButton.isClicked(mousePos)) {
                        gameState = SKIN_MENU; // Переход в меню скинов
                    } else if (exitButton.isClicked(mousePos)) {
                        window.close(); // Закрытие игры
                    }
                }
            } else if (gameState == SKIN_MENU) {
                // Проверка нажатия кнопки в меню скинов
                if (event.type == Event::MouseButtonPressed) {
                    Vector2i mousePos = Mouse::getPosition(window);
                    if (backToMenuButton.isClicked(mousePos)) {
                        gameState = MAIN_MENU; // Возврат в главное меню
                    }
                }
            } else if (gameState == GAME && isGameOver) {
                // Проверка нажатия кнопки "Back to Main Menu" после Game Over
                if (event.type == Event::MouseButtonPressed) {
                    Vector2i mousePos = Mouse::getPosition(window);
                    if (backToMainMenuFromGameOver.isClicked(mousePos)) {
                        gameState = MAIN_MENU; // Возврат в главное меню
                        isGameOver = false; // Сброс состояния Game Over
                    }
                }
            }

            if (isGameOver && event.type == Event::KeyPressed) {
                isGameOver = false;
                snake.reset();
                fruit.randomize();
            }

            // Если игрок уже начал игру, обработка движений змейки
            if (waitingToStart && event.type == Event::KeyPressed) {
                waitingToStart = false; // Ожидание завершено, можно начать игру
                snake.move(isGameOver);
            }

            if (!isGameOver && event.type == Event::KeyPressed) {
                if (event.key.code == Keyboard::Up && snake.direction.y != 1)
                    snake.direction = {0, -1};
                else if (event.key.code == Keyboard::Down && snake.direction.y != -1)
                    snake.direction = {0, 1};
                else if (event.key.code == Keyboard::Left && snake.direction.x != 1)
                    snake.direction = {-1, 0};
                else if (event.key.code == Keyboard::Right && snake.direction.x != -1)
                    snake.direction = {1, 0};
            }
        }

        if (!isGameOver && timer >= delay) {
            if (!waitingToStart) { // Только если игра началась
                snake.move(isGameOver);
                timer = 0;

                if (snake.body[0].x == fruit.position.x && snake.body[0].y == fruit.position.y) {
                    snake.newBlock = true;
                    fruit.randomize();
                    snake.score++;
                }
            }
        }

        if (snake.score > snake.record) {
            snake.record = snake.score;
        }

        scoreText.setString("Score: " + std::to_string(snake.score));
        recordText.setString("Record: " + std::to_string(snake.record));

        window.clear();

        // Обработка отображения в зависимости от состояния игры
        if (gameState == MAIN_MENU) {
            window.clear(Color(175, 215, 70)); // Зеленый фон для главного меню
            playButton.draw(window);
            skinButton.draw(window);
            exitButton.draw(window);
        } else if (gameState == SKIN_MENU) {
            window.clear(Color(175, 215, 70)); // Зеленый фон для меню скинов
            backToMenuButton.draw(window);
        } else if (gameState == GAME && waitingToStart) {
            window.clear(Color(175, 215, 70)); // Зеленый фон для ожидания начала игры
            window.draw(pressToStartText); // Отображение текста "Press to Start"
        } else if (isGameOver) {
            window.clear(Color::Red); // Красный фон при окончании игры
            window.draw(gameOverText);
            backToMainMenuFromGameOver.setColor(Color::Red); // Устанавливаем красный цвет для кнопки
            backToMainMenuFromGameOver.draw(window); // Отображение кнопки для возврата в главное меню
        } else {
            // Используем фон игры
            window.clear();
            Sprite bgSprite(gameBackground);
            window.draw(bgSprite);

            fruit.draw(window);
            snake.draw(window);
            window.draw(scoreText);
            window.draw(recordText);
        }

        window.display();
    }

    return 0;
}                                        Exit");
