## Установка и запуск

### Быстрая установка (локально)
```bash
git clone https://github.com/enarjord/passivbot.git
cd passivbot
python -m venv venv
source venv/bin/activate  # Windows: venv/Scripts/activate
pip install -r requirements.txt
cp api-keys.json.example api-keys.json
# заполните api-keys.json своими ключами
```

### Запуск лайва
```bash
python src/main.py -u bybit_01                 # по user из api-keys.json
python src/main.py configs/bybit_50usd_small.json  # по конкретному конфигу
```

### Запуск в фоне (Linux / Git Bash)
```bash
nohup python src/main.py configs/bybit_50usd_small.json > logs/live.log 2>&1 & disown
tail -f logs/live.log
```

### Docker / Docker Compose
`docker-compose.yml` содержит сервис `passivbot-live`.

Пример:
```yaml
passivbot-live:
  build:
    context: .
    dockerfile: Dockerfile_live
  volumes:
    - ./configs/:/app/configs/
    - ./api-keys.json:/app/api-keys.json
  command: ["python", "src/main.py", "configs/bybit_50usd_small.json"]
```

Сборка и запуск:
```bash
docker compose --profile live up -d --build
docker compose logs -f passivbot-live
```

### Логи через Dozzle (опционально)
В `docker-compose.yml` предусмотрен `dozzle`.
- Открыть: `http://<host>:3007/` или через nginx по префиксу `/dozzle/` (см. DOZZLE_BASE)

### Структура api-keys.json
```json
{
  "bybit_01": { "exchange": "bybit", "key": "...", "secret": "..." }
}
```
`-u bybit_01` должен совпадать с ключом верхнего уровня.

### Частые ошибки
- JSONDecodeError для `api-keys.json`: проверьте двойные кавычки, отсутствие лишних запятых, комментариев.
- 429 при сборке Docker: используйте зеркало базового образа (`mirror.gcr.io`) или `docker login`.
- Порт занят (Dozzle): измените порт, либо остановите занявший процесс.


