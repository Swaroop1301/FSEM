# FSEM — Fuzzy Smart Energy Monitor Dashboard

A premium, **single-file HTML dashboard** that connects to the FSEM n8n automation backend to evaluate room energy conditions using fuzzy logic and display historical readings.

---

## Live Demo

Open `index.html` directly in any browser — no build step, no server required.

Or visit the n8n-hosted version:
```
https://swaroop1301.app.n8n.cloud/webhook/fsem/dashboard
```

---

## What It Does

| Feature | Description |
|---|---|
| **Evaluate** | Submit room sensor data (temperature, occupancy, power, hour) and instantly get a fuzzy logic score, category, recommendation, and which rules fired |
| **Score Gauge** | Animated SVG ring that fills 0–100 and color-shifts: 🟢 Optimal · 🟡 Moderate · 🔴 High |
| **Fuzzy Memberships** | Displays the degree of membership for each input variable returned by the engine |
| **Top Rules** | Lists the fuzzy rules that were triggered during evaluation |
| **History Table** | Live feed of the last 50 readings — newest first, with colored category badges |
| **Auto-refresh** | History reloads automatically after every successful evaluation |
| **Live Clock** | Real-time clock in the header |

---

## n8n Workflow Architecture

```
Browser (index.html)
    │
    ├──POST /webhook/fsem/evaluate──► FSEM 01 Evaluate Reading API
    │                                      │
    │                                      └──► FSEM 00 Core Fuzzy Engine
    │                                                (not HTTP-exposed)
    │
    └──GET  /webhook/fsem/history ──► FSEM 02 History API
```

| Workflow | Role | Endpoint |
|---|---|---|
| **FSEM 00** | Core fuzzy logic engine | Internal only |
| **FSEM 01** | Accepts POST, runs engine, saves + returns result | `POST /webhook/fsem/evaluate` |
| **FSEM 02** | Returns saved readings, newest first | `GET /webhook/fsem/history?limit=50` |
| **FSEM 03** | Serves the HTML dashboard page | `GET /webhook/fsem/dashboard` |

---

## API Reference

### POST `/webhook/fsem/evaluate`

**Allowed fields only** (any extra field returns 400):

```json
{
  "temperature_c":   26,
  "occupancy_count": 15,
  "room_capacity":   40,
  "current_power_w": 1500,
  "rated_power_w":   2000,
  "hour":            14,
  "source":          "ui",
  "scenario":        "manual"
}
```

**Success response** (`200`):
```json
{
  "ok": true,
  "saved": true,
  "score": 72,
  "category": "Optimal",
  "recommendation": "Energy usage is optimal for current conditions.",
  "explanation": "Low occupancy ratio with moderate power draw during active hours.",
  "memberships": {
    "temperature": { "Comfortable": 0.67 },
    "occupancy":   { "Low": 0.17, "Medium": 0.63 },
    "power":       { "Normal": 0.75 },
    "hour":        { "Active": 1.0 }
  },
  "top_rules": ["IF temp=Comfortable AND occupancy=Medium THEN score=High"]
}
```

**Validation error** (`400`): `{ "errors": [...] }`  
**Engine/storage error** (`500`): `{ "message": "..." }`

### GET `/webhook/fsem/history?limit=50`

Returns saved readings newest first:
```json
{
  "records": [
    {
      "timestamp":     "2026-10-20T18:14:00Z",
      "score":         72,
      "category":      "Optimal",
      "recommendation": "...",
      "temperature_c": 26,
      "occupancy_count": 15,
      "room_capacity": 40,
      "current_power_w": 1500,
      "hour":          14,
      "scenario":      "manual"
    }
  ]
}
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **UI** | Vanilla HTML5 + CSS3 + ES2020 JavaScript (zero dependencies) |
| **Fonts** | [Outfit](https://fonts.google.com/specimen/Outfit) (UI) + [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) (data) |
| **Backend** | [n8n](https://n8n.io) automation workflows |
| **Logic** | Fuzzy logic engine (FSEM 00) — called internally by FSEM 01 |
| **Storage** | n8n built-in storage (accessed via FSEM 02) |

---

## Running Locally

Just open `index.html` in Chrome, Firefox, or Edge. The page calls the n8n cloud instance directly. CORS is already enabled (`Access-Control-Allow-Origin: *`) on all endpoints.

To point at a different n8n instance, change the `BASE` constant at the top of the `<script>` tag in `index.html`:

```js
const BASE = "https://your-instance.app.n8n.cloud/webhook/fsem";
```

---

## Design

- **Theme:** Dark-tech — deep navy `#080c13` background with emerald `#00d97e` accent
- **CSS grid background** for a terminal/monitoring aesthetic
- **Glassmorphism** card borders with hover glow
- **Animated SVG gauge** ring for the fuzzy score
- **Skeleton loaders** that match the table layout while data loads
- Fully **responsive** down to 480px

---

## License

MIT
