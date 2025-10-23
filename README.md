# 🎵 Lidarr-YTDLP
**Automatically download missing tracks from Lidarr using `yt-dlp` + `ffmpeg`.**  
It monitors your Lidarr wanted albums, finds matching YouTube audio, downloads and tags tracks, then triggers a rescan so Lidarr can import them automatically.

---

## 🚀 Features
- 🔁 Monitors Lidarr’s *wanted* albums and missing tracks  
- 🎧 Downloads songs from YouTube using [`yt-dlp`](https://github.com/yt-dlp/yt-dlp)  
- 🏷️ Tags MP3s with correct artist, album, title, and track number  
- 🧠 Fuzzy-match search for best YouTube results  
- 🎛️ Web dashboard (dark mode) on port **5004** for live status and logs  
- ⚡ Auto-rescan of Lidarr library after each successful download  

---

## 🧩 Example Stack (Docker Compose)
```yaml
version: "3.8"

services:
  lidarr-ytdlp:
    image: lpuffyy/lidarr-ytdlp:latest
    container_name: lidarr-ytdlp
    restart: unless-stopped
    ports:
      - "5004:5004"          # Dashboard port
    environment:
      - TZ=Etc/UTC
      - LIDARR_URL=http://localhost:8686/api/v1
      - LIDARR_API_KEY=changeme
      - MUSIC_DIR=/music
      - CHECK_INTERVAL=3600  # how often to check for wanted tracks (seconds)
    volumes:
      - /path/to/music:/music
      - /path/to/config:/config   # optional, for logs/config
    command: ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "5004"]
🐳 Dockerfile
If you’d like to build it yourself:

dockerfile
Copy code
FROM python:3.12-slim
RUN apt-get update && \
    apt-get install -y --no-install-recommends ffmpeg curl && \
    pip install --no-cache-dir yt-dlp fastapi uvicorn requests mutagen && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY app.py /app/app.py
ENV TZ=Etc/UTC \
    LIDARR_URL=http://localhost:8686/api/v1 \
    LIDARR_API_KEY=changeme \
    MUSIC_DIR=/music \
    CHECK_INTERVAL=3600
RUN mkdir -p /music
EXPOSE 5004
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "5004"]
Build:

bash
Copy code
docker build -t lpuffyy/lidarr-ytdlp .
Run:

bash
Copy code
docker run -d \
  --name lidarr-ytdlp \
  -p 5004:5004 \
  -v /path/to/music:/music \
  -e LIDARR_URL=http://localhost:8686/api/v1 \
  -e LIDARR_API_KEY=changeme \
  lpuffyy/lidarr-ytdlp
⚙️ Environment Variables
Variable	Description	Default
LIDARR_URL	URL to Lidarr API (http://host:8686/api/v1)	http://localhost:8686/api/v1
LIDARR_API_KEY	Your Lidarr API key (Settings → General)	(required)
MUSIC_DIR	Path where music will be saved	/music
CHECK_INTERVAL	Seconds between wanted album checks	3600
TZ	Timezone (for logs)	Etc/UTC

🖥️ Dashboard
Visit: http://localhost:5004
Displays:

Current action (e.g., “Downloading Artist – Track”)

Wanted track count

Download count

Next scan timer

Recent downloads table

🧠 How It Works
Fetches your wanted albums from Lidarr’s API.

Expands them into individual tracks.

Searches YouTube for the best audio match via yt-dlp.

Downloads and converts to MP3 with ffmpeg.

Tags each file (artist, album, title, tracknumber).

Triggers a Lidarr rescan for immediate import.

📂 Folder Structure
bash
Copy code
/music/
├── Artist/
│   ├── Album/
│   │   ├── 01 - Track.mp3
│   │   ├── 02 - Track.mp3
│   │   └── ...
🧾 Logs
Logs are printed to container output.
View in real-time:

bash
Copy code
docker logs -f lidarr-ytdlp
🛠️ Troubleshooting
Tracks not importing in Lidarr?

Ensure Lidarr’s root folder path matches /music inside both containers.

Verify MP3 tags with:

bash
Copy code
ffprobe -v error -show_entries format_tags -of json "/music/Artist/Album/Track.mp3"
Confirm your API key and URL are correct.

Lidarr may take a few seconds after rescan to import new tracks.

🧡 Credits
Lidarr

yt-dlp

FastAPI

Mutagen

📜 License
MIT © 2025 lpuffyy
