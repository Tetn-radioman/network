# network
---
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
## подключение
```cmake
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core Network)// <---здесь

target_link_libraries(TC PRIVATE
        Qt${QT_VERSION_MAJOR}::Core
        Qt${QT_VERSION_MAJOR}::Network// <---здесь
        pthread
    )
```
## код showip hostname
Этот код основан на примерах из **Beej's Guide to Network Programming** 
(https://beej.us/guide/bgnet/).
```cpp

#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <arpa/inet.h>
#include <netinet/in.h>

int main(int argc, char *argv[])
{
    struct addrinfo hints, *res, *p;
    int status;
    char ipstr[INET6_ADDRSTRLEN];


    if (argc != 2) {
        fprintf(stderr,"usage: showip hostname\n");
        return 1; // если эта ошибка значит вы не передаёте аргумент например как здесь ./showip google.com
    }

    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_UNSPEC; // AF_INET или AF_INET6 если требуется
    hints.ai_socktype = SOCK_STREAM;

    if ((status = getaddrinfo(argv[1], NULL, &hints, &res)) != 0) {
        fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(status));
        return 2;
    }

    printf("IP addresses for %s:\n\n", argv[1]);

    for(p = res;p != NULL; p = p->ai_next) {
        void *addr;
        char *ipver;

        // получить
        // в IPv4 и IPv6 поля разные
        if (p->ai_family == AF_INET) { //                                   IPv4
            struct sockaddr_in *ipv4 = (struct sockaddr_in *)p->ai_addr;
            addr = &(ipv4->sin_addr);
            ipver = "IPv4";
        } else { //                                                         IPv6
            struct sockaddr_in6 *ipv6 = (struct sockaddr_in6 *)p->ai_addr;
            addr = &(ipv6->sin6_addr);
            ipver = "IPv6";
        }

        // Перевести IP в строку и вывести
        inet_ntop(p->ai_family, addr, ipstr, sizeof ipstr);
        printf("  %s: %s\n", ipver, ipstr);
    }

    freeaddrinfo(res); // освободить связанный список

    return 0;
}

```
