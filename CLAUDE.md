# SingleFile CLI (scrapefruit-cli)

Fork of SingleFile CLI - command line tool for saving complete web pages as single HTML files.

## Overview

SingleFile saves web pages as self-contained HTML files with embedded CSS, images, and scripts. This CLI version runs through Deno with Chrome DevTools Protocol.

## Tech stack

- **Runtime**: Deno / Node.js
- **Browser**: Chrome/Chromium (headless)
- **Protocol**: Chrome DevTools Protocol

## Quick start

Download executable from releases, or run via Docker:

```bash
# Docker
docker pull capsulecode/singlefile
docker run singlefile "https://example.com"

# Direct usage
./single-file "https://example.com"
```

## Key files

```
scrapefruit-cli/
├── single-file-cli-api.js   # Main CLI API
├── single-file-launcher.js  # Browser launcher
├── options.js               # Configuration options
├── lib/                     # SingleFile core library
├── build.sh                 # Build script
└── Dockerfile               # Docker build
```

---

## Multi-machine workflow

This repo is developed across multiple machines. GitHub is the source of truth.

**Before switching machines:**
```bash
git add . && git commit -m "WIP" && git push
```

**After switching machines:**
```bash
git pull
```
