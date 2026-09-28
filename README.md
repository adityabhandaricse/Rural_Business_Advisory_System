# Rural Business Advisory Platform

Prototype for the SIH brief: *AI-Driven Hyper-Local Business Advisory & Financial
Structuring Assistant for Rural Micro-Entrepreneurs*.

## What's in this package

| Part | Status | Where |
|---|---|---|
| **Live, runnable demo** | ✅ Works right now, no install | `live-demo/index.html` |
| React frontend (Vite) | 📦 Scaffolded, real component code | `frontend/` |
| Django + DRF backend | 📦 Scaffolded, real endpoint code | `backend/` |

### 1. `live-demo/index.html` — open this first
A single self-contained HTML file that actually runs the full pipeline in your
browser, no server needed:
- **Deterministic financial engine** (Section 3 of the spec) in plain JavaScript —
  scheme routing, quarterly amortization, EMI table and charts.
- **Live competitor map** — geocodes your place name via **OpenStreetMap
  Nominatim**, then queries the **Overpass API** for real shops/businesses within
  5–10 km, drawn on a Leaflet map with a saturation index.
- **Multilingual UI** — switch the language dropdown (Hindi, Bengali, Marathi,
  Tamil, Telugu). Static labels are pre-translated; the dynamically generated
  advisory text and DPR summary are translated on the fly via the free,
  keyless **MyMemory Translation API**.
- **Rule-based advisory & SWOT** — stands in for the Gemini call in the demo
  (no API key available client-side). It consumes the same computed
  financial + geospatial data the real Gemini prompt would use, so swapping
  it for a live model call is a drop-in change (see below).
- **DPR export** — downloads a plain-text bank-ready summary.

Just open the file in a browser — everything else is client-side fetches to
public, keyless APIs (Nominatim, Overpass, MyMemory).

### 2. `backend/` — Django REST Framework
Implements Section 4 (models) and Section 5 (API) of the spec, file-for-file:
- `models.py` — `User`, `BusinessCategory`, `LocalBusinessPOI`, `FeasibilityReport`
- `calculators.py` — the same deterministic math as the live demo, in Python/Decimal
- `geo_services.py` — server-side Overpass client + Haversine fallback (no PostGIS required)
- `ai_services.py` — Gemini structured-JSON advisory call (Section 6); **requires `GEMINI_API_KEY`**
- `serializers.py`, `views.py`, `urls.py` — the 5 endpoints from Section 5
- `seed_categories.py` — populates `BusinessCategory` with the OSM tag sets the frontend/live-demo use
- `settings_snippet.py`, `requirements.txt`

To run for real: create a Django project, drop these files into an app named
`core`, apply the settings snippet, set `DATABASE_URL` (Postgres — PostGIS is
optional, the code auto-falls-back to Haversine) and `GEMINI_API_KEY`, then:
```bash
pip install -r requirements.txt --break-system-packages
python manage.py migrate
python manage.py shell < seed_categories.py
python manage.py runserver
```

### 3. `frontend/` — React 18 + Vite + Tailwind
Real component implementations matching Section 7:
- `FinancialCalculatorWidget.jsx` — slider + Recharts pie/bar charts, calls `/finance/structure-loan/`
- `InteractiveMap.jsx` — react-leaflet with 5km/10km circles + competitor markers
- `SwotMatrix.jsx`, `AdvisoryReportView.jsx` — renders the Gemini-shaped response
- `DprPdfGenerator.jsx` — client-side PDF export via html2pdf.js
- `context/LanguageContext.jsx` — language switch + Web Speech API (voice in/out)
- `api/client.js` — axios client with auth + error interceptors
- `App.jsx` — wires the wizard end-to-end

To run for real (needs the backend above running on `localhost:8000`):
```bash
cd frontend
npm install
npm run dev
```

## Why one file runs live and the rest is scaffolded
This chat environment can generate and hand you complete, correct code for
every layer, but it can't host a persistent Postgres/Django server or hold a
Gemini API key for you to call live. The `live-demo` file sidesteps that by
doing the deterministic math and the real Overpass/Nominatim/MyMemory calls
directly in the browser — so you can see the actual pipeline (not a mock)
working end to end today, while `backend/` and `frontend/` are the
production-shaped code to deploy when you're ready.

## Swapping in real Gemini output
The live demo's `generateAdvisory()` function and the backend's
`ai_services.generate_ai_feasibility_study()` both consume the identical
inputs (financial figures + geo/saturation data) and return the identical
shape (`market_reach_summary`, `swot_*`, `pricing_strategy`,
`bank_dpr_summary`). Once `backend/` is deployed with a `GEMINI_API_KEY`,
point the frontend's `generateFeasibility()` call at it and the rule-based
demo text is replaced by real Gemini structured output with no schema
changes needed anywhere else in the stack.
#
