[Гитхаб страница оригинальной библиотеки](https://github.com/reo7sp/tgbot-cpp)

В целом, все шаги есть на ней, но можно посмотреть тут. 

  

### Для начала, нам нужно скачать линух и к нему компилятор плюсов(g++), с этим вы должны справиться сами. 

#### Вводим в терминал команду 
	sudo apt-get install g++ make binutils cmake libboost-system-dev libssl-dev zlib1g-dev libcurl4-openssl-dev

  В целом эта штука должны установить все зависимости, которые должны нам помочь

#### Дальше копируем себе репо с помощью команды 
	git clone https://github.com/reo7sp/tgbot-cpp
#### Переходим в папку с нашим ботом
	cd tgbot-cpp
####  Создаем файлы проекта
	cmake .
#### Собираем проект так, чтобы он работал на 4 потоках(?)
    make -j4
#### Устанавливаем библиотеку
    sudo make install

Дальше можем проверить, все ли получилось сделать.

	/usr/local/include/tgbot/
По этому пути должны лежать все заголовочные файлы
## Проверить, все ли установилось
создаем файлик 
```bash
vim telegram_bot.cpp
```
В него закидываем код:
```cpp
#include <stdio.h>
#include <tgbot/tgbot.h>

int main() {
    TgBot::Bot bot("PLACE YOUR TOKEN HERE");
    bot.getEvents().onCommand("start", [&bot](TgBot::Message::Ptr message) {
        bot.getApi().sendMessage(message->chat->id, "Hi!");
    });
    bot.getEvents().onAnyMessage([&bot](TgBot::Message::Ptr message) {
        printf("User wrote %s\n", message->text.c_str());
        if (StringTools::startsWith(message->text, "/start")) {
            return;
        }
        bot.getApi().sendMessage(message->chat->id, "Your message is: " + message->text);
    });
    try {
        printf("Bot username: %s\n", bot.getApi().getMe()->username.c_str());
        TgBot::TgLongPoll longPoll(bot);
        while (true) {
            printf("Long poll started\n");
            longPoll.start();
        }
    } catch (TgBot::TgException& e) {
        printf("error: %s\n", e.what());
    }
    return 0;
}
```
Дальше выходим из vim'а, нажимаем esc и пишем :wq
#### Компилим файл
```sh
g++ telegram_bot.cpp -o telegram_bot --std=c++14 -I/usr/local/include -lTgBot -lboost_system -lssl -lcrypto -lpthread
```
## Также можно собрать проект через CMake
#### Создаем CMakeLists
	
	vim CMakeLists.txt
 Внутри пишем это:
```CMake
cmake_minimum_required(VERSION 3.0)
project(TelegramBot)

# Установка стандарта языка C++
set(CMAKE_CXX_STANDARD 20)

# Добавление исходных файлов
add_executable(telegram_bot telegram_bot.cpp)

# Добавление директивы для поиска заголовочных файлов
target_include_directories(telegram_bot PRIVATE /usr/local/include)

# Добавление зависимостей
target_link_libraries(telegram_bot PRIVATE TgBot boost_system ssl crypto pthread)
```
#### Выоходим из нее 
	:wq
#### После создаем папку build:
	mkdir build
 	cd build
#### Внутри нее запускаем cmake:
	 cmake ..
#### После запускаем сборку проекта:
	make
#### На выходе также получаем файлик, чтобы его запустить пишем:
	 ./telegram_bot  
Чтобы проверить работоспособность, напишите что угодно этому придурку - @yapi_test_bot
