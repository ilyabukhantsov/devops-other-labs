Звіт з лабораторної роботи №2
Тема: Контейнеризація застосунків

Виконав: Буханцов Ілля

Етапи виконання можна побачити в lab2/docs, там буде видно етапи виконання

Об'єкт дослідження: Python, Go, Docker, Multi-stage builds, DNS resolution (musl vs glibc).
1. Дослідження Python Application
Експеримент 1: Базовий образ (Debian-based)

Використовувався образ python:3.7.

    Dockerfile: Стандартна копіювання залежностей та коду.

    Час першої збірки: 49.4s.

    Розмір образу: 1.51GB (Content: 386MB).

Після додавання коментаря в app.py, час перезбірки склав 4.4s, оскільки шари з залежностями були закешовані.
Експеримент 2: Оптимізація розміру (Slim/Alpine)

Використано полегшену версію Python.

    Результат: Розмір впав до 244MB (Content: 58.6MB).

    Час збірки: 25.2s.

Експеримент 3: Додавання залежностей (Numpy)

Додано бібліотеку numpy та ендпоінт для множення матриць.

    Результат: Розмір образу зріс до 344MB, час збірки — 33.4s.

2. Дослідження Golang Application та Multi-stage builds
Підхід 1: Single-stage build

Використовувався образ golang:1.22.

    Проблема: В образі залишаються вихідний код, інструменти збірки та кеш Go.

    Розмір: ~530MB.

    Вміст: Зайві файли (src, pkg), що збільшують поверхню атаки та розмір.

Підхід 2: Multi-stage build (scratch)

Використано розділення на builder та фінальний мінімальний образ.
Dockerfile

FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o fizzbuzz

FROM scratch
COPY --from=builder /app/fizzbuzz /fizzbuzz
ENTRYPOINT ["/fizzbuzz", "serve"]

    Результат: Розмір образу — 16.7MB.

    Аналіз: В образі знаходиться лише бінарний файл.

    Труднощі: У scratch відсутній shell (sh), тому неможливо зробити docker exec -it. Для дебагу це незручно, але для продакшену — ідеально.

3. Порівняння Musl (Alpine) vs glibc (Ubuntu)
Хід експерименту

    Запущено DNS-сервер dnsmasq з кастомною адресою myservice.internal.corp -> 10.0.0.50.

    Перевірка через getent hosts в Ubuntu та Alpine з параметром --dns-search="corp".

Результати та висновки

    Ubuntu (glibc): Резолвер використовує NSS (Name Service Switch). При запиті myservice.internal він автоматично додає суфікс з search-list і отримує myservice.internal.corp, що успішно резолвиться.

    Alpine (musl): Резолвер у musl працює інакше. Він може ігнорувати певні комбінації search domains або робити паралельні запити (A та AAAA).

    Логи DNS: Помітно, що клієнт робить запити на NXDOMAIN (неіснуючі домени) перед тим, як знайти правильний.

Ризики: Різна поведінка резолверів може призвести до того, що сервіс працює в розробці (Ubuntu), але "не бачить" базу даних у продакшені (Alpine/k8s).
4. Практична частина
Контейнеризація Java-застосунку

Для пакування сервісу було обрано базовий образ Eclipse Temurin (дистрибутив OpenJDK від Adoptium), оскільки він забезпечує стабільну роботу Java 21 на базі Ubuntu Jammy.
Dockerfile аналіз:
Dockerfile

FROM eclipse-temurin:21-jdk-jammy

# Створення не-привілейованого користувача для безпеки
RUN useradd -m -d /home/app -s /usr/sbin/nologin app

WORKDIR /home/app

# Копіювання зібраного jar-файлу (артефакту)
COPY target/mywebapp.jar app.jar
RUN chown app:app app.jar

# Запуск від імені створеного користувача
USER app

EXPOSE 5200

ENTRYPOINT ["java","-jar","app.jar"]

Ключові особливості реалізації:

    Безпека (Non-root user): Застосунок запускається від імені користувача app. Це критично важливо: у разі зламу застосунку зловмисник не отримає прав root у контейнері.

    Оптимізація прав: Використання chown app:app гарантує, що застосунок має права на читання/виконання свого файлу без зайвих дозволів у системі.

    Версійність: JDK 21 забезпечує підтримку сучасних фіч (наприклад, Virtual Threads), що актуально для високонавантажених сервісів.

Автоматизація розгортання (Docker Compose)

Для локального запуску всієї інфраструктури (застосунок + база даних + проксі) було створено файл docker-compose.yml.
YAML

services:
  app:
    build: .
    ports:
      - "5200:5200"
    environment:
      - DB_URL=jdbc:postgresql://db:5432/myapp
      - DB_USER=user
      - DB_PASSWORD=pass
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend

  db:
    image: postgres:15-alpine
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  db_data:

### Висновки: 
1. Я бачу велику різницю в послідовності команд, ми можемо оптимізувати процесс і зменшити часу на build стадію
2. Я бачу ефективність використання легкий образів систем, хоч і є іноді проблеми, але часто це банально ефективніше
3. Ми бачимо більш легку працю, якщо порівнювати з VM