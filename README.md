# YouTube Music Python API and Flask audio extraction backend with yt-dlp streaming API

This project is a Flask audio extraction backend and YouTube Music Python API that returns song metadata and direct audio streams. The service receives search queries, extracts track information from YouTube Music, and delivers audio stream endpoints through a yt-dlp streaming API.

Clients query the REST endpoints to search songs by title and extract direct audio streaming URLs without running yt-dlp on the client device. The backend resolves media streams in webm or m4a formats and returns responses in JSON format.

## Architecture

A Flask application in app.py coordinates two modules.

1. Track search module:
   YoutubeMusicAPI.py queries https://music.youtube.com/search using httpx with a desktop user agent. A regular expression extracts video IDs from the HTML response. The module queries the YouTube oEmbed API for each video ID to collect track title, artist name, and thumbnail image. The route limits search results to 5 tracks.

2. Stream extraction module:
   ytdlp.py uses the yt-dlp library to inspect formats without downloading the media file. The extractor filters stream formats for audio streams with webm or m4a containers, selects the stream format at the end of the format list, and returns the direct playback URL. Authentication cookies from cookies.txt provide session credentials to prevent request blocking.

3. Server entry points:
   app.py runs the development server on port 1234. start_gunicorn.py detects the local IPv4 address, displays a QR code in ASCII format for mobile clients, and starts Gunicorn on port 8000. For cloud environments, railpack.json specifies Python 3.11 on Debian slim and executes Gunicorn bound to the PORT environment variable.

## Setup

Clone the repository and install the dependencies in a virtual environment.

```bash
git clone https://github.com/devsanthoshmk/m14u-python-backend.git
cd m14u-python-backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Place an exported YouTube cookies.txt file in the root directory. yt-dlp requires this file to authenticate requests against YouTube endpoints.

## Usage

Start the Flask development server on port 1234:

```bash
python app.py
```

Or start the Gunicorn server on port 8000 with local IP detection:

```bash
python start_gunicorn.py
```

### Search tracks

Endpoint: GET /song_results

Query parameter:
- input: Search query string.

Example request:

```bash
curl -X GET "http://localhost:1234/song_results?input=never+gonna+give+you+up"
```

Example response:

```json
{
  "0": {
    "artist": "Rick Astley",
    "cover": "https://i.ytimg.com/vi/dQw4w9WgXcQ/hqdefault.jpg",
    "name": "Never Gonna Give You Up",
    "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
  }
}
```

### Extract audio link

Endpoint: GET /playable_link

Query parameter:
- song_url: Full YouTube watch URL.

Example request:

```bash
curl -X GET "http://localhost:1234/playable_link?song_url=https://www.youtube.com/watch?v=dQw4w9WgXcQ"
```

Example response:

```json
"https://googlevideo.com/videoplayback?id=example"
```

## Configuration

The service uses the following files and configuration options:

- cookies.txt: Netscape format cookie file for yt-dlp authentication.
- railpack.json: Deployment configuration for Railway. The build uses Python 3.11 on Debian slim and sets the default port to 5000.
- requirements.txt: Package dependencies including Flask 3.1.2, flask-cors 6.0.1, yt-dlp 2025.9.5, and httpx 0.28.1.
