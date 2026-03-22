# Geo Intelligence Suite

A self-contained, web-based geo-locator tool with a dark C2-style dashboard.  
Runs locally at `http://localhost:5000`.

---

## Features

| Feature | Description |
|---|---|
| **Image GPS Extraction** | Upload a photo or paste an image URL — extracts GPS coordinates from EXIF metadata |
| **AI Scene Analysis** | Uses a local Ollama vision model to identify location clues in images with no GPS data |
| **IP Geolocation** | Look up any IPv4 or IPv6 address — returns city, ISP, timezone, and map marker |
| **Address Search** | Convert any place name or address to coordinates via OpenStreetMap geocoding |
| **Reverse Geocoding** | Ctrl+Click anywhere on the map to reverse-geocode that coordinate |
| **Interactive Map** | Leaflet.js + OpenStreetMap — dark-themed with color-coded markers and popups |
| **Activity Log** | Real-time log of all lookups — click any entry to re-focus the map |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.8+ + Flask |
| AI Vision | Ollama (local) — moondream, llava, llava-phi3, minicpm-v |
| Map | Leaflet.js + OpenStreetMap tiles |
| Geocoding | Nominatim (OpenStreetMap) — free, no API key |
| IP Geolocation | ip-api.com — free, no API key (45 req/min) |
| EXIF Extraction | Pillow (Python Imaging Library) |
| Frontend | Vanilla HTML/CSS/JavaScript |

---

## Project Structure

```
Geo_Intelligence_Suite/
├── start.bat               # One-click launcher (Windows) — sets up everything automatically
├── app.py                  # Flask application — routes and entry point
├── requirements.txt        # Python dependencies
├── README.md               # This file
├── uploads/                # Temporary folder for uploaded images (auto-cleared)
├── static/
│   └── style.css           # Dark C2-style dashboard CSS
├── templates/
│   └── index.html          # Frontend with map, forms, and activity log
└── utils/
    ├── __init__.py
    ├── exif_extractor.py   # GPS extraction from EXIF metadata
    ├── ip_lookup.py        # IP geolocation via ip-api.com
    ├── geocode.py          # Nominatim geocoding + reverse geocoding
    └── ollama_vision.py    # Ollama AI vision model integration
```

---

## Quick Start (Windows — Recommended)

### Double-click `start.bat`

The launcher does everything automatically:

1. Checks Python is installed
2. Creates a Python virtual environment (`.venv/`)
3. Installs all dependencies from `requirements.txt`
4. Checks if Ollama is installed
5. Starts the Ollama server in the background
6. Checks if a vision model is installed — **downloads `moondream` automatically** if none found
7. Opens the app in your browser at `http://localhost:5000`

> On first run, downloading `moondream` takes a few minutes (~1.8 GB). Subsequent runs are instant.

---

## Manual Setup

### 1. Install Python

