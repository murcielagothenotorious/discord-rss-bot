# discord-rss-bot

[![linting: pylint](https://img.shields.io/badge/linting-pylint-yellowgreen)](https://github.com/pylint-dev/pylint)
[![pylint score](./.github/badges/pylint.svg)](./.github/badges/pylint.svg)
[![code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Poetry](https://img.shields.io/endpoint?url=https://python-poetry.org/badge/v0.json)](https://python-poetry.org/)
![GitHub top language](https://img.shields.io/github/languages/top/murcielagothenotorious/discord-rss-bot)
![GitHub Release](https://img.shields.io/github/v/release/murcielagothenotorious/discord-rss-bot)

A Discord bot that delivers RSS feed updates in real-time to your servers.

👉 Check out the [Docker image](https://github.com/murcielagothenotorious/discord-rss-bot/pkgs/container/discord-rss-bot) for easy deployment.

Powered by [Feed Reader](https://github.com/lemon24/reader). Inspired by [FeedCord](https://github.com/Qolors/FeedCord).

![Preview](./.github/images/preview.jpg)

## Features

- 🔄 Automated RSS Feed Updates – Periodic updates with configurable intervals.
- 📜 Enhanced Message Formatting – Attempts to convert HTML to Markdown, truncates long summaries without breaking formatting.
- 🖼️ Image & Media Support – Uses the first image as an embed cover for rich Discord embeds.
- 🔍 Title Filters – Per-feed regex filters via slash commands; only matching entries get posted.
- ⚡ Efficient & Scalable – Optimized with async processing and concurrent execution.
- 🐋 Dockerized for Easy Deployment – Run anywhere with minimal setup (`linux/amd64` & `linux/arm64` supported).

## Configuration

1. Create a Discord server. See also [How do I create a server?](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server)
2. Create a Discord bot account and get its token. See also [Creating a bot account](https://discordpy.readthedocs.io/en/stable/discord.html)
3. Add the bot to your Discord server & channels.

The bot is configured via a YAML file. Here is an example:

```yaml
db_path: data/rss.sqlite3

feeds:
  - feed_url: https://www.daemonology.net/hn-daily/index.rss
    channel_id: 123456789
    update_interval: 30

  - feed_url: https://hnrss.org/best
    channel_id: 987654321
    update_interval: 30 # optional, defaults to 60 minutes if not provided
```

## Title Filters

You can set a per-feed regex filter directly from Discord using slash commands. Only entries whose title matches the pattern will be posted.

| Command | Description |
|---|---|
| `/filter set feed_url:<url> pattern:<regex>` | Set a filter for a feed |
| `/filter clear feed_url:<url>` | Remove the filter for a feed |
| `/filter list` | Show all current filters |

Patterns are **case-insensitive**. Examples:

```
/filter set feed_url:https://... pattern:python|django
/filter set feed_url:https://... pattern:\[.*8.*\]
```

Filters are stored in the SQLite database — they persist across restarts and do not require a config file change. Entries that don't match are still marked as read so they won't be reprocessed if the filter changes later.

## Docker run

```bash
docker run -d \
  --name discord-rss-bot \
  --restart unless-stopped \
  -e DISCORD_BOT_TOKEN=your_token_here \
  -v $(pwd)/config.yaml:/app/config.yaml \
  -v $(pwd)/data:/app/data \
  ghcr.io/murcielagothenotorious/discord-rss-bot:latest
```

## Docker compose

```yaml
services:
  discord-rss-bot:
    image: ghcr.io/murcielagothenotorious/discord-rss-bot:latest
    container_name: discord-rss-bot
    environment:
      - DISCORD_BOT_TOKEN=${DISCORD_BOT_TOKEN}
    volumes:
      - ./config.yaml:/app/config.yaml
      - ./data:/app/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:8080/healthz"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
```

```bash
docker compose up -d
```

To update to the latest image:

```bash
docker compose pull && docker compose up -d
```

## Local development

```bash
git clone https://github.com/murcielagothenotorious/discord-rss-bot.git && cd discord-rss-bot
poetry install --with dev

poetry run python -m discord_rss_bot <args>
poetry run pylint --verbose discord_rss_bot
poetry run black --verbose discord_rss_bot
```
