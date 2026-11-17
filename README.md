# NASA NEO Tracker (Lab 3)

A desktop app (Python + Tkinter) that shows near-Earth asteroids for a chosen date, using NASA's free [NeoWs](https://api.nasa.gov/) API. It is built with spec-driven development (GitHub Spec Kit → Google Antigravity) and follows SOLID principles.

![Screenshot](docs/screenshot.png)

## Features

- **Animated radar**: Earth in the centre, a rotating sweep beam and twinkling stars. Asteroids are placed on a logarithmic distance scale (the dashed ring labelled `1 LD` is the Moon's orbit); marker size reflects diameter; potentially hazardous asteroids are red and pulse. Hover for details, click to select.
- **Summary cards**: total found, potentially hazardous count, closest / fastest / largest asteroid.
- **Table**: name, diameter (m), velocity (km/s), distance (km and lunar distances), hazard flag. Hazardous rows are highlighted and linked to the radar.
- Date field (`YYYY-MM-DD`) with ◀ ▶ buttons for the previous/next day.
- Shows the data source, the data date and the response timestamp.

## Behaviour

| Situation | What happens |
|---|---|
| Invalid date (empty, wrong format, non-existent such as `2026-02-31`) | Error message, no request is made |
| No network / timeout | Message + last successfully cached day is shown with a warning |
| Empty response for the date (no data) | Last cached day is shown with a warning |
| API rate limit, bad key, server error, malformed JSON | Message + cached data if available |
| Some asteroid records are malformed | They are skipped and their number is reported |
| Successful non-empty response | Saved to `cache.json`; survives app restarts |

The app never crashes on network or data errors; requests run in a background thread so the UI stays responsive.

## Requirements

- Python 3.9+ with Tkinter (bundled on Windows/macOS; Debian/Ubuntu: `sudo apt install python3-tk`)
- No third-party packages needed

## Run

```bash
python main.py
```

The app uses NASA's `DEMO_KEY` by default, which has a strict rate limit. For regular use, get a free key at <https://api.nasa.gov> and set it:

```bash
export NASA_API_KEY=your_key      # Linux / macOS / Git Bash
set NASA_API_KEY=your_key         # Windows cmd
```

## Tests

```bash
python -m unittest
```

Tests need no network, display or disk access beyond temporary files. They cover response parsing, empty/invalid responses, HTTP errors, date validation, caching (including restart survival), the fallback logic, radar geometry, statistics and formatting.

## Architecture and SOLID

```
main.py                    Composition root: the only place that picks concrete classes
neo_tracker/
  errors.py  models.py     Exceptions and data models
  validation.py            Validator        → DateValidator
  http_client.py           HttpClient       → UrllibHttpClient
  parser.py                FeedParser       → NeoWsFeedParser
  source.py                DataSource       → NasaNeoWsSource
  cache.py                 Cache            → JsonFileCache
  service.py               AsteroidProvider → NeoService (source → cache → fallback)
  presentation.py          Formatter        → AsteroidFormatter
                           StatsProvider    → StatsCalculator
  radar_layout.py          Layout           → RadarLayout (pure geometry)
  theme.py                 Palette and colour blending
  widgets.py               StatCard, GlowButton, RadarCanvas
  gui.py                   App (depends only on abstractions)
tests/                     Unit tests using fakes instead of network/disk
docs/                      Spec draft and screenshot
```

| Principle | How it is applied |
|---|---|
| **S** – Single responsibility | Validation, HTTP, parsing, data source, cache, orchestration, formatting, layout and UI are separate classes/modules |
| **O** – Open/closed | A new data source, cache (e.g. SQLite) or HTTP client is a new class; `NeoService` and `App` stay unchanged |
| **L** – Liskov substitution | Test fakes (`FakeSource`, `InMemoryCache`, `FakeHttp`) replace the real implementations without changing behaviour |
| **I** – Interface segregation | Small interfaces: `HttpClient.get`, `DataSource.fetch`, `Cache.store/get/latest`, `AsteroidProvider.load`, `Validator.validate`, `Layout.compute` |
| **D** – Dependency inversion | `NeoService` depends on `DataSource` and `Cache`; `App` depends on `AsteroidProvider`, `Validator`, `Formatter`, `StatsProvider`, `Layout`; wiring happens only in `main.py` |

## Spec-driven development

The spec-first workflow (constitution → specification → plan → tasks) is documented in `docs/spec-draft.md`, which was used as input for GitHub Spec Kit. See `REPORT.md` for the lab report.
