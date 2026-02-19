# Practical lesson pz-MQTT
# Розгортання та налаштування MQTT-брокера

> У цьому занятті студенти отримують практичні навички роботи з MQTT-брокером.  
> Мета — навчитися розгортати брокер та обмінюватись повідомленнями за допомогою публікації/підписки.

---

## Структура проєкту

```
├── stt-pz-3
│   ├── broker
│   │   ├── mosquitto.conf       # конфігурація MQTT-брокера Mosquitto
│   │   ├── docker-compose.yml   # розгортання брокера у Docker
│   ├── screenshots              # докази роботи Publish/Subscribe
│   ├── .editorconfig
│   ├── .gitignore
│   └── README.md
```

---

## Теоретична довідка — основні поняття MQTT

| Поняття | Пояснення |
|---------|-----------|
| **Topic** | Рядок-адреса повідомлення (наприклад `home/sensors/temp`). Ієрархія рівнів розділяється символом `/`. Підтримуються wildcards: `+` (один рівень) та `#` (всі підрівні). |
| **Publish** | Відправлення повідомлення брокеру на конкретний топік. |
| **Subscribe** | Підписка клієнта на топік; брокер пересилатиме усі повідомлення цього топіку підписнику. |
| **QoS** | Рівень гарантії доставки: **0** — at most once (без підтвердження), **1** — at least once (з підтвердженням), **2** — exactly once (гарантована одноразова доставка). |

---

## Розгортання

### Передумови

- Docker ≥ 24.x
- Docker Compose ≥ 2.x

### Запуск брокера

```bash
cd broker
docker compose up -d
```

Перевірити стан контейнера:

```bash
docker compose ps
docker compose logs -f mosquitto
```

Зупинити:

```bash
docker compose down
```

---

## Конфігурація брокера (`mosquitto.conf`)

```conf
listener 1883          # MQTT (TCP)
protocol mqtt

listener 9001          # MQTT over WebSockets
protocol websockets

allow_anonymous true   # анонімні підключення дозволені (для тестування)

persistence true
persistence_location /mosquitto/data/

log_type all
log_dest stdout
```

> **Увага:** `allow_anonymous true` використовується лише в цілях розробки та тестування.  
> У продакшені необхідно налаштувати автентифікацію (`password_file`) та TLS.

---

## Тестування Publish/Subscribe

### Варіант 1 — утиліти mosquitto-clients (CLI)

Встановити клієнт (Linux/macOS):

```bash
# Ubuntu/Debian
sudo apt install mosquitto-clients

# macOS
brew install mosquitto
```

**Підписатись** на топік у одному терміналі:

```bash
mosquitto_sub -h localhost -p 1883 -t "test/topic" -v
```

**Опублікувати** повідомлення у іншому терміналі:

```bash
mosquitto_pub -h localhost -p 1883 -t "test/topic" -m "Hello, MQTT!"
```

Очікуваний результат у вікні підписника:

```
test/topic Hello, MQTT!
```

### Варіант 2 — Postman (MQTT over WebSockets)

1. Відкрити Postman → **New → MQTT Request**.
2. У полі **URL** ввести: `ws://localhost:9001`.
3. Натиснути **Connect**.
4. Перейти на вкладку **Subscribe** → додати топік `test/topic` → **Subscribe**.
5. Перейти на вкладку **Publish** → Topic: `test/topic`, Body: `Hello from Postman!` → **Send**.
6. В розділі **Messages** з'явиться отримане повідомлення.

> Скриншоти результату збережені у папці `screenshots/`.

### Варіант 3 — MQTT Explorer (GUI)

1. Завантажити [MQTT Explorer](http://mqtt-explorer.com/).
2. Підключитись до `localhost:1883`.
3. Опублікувати повідомлення на будь-який топік та спостерігати за деревом топіків у реальному часі.

---

## Приклади топіків

```
home/livingroom/temperature    → 23.5
home/livingroom/humidity       → 55
home/kitchen/light             → ON
sensors/+/temperature          → wildcard: температура з будь-якої кімнати
sensors/#                      → wildcard: всі повідомлення з sensors
```

---

## QoS — приклад використання

```bash
# Публікація з QoS 1 (at least once)
mosquitto_pub -h localhost -p 1883 -t "test/qos" -m "QoS1 message" -q 1

# Підписка з QoS 2 (exactly once)
mosquitto_sub -h localhost -p 1883 -t "test/qos" -q 2 -v
```

---

## Корисні посилання

- [MQTT Essentials — HiveMQ](https://www.hivemq.com/mqtt-essentials/)
- [Eclipse Mosquitto](https://mosquitto.org/)
- [EMQX Documentation](https://www.emqx.io/docs/en/latest/)
- [MQTT with Postman](https://learning.postman.com/docs/sending-mqtt-messages/intro-to-mqtt/)
- [MQTT Explorer](http://mqtt-explorer.com/)

---

## Критерії виконання

- [x] MQTT-брокер Mosquitto розгорнуто у Docker
- [x] Мінімальна конфігурація (`mosquitto.conf`) налаштована та працює стабільно
- [x] Студент розуміє основні поняття: Topic, Publish, Subscribe, QoS
- [x] Виконано демонстрацію Publish/Subscribe через CLI та/або Postman
- [x] Усі команди, налаштування та тести описані у README.md
- [x] Скриншоти або логи дій збережені у `screenshots/`
- [x] Завдання оформлене відповідно до структури проєкту