Download Python 3.8+ from [python.org](https://www.python.org/downloads/).  
During installation, check **"Add Python to PATH"**.

### 2. Create a virtual environment

```bat
cd Geo_Intelligence_Suite
python -m venv .venv
.venv\Scripts\activate
```

On Linux/macOS:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 4. Install Ollama (for AI Scene Analysis)

Download from [https://ollama.com/download](https://ollama.com/download) and install it.

Start the Ollama server:
```bash
ollama serve
```

Install a vision model (choose one):

| Model | Size | Speed | Command |
|---|---|---|---|
| `moondream` | ~1.8 GB | Fastest | `ollama pull moondream` |
| `llava` | ~4.7 GB | Fast | `ollama pull llava` |
| `llava:7b` | ~4.7 GB | Fast | `ollama pull llava:7b` |
| `llava-phi3` | ~2.9 GB | Fast | `ollama pull llava-phi3` |
| `minicpm-v` | ~5.5 GB | Medium | `ollama pull minicpm-v` |

> **Recommended:** `moondream` for speed, `llava-phi3` for accuracy.

### 5. Run the app

```bash
python app.py
```

### 6. Open in browser

```
http://localhost:5000
```

---

## Ollama AI Scene Analysis

When no EXIF GPS data is present in an image, the AI analysis feature uses a local Ollama vision model to examine the photo and identify geographic clues such as:

- Visible street signs or business names
- Recognizable landmarks or monuments
- Architectural style and building materials
- Vegetation type and terrain
- License plate regional formats
- Flags, national symbols
- Language visible on any text
- Climate and weather indicators

The model runs entirely on your local machine — no data is sent to any external server.

### Ollama Server Details

| Setting | Value |
|---|---|
| Default URL | `http://localhost:11434` |
| API used | `POST /api/chat` |
| Streaming | Disabled (waits for full response) |
| Timeout | 90 seconds |

### Checking Ollama status

```bash
# Check server is running
curl http://localhost:11434

# List installed models
ollama list

# Pull a new model
ollama pull moondream
```

---

## API Endpoints

All endpoints accept and return JSON (except `/upload` which takes multipart form data).

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Serve the main dashboard |
| `GET` | `/ollama/models` | List installed Ollama vision models |
| `POST` | `/upload` | Upload image file — EXIF GPS + optional Ollama analysis |
| `POST` | `/upload-url` | Fetch image from URL — EXIF GPS + optional Ollama analysis |
| `POST` | `/ip` | Body: `{ "ip": "8.8.8.8" }` — returns location data |
| `POST` | `/search` | Body: `{ "address": "Eiffel Tower" }` — returns coordinates |
| `POST` | `/reverse` | Body: `{ "lat": 48.858, "lon": 2.294 }` — returns address |

### Upload with Ollama (multipart form)

```
POST /upload
  image       = <file>
  use_ollama  = true
  ollama_model = moondream
```

### Upload from URL with Ollama (JSON)

```json
POST /upload-url
{
  "url": "https://example.com/photo.jpg",
  "use_ollama": true,
  "ollama_model": "moondream"
}
```

---

## Usage Guide

### Extract GPS from Image

1. Open the **Image GPS Extract** panel.
2. Choose **File Upload** or **Image URL** tab.
3. Select a file or paste a direct image URL.
4. Click **Extract GPS** / **Fetch & Extract**.
5. If the photo has GPS EXIF data, a green marker appears on the map.

> Many images shared online have EXIF stripped. Use original photos from a camera or phone.

### AI Scene Analysis (Ollama)

1. Toggle **AI Scene Analysis** in the Image GPS Extract panel.
2. Select a vision model from the dropdown (installed models are auto-detected).
3. Process any image — the AI analysis appears below the GPS result.
4. Useful for images with no GPS metadata.

### IP Geolocation

1. Open the **IP Geolocation** panel.
2. Type any IPv4 or IPv6 address (e.g., `8.8.8.8`).
3. Click **Lookup IP** or press Enter.

### Address Search

1. Open the **Address Search** panel.
2. Type any place, address, or landmark.
3. Click **Search Location** or press Enter.

### Reverse Geocode (Map)

- Hold **Ctrl** and click anywhere on the map to drop a reverse-geocoded marker.

### Map Controls

| Button | Action |
|---|---|
| Clear All Markers | Remove every marker from the map |
| Fit All Markers | Zoom/pan to show all placed markers |
| World View | Reset map to the default world view |

---

## External APIs

| API | Used For | Limits | Key Required |
|---|---|---|---|
| [ip-api.com](http://ip-api.com) | IP geolocation | 45 requests/minute | No |
| [Nominatim (OSM)](https://nominatim.org) | Address geocoding + reverse | 1 request/second | No |
| [OpenStreetMap Tiles](https://www.openstreetmap.org) | Map rendering | Fair use | No |
| [Ollama](https://ollama.com) | Local AI vision | Unlimited (local) | No |

---

## Security Notes

- Uploaded images are deleted immediately after processing.
- No image data is stored on disk beyond momentary processing.
- All external API calls are made server-side — no keys exposed to the browser.
- Ollama runs entirely locally — images are never sent to external AI services.
- The app binds to `localhost` only by default.

---

## Troubleshooting

**`start.bat` says Python not found**
- Reinstall Python from [python.org](https://www.python.org/downloads/) and check "Add Python to PATH".

**Ollama not found after installing**
- Close and reopen your terminal/command prompt so the PATH update takes effect.
- Or restart your PC.

**Ollama AI toggle shows "Cannot reach Ollama"**
- Run `ollama serve` in a separate terminal window and keep it open.
- `start.bat` does this automatically.

**AI analysis times out**
- The model may still be loading into memory on first use. Try again after 10–15 seconds.
- Use `moondream` for the fastest response times.

**Image shows no GPS data**
- The image may not have GPS EXIF tags.
- Enable **AI Scene Analysis** to use the Ollama vision model instead.
- Some apps (WhatsApp, Telegram, social media) strip EXIF before sharing.

**IP lookup returns inaccurate location**
- ip-api.com provides city-level accuracy. VPNs show the VPN server's location.
- IPv6 lookups may return broader regional results.

**Address not found**
- Use a more specific query: `"Eiffel Tower, Paris, France"` instead of `"Eiffel Tower"`.
- Nominatim enforces a 1 req/sec rate limit — wait a moment between rapid searches.

---

## Future Enhancements

- Batch IP lookup from a CSV file
- Export markers to KML / GeoJSON
- Satellite / hybrid map tile layer toggle
- Persistent lookup history using SQLite
- Automatic EXIF scrubber tool

---

## License

For personal / educational use. Respect the terms of service of all external APIs used.
