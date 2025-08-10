## Справочник по файлам и модулям

### Корень
- `README.md` — обзор проекта
- `docker-compose.yml` — сервисы (`passivbot`, `passivbot-live`, `dozzle`)
- `Dockerfile`, `Dockerfile_live` — сборка образов (dev/live), установка Rust, maturin
- `requirements*.txt` — зависимости (общие/для live/Rust‑сборки)
- `api-keys.json.example` — пример ключей

### `src/`
- `main.py` — проверяет/собирает Rust‑расширение; запускает `passivbot.main()`
- `passivbot.py` — лайв‑логика: цикл исполнения, выбор монет, ордер‑менеджмент, риск
- `config_utils.py` — чтение, преобразование, валидация конфигов
- `procedures.py` — утилиты (часы, файлы, загрузка `api-keys.json`, брокер‑коды)
- `backtest.py` — подготовка/анализ бэктеста, вызовы Rust
- `optimize.py` — DEAP/NSGA‑II оптимизация
- `downloader.py` — менеджер OHLCV: загрузка, кеширование, выбор лучшего источника
- `exchanges/*.py` — адаптеры бирж
- `pure_funcs.py` — численные/вспомогательные функции
- `njit_*.py` — Numba‑ускоренные ядра (расчёт «шумности» и т.п.)
- `plotting.py`, `interactive_plot.py` — графики/интерактив

### `passivbot-rust/`
- `src/*.rs` — ядро бэктеста/логики на Rust (entries/close/trailing/types/utils)
- `Cargo.toml` — конфигурация Rust проекта

### `configs/`
- Готовые конфиги (пример: `bybit_50usd_small.json`) и шаблон `template.json`
- Папка `examples/` — примеры для старта/демо

### `docs/`
- `guide_ru.md` — руководство (RU)
- `configuration.md`, `backtesting.md`, `optimizing.md`, `live.md` — тематические главы
- `handbook/` — расширенная документация (архитектура, конфиги, лайв‑логика и т.д.)


