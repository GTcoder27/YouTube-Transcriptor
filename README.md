# YouTube Transcriptor 🎬📝

A lightweight Flask REST API that retrieves available captions/transcripts for YouTube videos and returns the cleaned transcript as JSON. You can request a transcript using a YouTube URL or video ID, select a preferred language, check which transcript languages are available, and monitor service health.

The API is suitable for use as a backend component in applications such as video summarizers, search tools, note-taking apps, and language-learning workflows.

## ✨ Features

- Fetch a YouTube video's transcript using its URL or video ID.
- Request a preferred language, such as Hindi (`hi`) or English (`en`).
- Clean transcript text by joining segments and normalizing whitespace.
- Return transcript text, language metadata, character length, and word count.
- List the transcript languages available for a video.
- Health-check endpoint for deployment monitoring.
- Optional Webshare proxy configuration through environment variables.
- JSON error responses for invalid URLs, disabled captions, and unavailable transcripts.
- Flask development server and Gunicorn-compatible deployment setup.

> **Important:** This project retrieves captions that YouTube makes available through `youtube-transcript-api`. It does not download or transcribe the video's audio itself. A video needs accessible captions/subtitles for transcript retrieval to work.

## 🧰 Tech Stack

- **Python**
- **Flask** — REST API
- **youtube-transcript-api** — YouTube transcript retrieval
- **Flask-CORS** — Cross-origin requests
- **Gunicorn** — WSGI server for deployment
- **Webshare proxy support** — Optional proxy configuration

## 📁 Project Structure

```text
YouTube-Transcriptor/
├── app.py              # Flask API and transcript endpoints
├── requirements.txt    # Python dependencies
├── Procfile            # Platform process command
├── render.yaml         # Render deployment configuration
├── LICENSE
└── README.md
```

## ✅ Prerequisites

- Python 3.9 or newer (Python 3.10+ recommended)
- pip
- Internet access to retrieve YouTube captions
- Optional: a Webshare proxy account if you choose to use a proxy

Check your installation:

```bash
python --version
pip --version
```

## 🚀 Run Locally

### 1. Clone the repository

Replace the URL with your own GitHub repository URL:

```bash
git clone https://github.com/<your-username>/YouTube-Transcriptor.git
cd YouTube-Transcriptor
```

### 2. Create and activate a virtual environment

**Linux / macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows PowerShell:**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables (optional)

The app supports these environment variables:

| Variable | Required | Description |
|---|---|---|
| `PORT` | No | Server port; defaults to `5000` |
| `FLASK_DEBUG` | No | Set to `true` to enable Flask debug mode locally |
| `WEBSHARE_PROXY_USERNAME` | No | Webshare proxy username |
| `WEBSHARE_PROXY_PASSWORD` | No | Webshare proxy password |

If you use a proxy, set the credentials in your shell rather than hardcoding them in `app.py`.

**Linux / macOS:**

```bash
export WEBSHARE_PROXY_USERNAME="your_proxy_username"
export WEBSHARE_PROXY_PASSWORD="your_proxy_password"
```

**Windows PowerShell:**

```powershell
$env:WEBSHARE_PROXY_USERNAME="your_proxy_username"
$env:WEBSHARE_PROXY_PASSWORD="your_proxy_password"
```

If you do not use a proxy, leave these variables unset. Verify the current implementation's proxy fallback behavior before deploying; proxy providers may require account-specific settings.

### 5. Start the API

```bash
python app.py
```

By default, the server listens on:

```text
http://localhost:5000
```

