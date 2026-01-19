# SingleFile CLI (scrapefruit-cli)

Fork of [SingleFile CLI](https://github.com/gildas-lormeau/single-file-cli) - command line tool for saving complete web pages as single HTML files.

---

## Project overview

SingleFile saves web pages as self-contained HTML files with all resources (CSS, images, fonts, scripts) embedded inline. This CLI version uses Deno to inject the SingleFile script into pages via Chrome DevTools Protocol (CDP), enabling headless browser automation without a browser extension.

**Relationship to Scrapefruit:** This CLI is used as a fallback fetcher in the main [Scrapefruit](../scrapefruit/) project for capturing fully-rendered pages when simpler HTTP methods fail.

---

## Tech stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Runtime** | Deno 2.x | JavaScript/TypeScript runtime with built-in tooling |
| **Browser** | Chrome/Chromium | Headless page rendering |
| **Protocol** | Chrome DevTools Protocol | Browser automation via WebSocket |
| **Bundler** | esbuild | Bundle single-file-core into injected script |
| **Packaging** | Deno compile | Cross-platform executable generation |

---

## Directory structure

```
scrapefruit-cli/
├── single-file                 # Main entry point (shell script)
├── single-file-cli-api.js      # Core CLI API - page capture logic
├── single-file-launcher.js     # Browser process launcher
├── options.js                  # 100+ CLI options (28KB)
├── lib/
│   ├── browser.js              # Browser lifecycle management
│   ├── cdp-client.js           # Chrome DevTools Protocol client
│   ├── cdp-client-util.js      # CDP helper utilities
│   ├── constants.js            # Runtime constants
│   ├── deno-polyfill.js        # Node.js compatibility layer
│   ├── single-file-bundle.js   # Bundled SingleFile core (generated)
│   ├── single-file-script.js   # Script injection utilities
│   └── version.js              # Version info (generated)
├── build.sh                    # Bundle single-file-core into lib/
├── compile.sh                  # Compile cross-platform executables
├── dev-build.sh                # Development build script
├── Dockerfile                  # Docker image definition
├── deno.json                   # Deno configuration
├── package.json                # npm compatibility
└── resources/
    └── single-file.ico         # Windows executable icon
```

---

## Installation

### Option 1: Pre-built executable (recommended)

Download from releases and run directly:

```bash
./single-file "https://example.com" output.html
```

### Option 2: Docker

```bash
# Pull from Docker Hub
docker pull capsulecode/singlefile

# Save a page
docker run singlefile "https://wikipedia.org" > wiki.html

# With volume mount for output files
docker run -v $(pwd):/usr/src/app/out singlefile "https://example.com" --dump-content=false
```

### Option 3: From source

```bash
# Clone and enter directory
git clone https://github.com/gildas-lormeau/single-file-cli.git
cd single-file-cli

# Run directly with Deno
deno run --allow-read --allow-write --allow-net --allow-env --allow-run single-file "https://example.com"
```

**Requirements:**
- [Deno](https://deno.com/) 2.x or later
- Chrome or Chromium browser installed

---

## Usage examples

```bash
# Basic: save page to file
./single-file "https://example.com" page.html

# Dump HTML to stdout
./single-file "https://example.com" --dump-content

# Save with custom filename template
./single-file "https://example.com" --filename-template="{page-title}.html"

# Mobile emulation
./single-file "https://example.com" --browser-mobile-emulation

# Wait for dynamic content
./single-file "https://example.com" --browser-wait-delay=3000

# Crawl inner links (depth 1)
./single-file "https://example.com" --crawl-links=true --crawl-max-depth=1

# Block unnecessary resources for faster capture
./single-file "https://example.com" --block-videos --block-fonts

# Create self-extracting archive with password
./single-file "https://example.com" --self-extracting-archive --password=secret

# Use specific Chrome executable
./single-file "https://example.com" --browser-executable-path="/usr/bin/chromium"

# Specify custom viewport
./single-file "https://example.com" --browser-width=1920 --browser-height=1080
```

---

## Key options reference

Run `./single-file --help` for full list. Commonly used options:

### Browser configuration
| Option | Default | Description |
|--------|---------|-------------|
| `--browser-headless` | `true` | Run browser in headless mode |
| `--browser-executable-path` | auto-detect | Path to Chrome/Chromium |
| `--browser-width` | `1280` | Viewport width in pixels |
| `--browser-height` | `720` | Viewport height in pixels |

### Timing
| Option | Default | Description |
|--------|---------|-------------|
| `--browser-load-max-time` | `60000` | Max page load time (ms) |
| `--browser-wait-delay` | none | Wait time before capture (ms) |
| `--browser-wait-until` | `networkIdle` | Load event to wait for |

### Content processing
| Option | Default | Description |
|--------|---------|-------------|
| `--remove-hidden-elements` | `true` | Remove hidden DOM elements |
| `--remove-unused-styles` | `true` | Strip unused CSS |
| `--load-deferred-images` | `true` | Trigger lazy-loaded images |
| `--compress-HTML` | `true` | Minify HTML output |

### Output
| Option | Default | Description |
|--------|---------|-------------|
| `--dump-content` | `false` | Output HTML to stdout |
| `--compress-content` | `false` | Create ZIP instead of HTML |
| `--self-extracting-archive` | `true` | Self-extracting HTML/ZIP |
| `--output-directory` | cwd | Directory for saved files |

---

## Development

### Building the bundle

The build script fetches `single-file-core` from npm, bundles it with esbuild, and outputs to `lib/`:

```bash
./build.sh
```

This generates:
- `lib/single-file-bundle.js` - Bundled SingleFile core for injection
- `lib/version.js` - Version exported from deno.json

### Compiling executables

Cross-compile standalone executables for all platforms:

```bash
./compile.sh
```

Outputs to `dist/`:
- `single-file-aarch64-apple-darwin` (macOS Apple Silicon)
- `single-file-x86_64-apple-darwin` (macOS Intel)
- `single-file-x86_64-linux` (Linux x64)
- `single-file-aarch64-linux` (Linux ARM64)
- `single-file.exe` (Windows x64)

**Note:** macOS builds are code-signed with Apple Development certificate.

### Running tests

```bash
# Manual test: capture a page
./single-file "https://example.com" --dump-content | head -100
```

---

## Architecture notes

### How page capture works

1. **Browser launch** - `single-file-launcher.js` spawns Chrome with CDP enabled
2. **CDP connection** - `cdp-client.js` connects via WebSocket to browser
3. **Page navigation** - Navigate to URL, wait for load event
4. **Script injection** - Inject bundled SingleFile script via CDP
5. **Content capture** - SingleFile traverses DOM, inlines all resources
6. **Output** - Captured HTML returned and saved/printed

### CDP flow

```
single-file-cli-api.js
    │
    ├── single-file-launcher.js → spawns Chrome process
    │
    └── lib/cdp-client.js → WebSocket connection
            │
            └── Page.navigate(), Runtime.evaluate() → inject script
                    │
                    └── SingleFile captures → returns HTML
```

### Deno permissions required

- `--allow-read` - Read local files (for file:// URLs)
- `--allow-write` - Write output files
- `--allow-net` - CDP WebSocket, fetch page resources
- `--allow-env` - Environment variables (browser path)
- `--allow-run` - Spawn Chrome subprocess

---

## Troubleshooting

### Browser not found

```
Error: Could not find Chrome/Chromium
```

**Solution:** Install Chrome or set `--browser-executable-path`:

```bash
# Raspberry Pi / ARM Linux
./single-file "https://example.com" --browser-executable-path="/usr/bin/chromium-browser"

# Custom Chrome location
./single-file "https://example.com" --browser-executable-path="/opt/google/chrome/chrome"
```

### Timeout errors

```
Error: Page load timeout
```

**Solution:** Increase timeouts or change wait condition:

```bash
./single-file "https://example.com" \
  --browser-load-max-time=120000 \
  --browser-wait-until=domContentLoaded
```

### Memory issues on large pages

**Solution:** Block heavy resources:

```bash
./single-file "https://example.com" \
  --block-videos \
  --block-fonts \
  --max-resource-size=5
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

---

## License

SingleFile and SingleFile CLI are licensed under [AGPL-3.0](LICENSE). Code derived from third-party projects is licensed under MIT.
