# Patna live forecast log

An automated, prospective test of the frozen forecasting models from [weather-forecasting-patna](https://github.com/shashwatsrv/weather-forecasting-patna).

Every hour, a GitHub Actions workflow runs the frozen LightGBM models for the ERA5 grid cell at 25.5°N, 85.25°E (about 15 km from central Patna). It logs their 1h, 6h and 24h forecasts of temperature and rain, along with:
- the best simple baselines,
- the ECMWF IFS and GFS forecasts for the same hours, as issued at that moment.

Once a week the log is scored against ERA5, which is published about 5 days behind real time.

**This is an experiment, not a weather service.** Don't use these numbers for decisions.

## Files

- `data/live/forecast_log.csv`: one row per issue hour (`issue_utc`) and horizon (`h`). Forecasts are valid at `valid_utc` / `valid_ist`. Columns:
  - `lgbm_*`: LightGBM temperature (°C), rain probability, and expected rain amount (mm).
  - `base_*`: the best baseline per horizon, frozen in the main project.
  - `ecmwf_ifs025_*`, `gfs_seamless_*`: the weather-service temperature and rain amount for the same valid hour.
- `data/live/live_scores.csv`: the latest scores against ERA5, per horizon (written once there is something to score).
- `data/live/logger.log`: one status line per run.
- `data/models/`: the frozen final models, trained on 2011-2024. Re-fitting them reproduces the main project's 2025 test predictions.

## How to read the results

- **Comparing like with like:** the models were trained on ERA5. Live, their inputs come from the ECMWF IFS feed for the same grid cell, because ERA5 isn't available in real time. Expect live skill to be lower than in the backtest.
- **IFS has an advantage:** it comes from the same model family as ERA5, the data the scores are measured against.
- **Rain scoring for the weather services:** their rain is a single amount, not a probability, so their Brier score treats a forecast of at least 0.1 mm as 1 and anything less as 0.
- **How runs are started:** an external scheduler (cron-job.org) starts the workflow every hour at about :30 UTC through GitHub's API, using a token that can only start this repo's workflows. GitHub's own schedule (every 15 minutes) stays on as a backup. It's unreliable for a small repo: on the first two days it started only 4 of about 90 tries. Only the first successful run in each hour logs a forecast, so extra runs log nothing.
- **Gaps:** the hours before 27 November 2026 10:00 UTC have big gaps (7 of 48 covered). An hour that is missed stays missing, because a forecast can't be logged after the fact. Every forecast is scored on the same hours.
- **Pinned code:** the workflow runs the main project's code at commit `eb987d5`, so the live test always uses the frozen code.

## Data and attribution

- [Weather data by Open-Meteo.com](https://open-meteo.com/), licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The live inputs and the weather-service forecasts (ECMWF IFS, NOAA GFS) come from the Open-Meteo Forecast API. The scoring truth is ERA5 from the Open-Meteo Historical Weather API.
- The models and the climatology baselines were derived from ERA5. Contains modified Copernicus Climate Change Service information 2026. Neither the European Commission nor ECMWF is responsible for any use that may be made of the Copernicus information or data it contains. ERA5 dataset: https://doi.org/10.24381/cds.adbb2d47
- Neither Open-Meteo, ECMWF, NOAA nor the Copernicus programme endorses this project.
