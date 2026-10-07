# Dipriva Video Content Creator 🎬

AI-generated short-form video pipeline for Dipriva marketing content. Give it a topic or idea, and it produces a finished short video — script, stock footage, narration, subtitles, and background music — ready to post to LinkedIn.

Forked from [harry0703/MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo) ([MIT License](LICENSE)). This fork is trimmed down to the setup Dipriva actually uses: local Mac + Docker, Claude (Anthropic) for scripts, Pixabay for stock footage, and Edge TTS for narration.

---

## What it does

1. You provide a topic or idea (e.g. "3 signs your business is ready to scale past founder-led sales")
2. Claude generates a script from it
3. The app pulls matching stock footage from Pixabay
4. Edge TTS narrates the script
5. Subtitles are auto-generated and burned in
6. Background music is layered in
7. FFmpeg renders a finished MP4 in your chosen aspect ratio (9:16 recommended for LinkedIn)

Output lands in a local folder that syncs to OneDrive automatically (see **Output & Storage** below) — no manual upload step.

## What you need

| Requirement | Where to get it | Cost |
|---|---|---|
| Anthropic API key | [console.anthropic.com](https://console.anthropic.com) → API Keys | Pay-per-use, separate from any Claude.ai subscription |
| Pixabay API key | [pixabay.com/api/docs](https://pixabay.com/api/docs/) → sign up, key issued immediately on your account page | Free |
| Docker Desktop | [docker.com](https://www.docker.com/products/docker-desktop/) | Free |
| OneDrive (Microsoft 365) | Already installed/signed in on your Mac | Included in your plan |

TTS (Edge TTS) needs no key — it's free and built in.

## Setup

### 1. Clone this repo

```shell
git clone https://github.com/mdipresreyes-lab/Dipriva-Video-Content-Creator.git
cd Dipriva-Video-Content-Creator
```

### 2. Create your config

```shell
cp config.example.toml config.toml
```

Open `config.toml` and add:
- Your Anthropic API key (LLM section)
- Your Pixabay API key (materials section)

> **Note:** Pexels has paused new API key issuance as of late 2026. Pixabay's free key covers video search fully — the gated fields on Pixabay's API (high-res images, vector files) require separate approval, but don't apply to video sourcing, which is all this pipeline uses. No approval wait needed.

Leave the app-level `api_key` field blank — this stays local-only on your Mac, so no extra auth layer is needed. (If you ever expose the WebUI/API beyond `localhost`, set this field first.)

### 3. Point output at OneDrive

Create a folder inside your OneDrive sync directory:

```shell
mkdir -p ~/Library/CloudStorage/OneDrive-Dipriva/MarketingVideos
```

(Check the exact OneDrive folder name with `ls ~/Library/CloudStorage/` — it varies by tenant name.)

In `docker-compose.release.yml`, mount that folder as a volume so the container writes directly into it:

```yaml
volumes:
  - ~/Library/CloudStorage/OneDrive-Dipriva/MarketingVideos:/app/storage/tasks
```

Anything the app saves there syncs to OneDrive automatically — no manual export or upload step.

### 4. Run it

```shell
docker compose -f docker-compose.release.yml up
```

Open the WebUI at [http://127.0.0.1:8501](http://127.0.0.1:8501).

### 5. Command line (optional)

For quick one-off generation without the browser:

```shell
uv run python cli.py --video-subject "your topic here"
```

Run `uv run python cli.py --help` for all options, including `--batch-file` for queuing multiple videos at once (up to 100 per batch).

## Before posting anything real

- Run generated scripts through Dipriva's brand voice/tone check before publishing
- Verify background music licensing — bundled tracks in `resource/songs` are sourced from YouTube per the upstream project; confirm they're clear for commercial LinkedIn use, or swap in your own
- Pixabay's terms require crediting them when displaying search results to end users (relevant if you ever show footage-picker results to someone else) and prohibit automated mass/bulk downloading — fine for normal one-at-a-time video generation, just don't script unattended batch runs against their API
- Post to LinkedIn manually at first; automating that step is a separate project with its own API approval process

## Troubleshooting

**`RuntimeError: No ffmpeg exe could be found`**
FFmpeg usually auto-downloads. If it fails, download it manually from [gyan.dev](https://www.gyan.dev/ffmpeg/builds/) and set the path in `config.toml`:
```toml
[app]
ffmpeg_path = "/path/to/your/ffmpeg"
```

**`OSError: [Errno 24] Too many open files`**
```shell
ulimit -n 10240
```

## Reference

- Full upstream documentation (all providers, Windows installer, Colab, etc.): [harry0703/MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo)
- License: [MIT](LICENSE)
