# 🌲 ForestGuard

**Real-time wildfire risk mapping and early-warning alerts for the areas you care about.**

---

## Overview

ForestGuard is a map-based monitoring platform that visualizes wildfire risk across regions in near real time — think of it as a weather/traffic map, but for fire danger. It ingests satellite imagery, weather data (temperature, humidity, wind, drought indices), and historical fire records to compute a **Fire Risk Score** for every grid cell on the map, then color-codes zones from low to extreme risk.

Users can subscribe to specific regions (a town, a property, a hiking trail, a whole county) and receive push, SMS, or email alerts the moment risk crosses a threshold or an active fire is detected nearby.

**Core capabilities:**
- 🗺️ Interactive risk map with live-updating heat layers
- 🔥 Active fire detection via satellite hotspot feeds (VIIRS/MODIS-style data)
- 🌡️ Weather-driven risk modeling (temperature, humidity, wind speed, fuel dryness)
- 🔔 Configurable alerts (push, SMS, email, webhook)
- 📊 Historical trend analysis and risk forecasting (24h / 72h outlook)
- 🧭 Custom "Watch Zones" — draw or search an area to monitor

> ⚠️ **Note:** ForestGuard is a fictional/demo project. It is not a substitute for official emergency services or evacuation guidance. Always follow instructions from local authorities.

---

## Quick Start

### Prerequisites

- Node.js `>= 18.x` and npm `>= 9.x`
- Docker & Docker Compose (for local database + cache services)
- A free API key from a weather data provider (e.g. OpenWeatherMap) — set as `WEATHER_API_KEY`
- PostgreSQL 14+ with PostGIS extension (spun up automatically via Docker Compose)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/forestguard/forestguard.git
cd forestguard

# 2. Install dependencies
npm install

# 3. Copy the example environment file and fill in your keys
cp .env.example .env

# 4. Start supporting services (Postgres + Redis)
docker compose up -d

# 5. Run database migrations and seed sample regions
npm run db:migrate
npm run db:seed

# 6. Start the app in development mode
npm run dev
```

### Expected Output

```
[forestguard] loading risk models... done
[forestguard] connected to postgres (postgis enabled)
[forestguard] connected to redis
[forestguard] weather sync: 1,204 stations loaded
[forestguard] server listening on http://localhost:4000
[forestguard] map client available at http://localhost:3000
```

Open **http://localhost:3000** in your browser — you should see the risk map load with sample Watch Zones already plotted (e.g. "Sample County, CA").

---

## Usage

### Viewing the risk map

Navigate to the web app and use the search bar to jump to any location. Zones are color-coded:

| Color | Risk Level |
|-------|------------|
| 🟢 Green | Low |
| 🟡 Yellow | Moderate |
| 🟠 Orange | High |
| 🔴 Red | Very High |
| 🟣 Purple | Extreme |

### Creating a Watch Zone

1. Click **"New Watch Zone"** on the map.
2. Draw a polygon or search an address/radius.
3. Name the zone and set alert thresholds (e.g. notify at "High" risk or above).
4. Choose alert channels (push, SMS, email, webhook).

### CLI usage

```bash
# Check current risk score for a coordinate
forestguard risk check --lat 34.05 --lng -118.24

# List all active fire hotspots within 50km of a point
forestguard fires nearby --lat 34.05 --lng -118.24 --radius 50

# Subscribe an email to a saved Watch Zone
forestguard alerts subscribe --zone-id wz_1a2b3c --email you@example.com
```

### API usage

```bash
curl -X GET "https://api.forestguard.io/v1/risk?lat=34.05&lng=-118.24" \
  -H "Authorization: Bearer $FORESTGUARD_API_KEY"
```

Example response:

```json
{
  "location": { "lat": 34.05, "lng": -118.24 },
  "riskLevel": "high",
  "riskScore": 78,
  "factors": {
    "temperature": 34.2,
    "humidity": 12,
    "windSpeedKph": 28,
    "fuelDryness": 0.81
  },
  "activeFiresNearby": 1,
  "forecast24h": "very_high"
}
```

---

## Configuration

All configuration is managed via environment variables (see `.env.example`).

| Variable | Description | Default |
|----------|--------------|---------|
| `PORT` | Port for the API server | `4000` |
| `DATABASE_URL` | PostGIS-enabled Postgres connection string | `postgres://forestguard:forestguard@localhost:5432/forestguard` |
| `REDIS_URL` | Redis connection string, used for caching risk computations | `redis://localhost:6379` |
| `WEATHER_API_KEY` | API key for weather/climate data provider | — |
| `SATELLITE_FEED_URL` | URL of the satellite hotspot data feed | provider-specific |
| `ALERT_SMS_PROVIDER` | SMS gateway to use for alerts (`twilio`, `none`) | `none` |
| `ALERT_EMAIL_FROM` | From-address for email alerts | `alerts@forestguard.local` |
| `RISK_UPDATE_INTERVAL_MIN` | How often risk scores are recomputed, in minutes | `15` |
| `RISK_THRESHOLD_DEFAULT` | Default risk level that triggers alerts | `high` |

Configuration files:
- `config/regions.yml` — predefined regions and default zoom levels
- `config/risk-model.yml` — weights used in the risk scoring algorithm (fuel type, slope, drought index, wind, etc.)

---

## Development

```bash
# Run the test suite
npm test

# Run linter and formatter
npm run lint
npm run format

# Run in watch mode with hot reload
npm run dev

# Build for production
npm run build

# Run production build locally
npm start
```

### Project structure

```
forestguard/
├── apps/
│   ├── api/          # REST API + risk scoring engine
│   └── web/           # Map client (React + Mapbox/Leaflet)
├── packages/
│   ├── risk-model/     # Fire risk scoring logic
│   ├── alerts/         # Alert dispatch (push/SMS/email/webhook)
│   └── ingestion/       # Weather + satellite data pipelines
├── config/
├── docker-compose.yml
└── .env.example
```

### Running tests for a single package

```bash
npm run test --workspace=packages/risk-model
```

---

## Contributing

Contributions are welcome!

1. Fork the repository and create a feature branch:
   `git checkout -b feature/my-improvement`
2. Make your changes, following the existing code style (`npm run lint` before committing).
3. Add or update tests covering your change.
4. Commit using [Conventional Commits](https://www.conventionalcommits.org/) (e.g. `feat: add drought index to risk model`).
5. Open a pull request describing the change and its motivation.

Please read `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md` before submitting significant changes. For larger features (e.g. new data providers, alert channels), open an issue to discuss the approach first.

---

## Support

- 📖 Documentation: [docs.forestguard.io](https://docs.forestguard.io)
- 🐛 Bug reports & feature requests: [GitHub Issues](https://github.com/forestguard/forestguard/issues)
- 💬 Community chat: [ForestGuard Discord](https://discord.gg/forestguard)
- 📧 Direct support: support@forestguard.io

---

## License

This project is licensed under the [MIT License](LICENSE).

---

*ForestGuard is a demo/fictional project created for illustrative purposes and is not an official emergency alert system.*
