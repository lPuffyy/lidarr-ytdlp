# 🎵 Lidarr-YTDLP
**Automatically download missing tracks from Lidarr using `yt-dlp` + `ffmpeg`.**  
It monitors your Lidarr wanted albums, finds matching YouTube audio, downloads and tags tracks, then triggers a rescan so Lidarr can import them automatically.

---

## ✨ Features
- 🔁 Monitors Lidarr’s *wanted* albums and missing tracks  
- 🎧 Downloads songs from YouTube using [`yt-dlp`](https://github.com/yt-dlp/yt-dlp)  
- 🏷️ Tags MP3s with correct artist, album, title, and track number  
- 🧠 Fuzzy matching for better search accuracy  
- 🎛️ Beautiful dark web dashboard (port **5004**)  
- ⚡ Automatic Lidarr rescan after successful downloads  

---

## 🧩 Docker Compose Example

```yaml
version: "3.8"

services:
  lidarr-ytdlp:
    image: lpuffyy/lidarr-ytdlp:latest
    container_name: lidarr-ytdlp
    restart: unless-stopped
    ports:
      - "5004:5004" # Dashboard port
    environment:
      - TZ=Etc/UTC
      - LIDARR_URL=http://192.168.5.47:8686/api/v1
      - LIDARR_API_KEY=changeme
      - MUSIC_DIR=/music
      - CHECK_INTERVAL=3600  # how often to check (in seconds)
    volumes:
      - /path/to/music:/music
      - /path/to/config:/config
    command: ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "5004"]
```

---

## 🐳 Manual Dockerfile (if building yourself)

```dockerfile
# ==========================
# Lidarr-YTDLP Dockerfile
# ==========================
FROM python:3.12-slim

RUN apt-get update &&     apt-get install -y --no-install-recommends ffmpeg curl &&     pip install --no-cache-dir yt-dlp fastapi uvicorn requests mutagen &&     apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY app.py /app/app.py

ENV TZ=Etc/UTC     LIDARR_URL=http://localhost:8686/api/v1     LIDARR_API_KEY=changeme     MUSIC_DIR=/music     CHECK_INTERVAL=3600

RUN mkdir -p /music
EXPOSE 5004

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "5004"]
```

Build it yourself:
```bash
docker build -t lpuffyy/lidarr-ytdlp .
```

Run it manually:
```bash
docker run -d   --name lidarr-ytdlp   -p 5004:5004   -e LIDARR_URL=http://192.168.5.47:8686/api/v1   -e LIDARR_API_KEY=changeme   -v /path/to/music:/music   lpuffyy/lidarr-ytdlp
```

---

## ⚙️ Environment Variables

| Variable | Description | Default |
|-----------|--------------|----------|
| `LIDARR_URL` | URL to Lidarr API (e.g., `http://192.168.5.47:8686/api/v1`) | `http://localhost:8686/api/v1` |
| `LIDARR_API_KEY` | Your Lidarr API key (Settings → General) | *(required)* |
| `MUSIC_DIR` | Directory where MP3s are saved | `/music` |
| `CHECK_INTERVAL` | Time in seconds between checks for wanted albums | `3600` |
| `TZ` | Timezone for container logs | `Etc/UTC` |

---

## 🖥️ Dashboard
Visit: [http://localhost:5004](http://localhost:5004)

Displays real-time data including:
- Current download action  
- Wanted track count  
- Downloaded track count  
- Next scheduled check  
- Recent downloads with timestamps  

The dashboard updates automatically every few seconds.  

---

## 🧠 How It Works
1. Pulls your **wanted albums** from Lidarr’s API.  
2. Expands them into individual missing tracks.  
3. Searches YouTube for best-quality matches via `yt-dlp`.  
4. Downloads audio and converts it to MP3 using `ffmpeg`.  
5. Tags each file (`artist`, `album`, `title`, `tracknumber`).  
6. Triggers a Lidarr rescan to import the new tracks automatically.  

---

## 📂 Folder Structure
```
/music/
├── Artist/
│   ├── Album/
│   │   ├── 01 - Track.mp3
│   │   ├── 02 - Track.mp3
│   │   └── ...
```

---

## 🧾 Logs
Real-time logs are available via Docker:
```bash
docker logs -f lidarr-ytdlp
```

Look for lines like:
```
[LIDARR-YTDLP] ✅ Downloaded: /music/Artist/Album/Track.mp3
[LIDARR-YTDLP] 🔄 Triggered rescan for /music/Artist
```

---

## 🛠️ Troubleshooting

**❓ Tracks not importing in Lidarr**
- Make sure Lidarr’s *root folder* matches the container’s `/music`.
- Verify tags are correct:
  ```bash
  ffprobe -v error -show_entries format_tags -of json "/music/Artist/Album/Track.mp3"
  ```
- Ensure API key and URL are valid.
- Lidarr may take up to ~10 seconds to detect imports after a rescan.

**❓ Dashboard shows 0 downloads**
- The app updates every few seconds.
- Check if the container restarted — session counts reset on restart.

---

## 🧡 Credits
- [Lidarr](https://lidarr.audio/) — music collection manager  
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) — YouTube downloader  
- [FastAPI](https://fastapi.tiangolo.com/) — async web framework  
- [Mutagen](https://mutagen.readthedocs.io/) — metadata tagging library  

---

## 📜 License
**MIT License** © 2025 [lpuffyy](https://github.com/lpuffyy)

You are free to use, modify, and distribute this project under the MIT terms.
