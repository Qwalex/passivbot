## Конфигурация: полный разбор

Эта глава описывает все ключи конфигов v7: `backtest`, `bot`, `live`, `coin_overrides`, `optimize`.

### backtest
- `base_dir`: каталог результатов
- `combine_ohlcvs`: объединять источники OHLCV
- `compress_cache`: сжатие кеша на диск
- `end_date` / `start_date`: период теста
- `exchanges`: список источников 1m OHLCV
- `starting_balance`: стартовый баланс (USD)
- `use_btc_collateral`: режим BTC‑коллатераля

### bot.long / bot.short
Общие поля:
- `ema_span_0`, `ema_span_1`: периоды EMA; используется EMA‑пояс
- `entry_initial_ema_dist`: смещение первого входа от EMA‑пояса
- `entry_initial_qty_pct`: доля лимита WE на первый вход
- `entry_grid_spacing_pct`: базовый шаг между усреднениями от средней цены
- `entry_grid_spacing_weight`: «растяжка» шага по мере набора позиции
- `entry_grid_double_down_factor`: множитель размеров усреднений
- `entry_trailing_*`: трейлинг‑входы (threshold/retracement/grid_ratio/double_down_factor)
- `close_grid_markup_start`, `close_grid_markup_end`: диапазон ТП‑сетки
- `close_grid_qty_pct`: доля позиции на один ТП
- `close_trailing_*`: трейлинг‑закрытия
- `n_positions`: максимум позиций на сторону
- `total_wallet_exposure_limit`: суммарный лимит экспозиции на сторону
- `enforce_exposure_limit`: принудительно держать WE ≤ лимит
- `filter_noisiness_rolling_window`, `filter_volume_rolling_window`: окна для forager
- `filter_volume_drop_pct`: отсекание нижней доли по объёму
- `unstuck_threshold`, `unstuck_close_pct`, `unstuck_ema_dist`, `unstuck_loss_allowance_pct`

Практика:
- Меньший депозит → выше `entry_grid_spacing_pct`, меньше `entry_initial_qty_pct`, `n_positions=1`.
- Хотите больше инструментов → увеличьте `n_positions` и проверьте min_cost на бирже.

### coin_overrides
Точечные изменения на монету.
Пример:
```json
{
  "coin_overrides": {
    "ADA": { "bot": { "long": { "entry_initial_qty_pct": 0.2 } } }
  }
}
```

### live
- `approved_coins` / `ignored_coins`: списки (поддерживается разделение на long/short и путь к файлу)
- `empty_means_all_approved`: пустой approved = все разрешены
- `leverage`: плечо на бирже (технический потолок маржи)
- `market_orders_allowed`: разрешить маркет при близкой цене
- `max_n_cancellations_per_batch` / `max_n_creations_per_batch`
- `execution_delay_seconds`, `time_in_force`, `price_distance_threshold`
- `mimic_backtest_1m_delay`
- `minimum_coin_age_days`
- `ohlcvs_1m_rolling_window_days`, `ohlcvs_1m_update_after_minutes`
- `pnls_max_lookback_days`
- `forced_mode_long`, `forced_mode_short`: m, gs, t, p, ""
- `auto_gs`: недопущенные → graceful_stop
- `user`: имя в `api-keys.json`
- `filter_by_min_effective_cost`: фильтр по минимальному размеру ордера

### optimize (NSGA‑II)
- `bounds`: диапазоны для поиска
- `limits`: штрафы/ограничения (просадка, стагнация и т.д.)
- `scoring`: метрики целевых функций
- `n_cpus`, `population_size`, `iters`, `crossover_probability`, `mutation_probability`
- `round_to_n_significant_digits`, `compress_results_file`, `write_all_results`

Пример запуска:
```bash
python src/optimize.py configs/template.json --start configs/examples/
```

