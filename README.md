<div align="center">

# CrowdSense

**Crowd-aware multimodal journey planning for Indian transit**

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white)
![Tests](https://img.shields.io/badge/tests-Pytest%20%7C%20Vitest%20%7C%20Playwright-brightgreen)

[Live Prototype](https://crowdsense.ujjwalraj.online) · [Demo Video](https://drive.google.com/file/d/1nwtZYWTwHembOtgsM6kC1fRYLB7lRSQS/view?usp=sharing) · [Source Code](https://github.com/Molly-Arora02/Crowdsense)

</div>

---

## Overview

CrowdSense helps commuters choose routes that are not only fast but comfortable, by combining GTFS schedule data with crowd occupancy estimates. It also includes an Operations Command Center (OCC) for transit operators, with corridor density monitoring and human-in-the-loop advisory actions.

![CrowdSense home](screenshots/01_homepage.png)

## Table of Contents

- [Features](#features)
- [Technical Note](#technical-note)
- [Screenshots](#screenshots)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [API Overview](#api-overview)
- [Testing](#testing)
- [Documentation](#documentation)
- [Author](#author)

## Features

### For commuters
- **Multimodal routing** across Indian Railways and metro networks (e.g. Delhi Metro, with intermodal walking transfers), using time-dependent GTFS data and an A\* search.
- **Three route alternatives:** Fastest, Balanced and Comfort, controlled by a `comfort_weight`.
- **Station search** with autocomplete and one-tap presets for popular interchanges.
- **Itinerary view** with mode badges, platform info, fare estimates and CO₂ savings versus driving.
- **Interactive map** (Leaflet + OpenStreetMap) with colour-coded transit modes, dashed walking links and crowdedness levels (green / yellow / red / purple).
- **Honest data labelling:** every occupancy figure is tagged `live`, `predicted` or `historical` and shown as an estimate, never a guarantee.
- Preferences and recent/saved journeys stay in your browser's local storage.

### For operators (OCC)
- Corridor density monitoring (`GET /api/v1/occ/occupancy/corridor`).
- Alert engine for overcrowded segments.
- Recommendation workflow: review, approve, reject or defer, with a CSV-exportable audit trail. Advisory actions are recorded locally and never publish or change real transport operations.

> **Data integrity:** the UI never fabricates results or schedules. Missing data is shown as unavailable, and development fixtures are clearly labelled as estimated.

## Technical Note

- **Routing:** GTFS data (Delhi Metro, Indian Railways, combined multimodal set) is loaded into an in-memory network cache. A time-dependent A\* search, with a spatial index for walking transfers between modes, returns Fastest, Balanced and Comfort alternatives ranked by `comfort_weight`, which trades travel time against crowding.
- **Crowd data:** Every occupancy value is tagged `live`, `predicted` or `historical`. If no prediction source is connected, crowd data is marked unavailable rather than invented. Providers sit behind an integration layer so real feeds can be plugged in.
- **Operations:** An alert engine flags overcrowded corridors and generates recommendations. They go through a human-in-the-loop review, approve, reject or defer workflow, and each step is written to an exportable audit log.
- **Persistence:** SQLAlchemy 2 with SQLite by default (PostgreSQL supported). Writes use batch upserts, so they are idempotent. Tables are created at startup; versioned migrations are not set up yet.
- **Frontend:** React 18 + Vite, with a Leaflet map. Preferences and saved journeys stay in browser local storage.

## Screenshots

| Route results | Operations Center |
| --- | --- |
| ![Hapur to Noida](screenshots/02_hapur_noida_results.png) | ![OCC](screenshots/04_operations_center.png) |

## Tech Stack

| Layer | Tools |
| --- | --- |
| Frontend | React 18, TypeScript, Vite, Leaflet / react-leaflet |
| Backend | Python 3.10+, FastAPI, Uvicorn, pandas, PyArrow |
| Persistence | SQLAlchemy 2, Alembic, SQLite (default) or PostgreSQL |
| Testing | Pytest, Vitest, Testing Library, Playwright |

## Project Structure

```text
app/            FastAPI backend (routing, GTFS loader, alerts, persistence)
  api/routes.py   All /api/v1 endpoints
src/            React frontend
  components/     MapView, OperationsCenter
data/gtfs/      GTFS datasets (Delhi Metro, Indian Railways, combined multimodal)
docs/           Journey API, database, data requirements and provider guides
tests/          Backend tests (pytest)
e2e/            Playwright end-to-end tests
```

## Getting Started

### Prerequisites
- Node.js 18+ and npm
- Python 3.10+

### 1. Backend

```sh
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -e ".[test]"

export CROWDSENSE_GTFS_PATH=data/gtfs/india_multimodal.zip
uvicorn app.main:app --reload --port 8000
```

The API runs at `http://127.0.0.1:8000`, with interactive docs at `/docs`.

### 2. Frontend

```sh
cp .env.example .env
npm install
npm run dev
```

Open `http://localhost:5173`. In development, Vite proxies `/api` to the backend.

## Configuration

| Variable | Purpose |
| --- | --- |
| `CROWDSENSE_GTFS_PATH` | GTFS directory or `.zip` used for journey planning |
| `CROWDSENSE_DATABASE_URL` | Database URL (default: SQLite at `.crowdsense/crowdsense.db`) |
| `CROWDSENSE_DATA_DIR` | Local data directory (default: `.crowdsense`) |
| `CROWDSENSE_CORS_ORIGINS` | Allowed CORS origins |
| `VITE_API_URL` | Backend URL for the frontend (default: `http://127.0.0.1:8000`) |

Tables are created automatically at startup. For PostgreSQL setup, see [docs/DATABASE_SETUP.md](docs/DATABASE_SETUP.md).

## API Overview

All endpoints live under `/api/v1`.

| Method | Endpoint | Description |
| --- | --- | --- |
| POST | `/journey/plan` | Plan a journey (`origin`, `destination`, `comfort_weight`, optional `departure_time`) |
| GET | `/stops/search` | Station autocomplete |
| GET | `/network/summary` | Network statistics |
| GET | `/occ/occupancy/corridor` | Corridor occupancy for the OCC |
| POST | `/alerts/evaluate` | Run the alert engine |
| GET | `/alerts`, `/alerts/overcrowded` | List alerts |
| GET | `/recommendations` | List operator recommendations |
| POST | `/recommendations/{id}/review`, `/approve`, `/reject`, `/defer` | Recommendation workflow |
| GET | `/audit` | Audit log |
| GET | `/health/readiness` | Readiness check |

Crowd data is marked unavailable unless a separate prediction source is connected. See [docs/JOURNEY_API.md](docs/JOURNEY_API.md) and [docs/PROVIDER_INTEGRATION_GUIDE.md](docs/PROVIDER_INTEGRATION_GUIDE.md).

## Testing

```sh
pytest                # backend
npm test              # frontend unit tests (Vitest)
npm run build         # type-check and production build
npm run test:e2e      # Playwright end-to-end tests
```

## Documentation

- [Journey API](docs/JOURNEY_API.md)
- [GTFS data requirements](docs/GTFS_DATA_REQUIREMENTS.md)
- [Crowd data requirements](docs/CROWD_DATA_REQUIREMENTS.md)
- [Provider integration guide](docs/PROVIDER_INTEGRATION_GUIDE.md)
- [Database architecture](docs/DATABASE_ARCHITECTURE.md) and [setup](docs/DATABASE_SETUP.md)
- [Data quality report](docs/DATA_QUALITY_REPORT.md)
- [Project status](PROJECT_STATUS.md)

## Author

**Molly Arora** · [GitHub](https://github.com/Molly-Arora02)
