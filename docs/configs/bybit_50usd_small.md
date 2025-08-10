## Конфиг bybit_50usd_small.json — подробная документация

Назначение: готовый конфиг для лайв‑торговли на Bybit с балансом ~$50, с консервативными параметрами (плечо 3x, более широкий шаг усреднения).

Файл: `configs/bybit_50usd_small.json`

### Структура конфига
- **backtest**: параметры бэктеста
- **bot**: логика сетки/трейлинга для long/short
- **coin_overrides**: точечные переопределения на монету
- **live**: поведение в лайве (биржа, риск, список монет)
- **optimize**: рамки и параметры оптимизатора (NSGA‑II)

---

## backtest
- **base_dir**: каталог для результатов бэктеста (по умолчанию `backtests`).
- **combine_ohlcvs**: объединять 1m‑данные различных бирж для лучшего покрытия. `true`/`false`.
- **compress_cache**: сжимать кеш OHLCV на диск (экономит место, дольше загрузка).
- **end_date**: конец периода, например `"now"`.
- **exchanges**: источники минутных свечей, например `["binance", "bybit"]`.
- **gap_tolerance_ohlcvs_minutes**: допустимый разрыв данных в минутах.
- **start_date**: начало периода бэктеста.
- **starting_balance**: стартовый баланс в USD (здесь 50).
- **use_btc_collateral**: бэктест в BTC‑коллатерале (прибыль наращивает BTC, убыток — USD‑долг).

Пример запуска бэктеста:

```bash
python src/backtest.py configs/bybit_50usd_small.json -dp
```

---

## bot.long — торговая логика (лонг)
- **ema_span_0 / ema_span_1**: периоды EMA (в минутах). Из них формируется EMA‑пояс для начальных входов и unstuck‑закрытий.
- **entry_initial_ema_dist**: смещение первого входа от EMA‑пояса (отрицательное — «дешевле»). Например `-0.01`.
- **entry_initial_qty_pct**: доля от лимита экспозиции на первый вход (0..1). Например `0.4`.
- **entry_grid_spacing_pct**: базовый шаг между усреднениями от средней цены позиции. Здесь `0.03` (3%).
- **entry_grid_spacing_weight**: «растяжка» шага в зависимости от заполнения экспозиции (>0 — шаг растёт).
- **entry_grid_double_down_factor**: множитель размера следующего усреднения. <1 сдерживает рост объёма.
- **entry_trailing_* (threshold_pct / retracement_pct / grid_ratio / double_down_factor)**: трейлинг‑входы вместо фиксированных уровней.
- **close_grid_markup_start / close_grid_markup_end**: диапазон наценок к средней цене позиции для постановки ТП‑ордеров.
- **close_grid_qty_pct**: доля «полного» размера позиции на один ТП‑ордер.
- **close_trailing_***: трейлинговые закрытия (аналогично entry_trailing_*, но для выхода).
- **n_positions**: максимальное число одновременных позиций на сторону. Здесь `1`.
- **total_wallet_exposure_limit**: суммарный лимит экспозиции на сторону (в «единицах баланса»). Здесь `1.0`.
- **enforce_exposure_limit**: при превышении лимита — частично сокращать позицию.
- **filter_noisiness_rolling_window / filter_volume_rolling_window**: окна (мин) для расчёта «шумности»/объёма (forager).
- **filter_volume_drop_pct**: отсечь нижний процент монет по объёму. Здесь `0.25` (отбрасывает 25% низколиквидных).
- **unstuck_* (threshold / close_pct / ema_dist / loss_allowance_pct)**: параметры «разморозки» застрявших позиций.

Пример: цена упала на 3% → бот ставит усреднение (entry_grid_spacing_pct). После заполнения ТП‑сетка пересчитывается в заданном диапазоне markup.

## bot.short — торговая логика (шорт)
Параметры зеркальны `long`. В конфиге шорт отключён: `n_positions: 0`, `total_wallet_exposure_limit: 0`.

Чтобы включить шорт, увеличьте `n_positions` и задайте безопасные лимиты/шаги аналогично лонгу.

---

## coin_overrides — переопределения на монету
Позволяет менять отдельные параметры для конкретных активов.

Пример:

```json
{
  "coin_overrides": {
    "BTC": {
      "bot": { "long": { "entry_grid_spacing_pct": 0.02 } },
      "live": { "leverage": 5 }
    }
  }
}
```

---

## live — настройки лайва
- **approved_coins**: разрешённые монеты. Может быть массивом, объектом `{long:[], short:[]}` или путём к файлу. В конфиге задан список для long.
- **ignored_coins**: исключённые монеты (аналогично формату выше).
- **empty_means_all_approved**: если `true` и список пуст — значит «все разрешены».
- **leverage**: кредитное плечо на бирже. Здесь `3`. Это технический потолок маржи; риск управляется через WE‑лимиты.
- **market_orders_allowed**: разрешить маркет при близкой цене.
- **max_n_cancellations_per_batch / max_n_creations_per_batch**: лимиты на отмены/создания ордеров за один цикл.
- **execution_delay_seconds**: задержка после отправки ордера на биржу.
- **time_in_force**: политика ордеров (например, `good_till_cancelled`).
- **price_distance_threshold**: минимальная дистанция до цены для EMA‑ордеров.
- **mimic_backtest_1m_delay**: переоценка ордеров раз в минуту (для соответствия бэктесту).
- **minimum_coin_age_days**: минимальный «возраст» монеты.
- **ohlcvs_1m_rolling_window_days / ohlcvs_1m_update_after_minutes**: окна и период обновления 1m‑данных.
- **pnls_max_lookback_days**: история PnL.
- **forced_mode_long / forced_mode_short**: принудительные режимы: `m` (manual), `gs` (graceful_stop), `t` (tp_only), `p` (panic), либо пусто (normal).
- **auto_gs**: недопущенная монета → `graceful_stop`.
- **user**: имя учётки из `api-keys.json`. Важно: ключ верхнего уровня в `api-keys.json` должен совпадать.
- **filter_by_min_effective_cost**: отсекать монеты, где минимальный допустимый размер ордера на бирже выше предполагаемого.

Быстрый запуск лайва:

```bash
python src/main.py configs/bybit_50usd_small.json
```

Через Docker Compose (пример):

```yaml
passivbot-live:
  command: ["python", "src/main.py", "configs/bybit_50usd_small.json"]
```

```bash
docker compose --profile live up -d --build
```

---

## optimize — оптимизация параметров (NSGA‑II)
- **bounds**: диапазоны для подбора (ключи соответствуют полям из `bot.long/short`).
- **limits**: ограничения/штрафы (например, по просадке/стагнации).
- **scoring**: целевые метрики (например, `btc_adg_w`, `btc_mdg_w`).
- **population_size / iters / crossover_probability / mutation_probability**: настройки генетического алгоритма.
- **round_to_n_significant_digits**: округление параметров при поиске.

Запуск оптимизации:

```bash
python src/optimize.py configs/bybit_50usd_small.json --start configs/examples/
```

---

## Рекомендации под баланс ~$50
- Держите `bot.long.n_positions = 1` и `total_wallet_exposure_limit ≤ 1.0`.
- Для более редких усреднений увеличивайте `entry_grid_spacing_pct` (уже 0.03), но учитывайте большую просадку до выхода.
- При желании торговать более чем одной монетой повышайте `n_positions` и проверяйте, хватает ли минимального размера ордеров (min_cost) на бирже.


