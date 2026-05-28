        # redis — AOF — append-only file

        Homework-шаблон для урока **l2_aof** (AOF — append-only file) на платформе Vibe Learn.

        ## Что делать

        Напиши Go-бенчмарк, который показывает trade-off `appendfsync` на реальных числах.

**Окружение:** docker-compose с Redis; конфиг fsync переключается между прогонами
через `CONFIG SET appendfsync <policy>`.

**Шаги:**
1. Для каждой политики (`always` / `everysec` / `no`): прогони N=10000 `INCR`
   команд, замерь throughput (ops/sec) и p99 latency.
2. Эмулируй потерю питания (не graceful shutdown, а резкий `docker kill --signal=KILL`
   либо обрыв в середине) в случайный момент и измерь, сколько последних команд
   не доехало до AOF при перезапуске.
3. Сформируй сравнительную таблицу: policy → throughput → p99 → lost_commands.

**CI-проверки в template repo:**
- `assert throughput[always] < throughput[everysec]` — always медленнее из-за fsync на команду.
- `assert lost_commands[always] == 0` — always не теряет (с точностью до незавершённой команды).
- `assert lost_commands[no] >= lost_commands[everysec]` — no теряет не меньше everysec.

## Контекст (из transfer-задачи урока)

У тебя two-region setup: primary Redis в EU и replica в US. На обоих включён AOF
с appendfsync everysec. CTO спрашивает: «зачем нам AOF на реплике, она же восстановится
из primary?». Объясни (a) зачем AOF на primary, (b) зачем AOF на реплике, (c) можно
ли отключить на реплике без риска.

## Recap из урока

- AOF — append-only лог всех write-команд в формате RESP. Включается `appendonly yes`.
- Главный trade-off: `appendfsync` — always (zero loss, ~10k RPS потолок), everysec (≤1с потерь, default), no (минуты потерь, max throughput).
- AOF rewrite (BGREWRITEAOF) сжимает повторяющиеся изменения в минимальный snapshot. Триггерится `auto-aof-rewrite-percentage`.
- При старте Redis грузит AOF если он есть (более свежий чем RDB). Скорость загрузки = время даунтайма.
- Поломанный AOF (crash в середине fsync) чинится `redis-check-aof --fix` — обрезает по последнему валидному фрейму.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose up -d` поднимает single-node Redis 7 на `localhost:6379` (с включёнными keyspace-notifications и AOF). Адрес переопределяется через env `REDIS_ADDR`.

        ## Запуск

        ```bash
        # Поднять локальный Redis
        docker compose up -d

        # Прогнать тесты (интеграционный включается через REDIS_INTEGRATION=1)
        go test ./...
        REDIS_INTEGRATION=1 go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
