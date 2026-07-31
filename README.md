# network

### здесь я буду писать свои заметки по изучению сетевого програмиирования
---

уровни полнофункциональной модели для сетевых примеров:
• Приложений
• Представления
• Сессии
• Транспортный
• Сетевой
• Данных
• Физический

Более совместимая с Unix модель уровней может быть такой:
• Уровень приложений (telnet, ftp и т. д.)
• Уровень транспорта хост-хост (TCP, UDP)
• Уровень интернета (IP и маршрутизация)
• Уровень доступа к сети (Ethernet, wi-fi и т.п.)

---
### мини примеры

```cmake
cmake_minimum_required(VERSION 3.14)

project(TS LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core Network)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core Network)

add_executable(TS
  main.cpp
)
target_link_libraries(TS PRIVATE
        Qt${QT_VERSION_MAJOR}::Core
        Qt${QT_VERSION_MAJOR}::Network
        pthread
    )

include(GNUInstallDirs)
install(TARGETS TS
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

```

#### сервер

```cpp
#include <QCoreApplication>
#include <QTcpServer>
#include <QTcpSocket>
#include <QDebug>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    QTcpServer server;
    // 1. Запускаем сервер на всех интерфейсах и порту 12345
    if (!server.listen(QHostAddress::Any, 12345)) {
        qDebug() << "Server could not start!";
        return -1;
    }
    qDebug() << "Server started on port 12345";

    // 2. Обрабатываем сигнал о новом подключении
    QObject::connect(&server, &QTcpServer::newConnection, [&]() {
        // 3. Принимаем подключение
        QTcpSocket *client = server.nextPendingConnection();
        qDebug() << "Client connected!";

        // 4. Обрабатываем сигнал о получении данных от этого клиента
        QObject::connect(client, &QTcpSocket::readyRead, [client]() {

            // 5. Читаем все полученные данные
            QByteArray data = client->readAll();
            qDebug() << "Received:" << data;

            // 6. Отправляем данные обратно (эхо)
            client->write(data);
        });

        // 7. Очищаем ресурсы при отключении клиента
        QObject::connect(client, &QTcpSocket::disconnected, client, &QTcpSocket::deleteLater);
    });

    return a.exec();
}
```
#### клиент
```cpp
#include <QCoreApplication>
#include <QTcpSocket>
#include <QDebug>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    QTcpSocket socket;

    // 1. Обрабатываем успешное подключение
    QObject::connect(&socket, &QTcpSocket::connected, [&]() {
        qDebug() << "Connected to server!";
        // 3. Отправляем сообщение
        socket.write("Hello, server!");
    });

    // 2. Обрабатываем получение ответа от сервера
    QObject::connect(&socket, &QTcpSocket::readyRead, [&]() {
        QByteArray data = socket.readAll();
        qDebug() << "Server said:" << data;
        socket.disconnectFromHost();
    });

    // 4. Подключаемся к серверу
    socket.connectToHost("127.0.0.1", 12345); // 127.0.0.1 - это localhost

    return a.exec();
}

```