Check that the service is running by opening [http://localhost:5000/health](http://localhost:5000/health).

For production, use a WSGI server such as Gunicorn instead of Flask's built-in development server:

```bash
gunicorn app:app --bind 0.0.0.0:5000
```

## 📖 How to Use the API

### 1. Check the service

```bash
curl http://localhost:5000/
```

The root endpoint returns basic API information, the available routes, and usage examples.

### 2. Check health

```bash
curl http://localhost:5000/health
```

Example response:

```json
{
  "status": "healthy",
  "service": "YouTube Transcript API",
  "port": "5000",
  "proxy_configured": false,
  "version": "1.2"
}
```

### 3. Get a transcript using POST

Send a JSON body with a YouTube URL and an optional list of preferred languages:

```bash
curl -X POST "http://localhost:5000/transcript" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://www.youtube.com/watch?v=VIDEO_ID",
    "languages": ["hi", "en"]
  }'
```

Replace `VIDEO_ID` with a real video ID that has accessible captions.

The API tries the requested language preferences using the transcript library. If a transcript is retrieved, the response includes fields such as:

```json
{
  "success": true,
  "video_id": "VIDEO_ID",
  "url": "https://www.youtube.com/watch?v=VIDEO_ID",
  "language_used": "en",
  "transcript": "Example transcript text...",
  "length": 28,
  "word_count": 4,
  "proxy_used": false
}
```

The exact `language_used` value depends on the captions returned by the library and the video.

### 4. Get a transcript using GET

You can also pass the URL as a query parameter:

```text
http://localhost:5000/transcript?url=https://youtu.be/VIDEO_ID&lang=hi
```

Or with `curl`:

```bash
curl --get "http://localhost:5000/transcript" \
  --data-urlencode "url=https://youtu.be/VIDEO_ID" \
  --data-urlencode "lang=hi"
```

The `lang` parameter is optional. Without it, the API lets the transcript library choose from available captions.

### 5. Check available transcript languages

```bash
curl "http://localhost:5000/languages/VIDEO_ID"
```

The response lists available caption languages and whether each transcript is auto-generated. Use a returned `language_code` as a preference in the transcript request.

### Supported YouTube URL formats

The current URL parser supports:

```text
https://www.youtube.com/watch?v=VIDEO_ID
https://youtu.be/VIDEO_ID
https://www.youtube.com/embed/VIDEO_ID
VIDEO_ID
```

A standard YouTube video ID is typically 11 characters. Replace placeholder IDs with the actual ID from the video URL.

## 🔌 API Reference

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/` | API information and usage examples |
| `GET` | `/health` | Health-check status |
| `POST` | `/transcript` | Retrieve transcript using a JSON body |
| `GET` | `/transcript?url=...&lang=...` | Retrieve transcript using query parameters |
| `GET` | `/languages/<video_id>` | List available transcript languages |

### POST `/transcript`

Request body:

```json
{
  "url": "https://youtu.be/VIDEO_ID",
  "languages": ["hi", "en"]
}
```

- `url` (**required**): a supported YouTube URL or video ID.
- `languages` (optional): an array of preferred language codes.

Successful responses include `success`, `video_id`, `url`, `language_used`, `transcript`, `length`, `word_count`, and `proxy_used`.

Errors include a descriptive `error` and may include `detailed_error`, `possible_reasons`, and a suggestion to inspect available languages.

## 🧩 Use It From Another Application

Any frontend or backend that can make HTTP requests can call this API.

Example Python client:

```python
import requests

response = requests.post(
    "http://localhost:5000/transcript",
    json={
        "url": "https://youtu.be/VIDEO_ID",
        "languages": ["en", "hi"],
    },
    timeout=60,
)

response.raise_for_status()
data = response.json()

print(data["transcript"])
```

If you call the API from a browser-based frontend hosted on another origin, configure CORS for the specific trusted origin before deploying publicly.

## ☁️ Deployment

The repository includes a `Procfile` and `render.yaml` as deployment configuration examples. To deploy on a Python-compatible hosting provider:

1. Push the project to a GitHub repository.
2. Create a web service connected to that repository.
3. Install dependencies with `pip install -r requirements.txt`.
4. Configure the start command to run Gunicorn, for example:

   ```bash
   gunicorn app:app --bind 0.0.0.0:$PORT
   ```

5. Add proxy credentials as environment variables only if you use a proxy.
6. Deploy and check `/health` on the public service URL.

The host generally supplies the port through the `PORT` environment variable. Confirm the selected platform's current build and start-command requirements. The included deployment files may need adjustment for your provider.

## 🛠️ Troubleshooting

- **Invalid YouTube URL:** Use one of the supported formats shown above or provide a valid video ID.
- **Transcript unavailable:** The video may have no captions, captions may be disabled, the video may be private, or the requested language may not exist.
- **Language not found:** Call `/languages/<video_id>` and request a language code listed in the response.
- **Request blocked or rate-limited:** YouTube may restrict automated requests from your IP. Review the library's current guidance and use a proxy only if you have a legitimate, properly configured provider.
- **Proxy errors:** Check the proxy username/password and account status. Try without proxy credentials to determine whether the proxy is the cause.
- **Import or method errors:** Check the installed `youtube-transcript-api` version against the methods used in `app.py`. The library API can change between releases; pin a tested version in `requirements.txt` when you have verified compatibility.
- **CORS error in browser:** Configure `CORS` for the frontend origin rather than allowing every origin in production.
- **Slow responses:** Transcript retrieval depends on YouTube and network availability. Set suitable client timeouts and handle failures gracefully.

## 🔐 Security and Responsible Use

- **Do not commit proxy credentials.** The uploaded source has fallback proxy credential values in `app.py`. Remove those defaults and rotate/revoke the credentials if they are real before publishing the repository.
- Keep secrets in environment variables or your deployment platform's secret manager.
- Restrict CORS to trusted origins for production.
- Add request rate limits and sensible request timeouts before exposing the API publicly.
- Avoid logging sensitive configuration values.
- Respect YouTube's terms, caption ownership, copyright, and applicable privacy rules. Only retrieve and reuse transcripts when you have the appropriate rights and permission.
- Use production hosting with a WSGI server such as Gunicorn; do not enable Flask debug mode in production.

## 🤝 Contributing

Bug reports, improvements, and pull requests are welcome. For major changes, open an issue first to discuss what you would like to change.

## 📄 License

This repository includes a `LICENSE` file. Review it for the applicable license terms before redistributing or reusing the project.
