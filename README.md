# 🌊 Cassini Water Monitoring Dashboard

> **11th CASSINI Hackathon — Space for Water**  
> Real-time water quality tracking, satellite telemetry, and AI-powered forecasting for inland water bodies.

---

## 📺 Demo

[![Watch the demo](https://img.shields.io/badge/YouTube-Watch%20Demo-red?logo=youtube)](https://youtu.be/jx2_00zO7Ac)

---

## Overview

The **Cassini Water Monitoring Dashboard** is a full-stack environmental web application that combines **Copernicus Sentinel-2 satellite imagery** with an **AI-driven prediction model** to monitor, analyze, and forecast water quality at user-defined geographic coordinates.

Users add custom water sources by latitude/longitude. The platform fetches live satellite indices, evaluates compliance against **EU Bathing Water Directive 2006/7/EC**, and delivers a 5-day risk forecast enriched with real-time rainfall data.

---

## ✨ Features

- **Custom Water Source Management** — Add, edit, and persistently track water locations by coordinates. Each source is a live monitoring node on the map and dashboard.
- **Live Satellite Telemetry** — Fetches and computes key indices directly from Sentinel-2 imagery via the openEO Copernicus Data Space:
  - `NDWI` — Normalized Difference Water Index (water vs. land detection)
  - `NDCI` — Normalized Difference Chlorophyll Index (algae bloom detection)
  - `Chlorophyll-a` — Estimation of chlorophyll concentration to track eutrophication
  - `Turbidity` — Water clarity measurement
  - `Suspended Sediment Load` — Physical disturbance indicator
- **AI-Powered 5-Day Forecast** — Predicts future water quality by combining current satellite indices with daily rainfall forecasts. Each day is assigned a risk score, category, and status color.
- **EU Directive Compliance Alerts** — Automatically evaluates water conditions against the EU Bathing Water Directive. If exceedance is detected or projected (`NDCI > 0.20`), the UI triggers an early-warning alert with the expected breach date.
- **14-Day Historical Trend Chart** — Interactive area chart visualizing the quality score history for any monitored location.
- **Active Disturbance Log** — Dynamically categorizes and highlights active ecological events: algae blooms, industrial/urban runoff pollution, turbidity spikes, and temperature anomalies.

---

## 🏗️ Architecture

```
User → Next.js Frontend → FastAPI Backend → openEO (Copernicus Sentinel-2)
                                         → Open-Meteo (Rainfall Forecast)
```

---

## 🗂️ Project Structure

```
cassini-water-monitor/
├── frontend/                    # Next.js web application
│   ├── app/                     # App Router pages and layouts
│   ├── components/              # UI components (map, charts, modals)
│   └── ...
│
└── backend/                     # Python FastAPI service
    ├── api.py                   # Endpoints + ThreadPoolExecutor
    ├── data_fetch.py            # openEO pipeline — fetches Sentinel-2 bands
    ├── processing.py            # NDWI/NDCI computation + pollution classification
    ├── requirements.txt
    └── water_quality_notebook.ipynb   # Copernicus JupyterHub exploration notebook
```

---

## 🖥️ Tech Stack

### Frontend
| Technology | Purpose |
|---|---|
| Next.js (App Router) + React | Core framework |
| TypeScript | Type safety |
| Tailwind CSS | Styling, dark mode, responsive layout |
| Recharts | Historical trend visualization |
| Lucide React | Icon system |

### Backend
| Technology | Purpose |
|---|---|
| FastAPI + Uvicorn | REST API server |
| Python | Core processing language |
| openEO SDK | Copernicus Sentinel-2 data pipeline |
| Open-Meteo API | Rainfall forecast data |
| ThreadPoolExecutor | Concurrent satellite job execution |

---

## 🚀 Getting Started

### Prerequisites

- Node.js ≥ 18
- Python ≥ 3.10
- A Copernicus Data Space account ([register here](https://dataspace.copernicus.eu))

---

### Backend Setup

**1. Install dependencies**
```bash
cd backend
pip install -r requirements.txt
```

**2. Authenticate with Copernicus (first time only)**
```bash
python -c "import openeo; c = openeo.connect('https://openeo.dataspace.copernicus.eu'); c.authenticate_oidc()"
```

**3. Start the API server**
```bash
uvicorn api:app --host 0.0.0.0 --port 8000 --reload
```

**4. Verify it's running**
```bash
Invoke-RestMethod -Uri “http://localhost:8000/analyze-water” -Method Post -ContentType “application/json” -Body ‘{“lat”: 41.0297, “lon”: 20.7169}’
```

---

### Frontend Setup

**1. Install dependencies**
```bash
cd frontend
npm install
```

**2. Configure environment**

Create a `.env.local` file:
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

**3. Start the development server**
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 📡 API Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/analyze-water` | Analyze water quality at `{ lat, lon }` |
| `GET` | `/health` | Health check + cache stats |
| `GET` | `/docs` | Auto-generated Swagger UI |

### Example Request
```json
POST /analyze-water
{
  "lat": 41.0297,
  "lon": 20.7169
}
```

### Example Response
```json
{
  "location": { "lat": 41.0297, "lon": 20.7169 },
  "ndwi": 0.4404,
  "ndci": 0.0027,
  "turbidity": -0.257,
  "water_detected": true,
  "pollution_status": "MEDIUM",
  "rainfall_mm": 22.3,
  "rainfall_impact": "HIGH",
  "forecast": [ ... ],
  "eu_alert": {
    "triggered": false,
    "message": "Water safe for irrigation."
  }
}
```

---

## 🟢 Pollution Classification

| NDCI Value | Status | Meaning |
|---|---|---|
| `< 0` | 🟢 LOW | Clean water or land surface |
| `0 – 0.2` | 🟡 MEDIUM | Moderate algae presence |
| `> 0.2` | 🔴 HIGH | Algae bloom / likely pollution |

---

## 🔗 Connecting Frontend to Backend

When a user selects a water location on the map:

```javascript
const response = await fetch('http://localhost:8000/analyze-water', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ lat: clickedLat, lon: clickedLon })
});

const data = await response.json();
// data.pollution_status → "LOW" | "MEDIUM" | "HIGH"
// data.water_detected   → true | false
// data.ndwi             → float
// data.ndci             → float
// data.forecast         → array of 5 daily predictions
```

---

## ⚡ Performance Notes

- Satellite data fetch takes **60–120 seconds** per unique location (openEO job execution time)
- Results are **cached for 30 minutes** to avoid redundant processing
- Cache key is rounded to ±0.01° (~1 km grid resolution)
- Up to **10 concurrent** satellite requests via `ThreadPoolExecutor`

> 💡 **Hackathon tip:** Pre-warm the cache by calling key locations on startup to ensure instant responses during demos.

---

## 🔬 JupyterHub Notebook

Upload `water_quality_notebook.ipynb` to the [Copernicus JupyterHub](https://jupyterhub.dataspace.copernicus.eu) to:

- Explore any coordinate interactively
- Visualize NDWI and NDCI satellite maps
- Run batch analysis across multiple water bodies
- Call the FastAPI backend directly from the notebook

## Team

- Ognen Mladenovski - FINKI
- Hristina Gjorgjievska - FINKI
- Marko Mijoski - FINKI
- Marija Misajlovska - FINKI
- Tamara Stojanoska - FINKI
- Sara Andonovska - FINKI
- Ana Marija Utevska - AFS
