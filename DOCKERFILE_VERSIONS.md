# Dockerfile Versions

This repository provides five Dockerfile versions with multi-browser support:

## 1. Dockerfile (Unified - Chrome or Brave)

**Recommended for: Production, PJeOffice compatibility, privacy-focused automation**

**NEW**: The unified Dockerfile now supports multiple browsers via build argument:
- **Chrome (default)**: Production-ready Google Chrome from Chrome for Testing
- **Brave**: Privacy-focused Brave browser with built-in ad-blocking

### Build with Chrome (default):
```bash
docker build -t rpa-worker-selenium .
# or explicitly
docker build --build-arg BROWSER_TYPE=chrome -t rpa-worker-selenium .
```

### Build with Brave:
```bash
docker build --build-arg BROWSER_TYPE=brave -t rpa-worker-selenium-brave .
```

**Features:**
- Multi-stage build for optimization
- Downloads specific Chrome version or installs Brave from official repository
- Downloads matched ChromeDriver from Chrome for Testing
- Optimized for PJeOffice digital signature compatibility (Chrome)
- Enhanced privacy features and built-in ad-blocking (Brave)
- Single Dockerfile for both browsers - reduces maintenance overhead

**Build Requirements:**
- Chrome: Requires internet access to `dl.google.com`, `storage.googleapis.com`, `googlechromelabs.github.io`
- Brave: Requires internet access to `brave-browser-apt-release.s3.brave.com`, `storage.googleapis.com`, `googlechromelabs.github.io`

## 2. Dockerfile.firefox (Firefox Browser)

**Recommended for: Firefox-specific testing, Gecko engine compatibility, Mozilla standards**

- Uses multi-stage build for optimization
- Downloads specific Firefox version (145.0)
- Downloads matched GeckoDriver (v0.36.0) from GitHub releases
- Mozilla's Gecko rendering engine
- Full support for Firefox WebDriver capabilities
- **Requires internet access to ftp.mozilla.org and github.com during build**

**Build command:**
```bash
docker build -f Dockerfile.firefox -t rpa-worker-selenium-firefox .
```

**Example usage:**
```bash
docker run --rm rpa-worker-selenium-firefox example_script_firefox.py
```

**Note:** If building behind a corporate firewall or in a restricted network environment, ensure that the following domains are accessible during build:
- `ftp.mozilla.org` (for Firefox download)
- `github.com` (for GeckoDriver download)

## 3. dockerfile.slim (Optimized Debian Trixie - Conditional Features) ⭐ NEW

**Recommended for: Production deployments, cost-conscious environments, modular feature selection**

**OPTIMIZED**: Significantly smaller than Dockerfile.trixie while preserving all critical functionality!

- Uses `debian:trixie-slim` as base (vs full `debian:trixie`)
- **Default build is minimal (~2-2.5 GB vs ~4 GB for Dockerfile.trixie)**
- Multi-stage build with Chrome for Testing + Firefox ESR
- **Build-time flags for optional features:**
  - `ENABLE_VNC=0` (default) - Install x11vnc for remote debugging
  - `ENABLE_FFMPEG=0` (default) - Install FFmpeg for screen recording
  - `ENABLE_NOVNC=0` (default) - Install noVNC for browser-based VNC
  - `ENABLE_PDF_TOOLS=0` (default) - Install ImageMagick + Ghostscript
  - `BUILD_PJEOFFICE=0` (default) - Install PJeOffice for Brazilian legal
- **Removed audio stack** (pulseaudio, libasound2) - focus on headless/GUI-over-X11
- **Minimal font set** (fonts-liberation, fonts-dejavu-core only)
- **Build tools cleaned** (build-essential, python3-dev purged after pip install)
- Full certificate handling (A1 .pfx/.p12) with NSS tools
- Screenshot support via Selenium (no extra packages needed)
- Health check to verify browser availability
- **Requires internet access to `storage.googleapis.com`, `download.mozilla.org`, `github.com` during build**

**Build command (minimal - recommended):**
```bash
docker build -f dockerfile.slim -t rpa-worker-selenium-slim .
```

**Build command (with all debug features):**
```bash
docker build -f dockerfile.slim \
  --build-arg ENABLE_VNC=1 \
  --build-arg ENABLE_NOVNC=1 \
  --build-arg ENABLE_FFMPEG=1 \
  --build-arg ENABLE_PDF_TOOLS=1 \
  -t rpa-worker-selenium-slim-debug .
```

**Build with PJeOffice:**
```bash
docker build -f dockerfile.slim \
  --build-arg BUILD_PJEOFFICE=1 \
  -t rpa-worker-selenium-slim-pje .
```

**Example usage (minimal):**
```bash
# Basic headless automation
docker run --rm rpa-worker-selenium-slim python /app/example_script.py

# With virtual display for screenshots
docker run --rm -e USE_XVFB=1 rpa-worker-selenium-slim python /app/script.py
```

**Example usage (with debug features):**
```bash
# With VNC debugging
docker run --rm \
  -e USE_XVFB=1 \
  -e USE_VNC=1 \
  -p 5900:5900 \
  rpa-worker-selenium-slim-debug python /app/script.py

# With noVNC (browser-based VNC)
docker run --rm \
  -e USE_XVFB=1 \
  -e USE_VNC=1 \
  -e USE_NOVNC=1 \
  -p 6080:6080 \
  rpa-worker-selenium-slim-debug python /app/script.py
# Access via: http://localhost:6080/vnc.html
```

**Key advantages:**
- ✅ **44% smaller** than Dockerfile.trixie (default build)
- ✅ **Pay only for what you use** - enable features as needed
- ✅ Same functionality as Dockerfile.trixie when all features enabled
- ✅ Perfect for production (minimal) and development (with debug args)
- ✅ Faster builds (fewer packages to install)
- ✅ Lower storage and bandwidth costs
- ✅ Compatible with all existing automation scripts

**When to use:**
- ✅ Production deployments where size matters
- ✅ CI/CD pipelines (build minimal for prod, full for dev)
- ✅ Cost-conscious cloud environments
- ✅ When you want fine-grained control over features
- ✅ When you only need VNC/recording occasionally

**See also:**
- [DOCKERFILE_SLIM.md](DOCKERFILE_SLIM.md) - Detailed documentation
- [DOCKERFILE_SLIM_EXAMPLES.md](DOCKERFILE_SLIM_EXAMPLES.md) - Build examples and use cases

## 4. Dockerfile.trixie (Debian Trixie Desktop - Enhanced GUI/Window Management + Multi-Browser)

**Recommended for: PJeOffice certificate dialogs, complex window interactions, GUI automation, robust graphical worker with noVNC**

**ENHANCED**: Now uses Debian Trixie (13) - more complete and updated than Bookworm (12)!

- Uses Debian Trixie (13) as base image - latest testing version with up-to-date packages
- **NEW**: Includes both Chrome and Firefox ESR browsers
- **NEW**: Includes both ChromeDriver and GeckoDriver
- Comprehensive desktop environment and GUI libraries
- Enhanced window management tools (wmctrl, xdotool, xautomation)
- Full GTK2/GTK3 support for certificate and authentication dialogs
- D-Bus and PolicyKit support for system dialogs
- AT-SPI accessibility for complex GUI interactions
- Audio support (PulseAudio) for multimedia dialogs
- noVNC support for browser-based remote access
- Robust for graphical environments with comprehensive package support
- Larger image size but maximum compatibility
- **Java 21 runtime for applets and PJeOffice/other Java-based signers**
- Health check to verify browser availability
- **Requires internet access to `storage.googleapis.com`, `ftp.mozilla.org`, `github.com` during build**

**Build command:**
```bash
docker build -f Dockerfile.trixie -t rpa-worker-selenium-debian .
```

**Build with PJeOffice support:**
```bash
docker build -f Dockerfile.trixie --build-arg BUILD_PJEOFFICE=1 -t rpa-worker-selenium-debian-pje .
```

**Example usage with Chrome:**
```bash
# Basic usage
docker run --rm rpa-worker-selenium-debian example_script.py

# With GUI services enabled for PJeOffice
docker run --rm \
  -e USE_XVFB=1 \
  -e USE_OPENBOX=1 \
  -e USE_PJEOFFICE=1 \
  rpa-worker-selenium-debian-pje my_pjeoffice_script.py
```

**Example usage with Firefox:**
```bash
# Basic Firefox usage
docker run --rm rpa-worker-selenium-debian example_script_firefox.py

# With GUI services
docker run --rm \
  -e USE_XVFB=1 \
  -e USE_OPENBOX=1 \
  rpa-worker-selenium-debian example_script_firefox.py
```

**Note:** This image is specifically designed to handle:
- PJeOffice certificate password dialogs and other Java-based digital signers
- Complex window interactions requiring full desktop environment
- GTK-based authentication prompts
- Robust graphical worker environments with noVNC support
- Applications requiring complete Debian Trixie environment compatibility
- Multi-browser testing (Chrome + Firefox) in same environment

- **Use `Dockerfile.trixie`** if:
  - You need to handle PJeOffice certificate password dialogs
  - You need support for Java-based digital signers and applets
  - You're experiencing window management issues with other images
  - You need maximum Debian Trixie environment compatibility
  - You want the latest testing packages (more updated than Bookworm)
  - You need full desktop environment support for complex GUI interactions
  - You're working with GTK-based authentication dialogs
  - You need noVNC for browser-based remote access
  - You need both Chrome and Firefox in the same image
  - Image size is less important than compatibility and feature completeness

## 5. Dockerfile.alpine (Lightweight Serverless - Chromium & Firefox)

**Recommended for: Serverless environments (AWS Lambda, Google Cloud Run), minimal footprint, cost optimization**

- Lightweight Debian-slim based image (named "alpine" for consistency)
- Minimal dependencies for reduced image size
- Includes Chromium browser and ChromeDriver from Debian repos
- Includes Firefox ESR browser
- NO PJeOffice, VNC, or GUI features (headless only)
- Optimized for serverless and containerized environments
- Perfect for Lambda functions, Cloud Run, Fargate, etc.
- Smaller image size than full-featured versions

**Build command:**
```bash
docker build -f Dockerfile.alpine -t rpa-worker-selenium-alpine .
```

**Example usage:**
```bash
# Basic headless automation
docker run --rm rpa-worker-selenium-alpine python /app/alpine_smoke_test.py

# Custom script
docker run --rm -v $(pwd)/data:/data rpa-worker-selenium-alpine python your_script.py
```

**Key Features:**
- **Browsers**: Chromium 142.x, Firefox 145.0
- **Drivers**: ChromeDriver (included), GeckoDriver (v0.36.0)
- **Size**: Significantly smaller than full images
- **Use Cases**: Lambda functions, Cloud Run, ECS Fargate, Kubernetes jobs
- **Limitations**: No GUI tools, no VNC, no PJeOffice support

**Testing:**
```bash
# Run the Alpine smoke test
docker run --rm -v $(pwd)/data:/data rpa-worker-selenium-alpine python /app/alpine_smoke_test.py
```

- **Use `Dockerfile.alpine`** if:
  - You're deploying to serverless environments (Lambda, Cloud Run, Fargate)
  - You need minimal image size for cost optimization
  - You only need headless browser automation
  - You don't need GUI features, VNC, or PJeOffice
  - You're running in containerized environments with resource constraints
  - You want faster container startup times

## Which Should You Use?

### Quick Decision Guide

| Dockerfile | Best For | Image Size | Browser(s) | GUI Support | Build Time |
|------------|----------|------------|------------|-------------|------------|
| `Dockerfile` (Chrome) | Production, PJeOffice | Medium | Chrome (Latest) | Yes | Medium |
| `Dockerfile` (Brave) | Privacy, Ad-blocking | Medium | Brave | Yes | Medium |
| `Dockerfile.firefox` | Firefox Testing | Medium | Firefox | Yes | Medium |
| **`dockerfile.slim`** ⭐ | **Production, Cost-saving** | **Small-Large** | **Chrome + Firefox ESR** | **Yes** | **Fast-Medium** |
| `Dockerfile.trixie` | PJeOffice, Full Features | Large | Chrome + Firefox ESR | Full (Debian Trixie) | Slow |
| `Dockerfile.alpine` | Serverless, Lambda | **Smallest** | Chromium & Firefox | No | Fast |

**Note:** `dockerfile.slim` size varies: ~2-2.5 GB (minimal) to ~3.5-4 GB (all features enabled)

### Detailed Decision Guide

- **Use `dockerfile.slim`** ⭐ **NEW** if:
  - You want a smaller image for production (~2-2.5 GB vs ~4 GB)
  - You want to control which features are installed (VNC, FFmpeg, noVNC, PDF tools)
  - You need Chrome + Firefox ESR in the same image
  - You want the flexibility to build minimal (prod) or full (dev) images
  - You're cost-conscious about storage and bandwidth
  - You want faster builds and deployments
  - **This is the recommended choice for most use cases**

- **Use `Dockerfile` (Chrome)** if:
  - You're deploying to production
  - You need PJeOffice digital signature compatibility
  - You need the official Google Chrome browser
  - You need maximum compatibility
  - You have internet access during builds
  - This is the default and most common choice

- **Use `Dockerfile` (Brave)** if:
  - You need privacy-focused browsing automation
  - You want built-in ad-blocking capabilities
  - You're testing websites with Brave browser specifically
  - You prefer Brave's Chromium-based features
  - Same features as Chrome version but with Brave browser

- **Use `Dockerfile.firefox`** if:
  - You need to test with Firefox/Gecko engine specifically
  - You need Mozilla-specific WebDriver features
  - You're validating cross-browser compatibility
  - You prefer Firefox's rendering and standards compliance
  - You need Firefox-specific features or extensions

- **Use `Dockerfile.trixie`** if:
  - You need to handle PJeOffice certificate password dialogs
  - You need support for Java-based digital signers (PJeOffice and others)
  - You're experiencing window management issues with other images
  - You need Debian Trixie (13) - more updated packages than Bookworm (12)
  - You need robust graphical worker environment with noVNC support
  - You need full desktop environment support for complex GUI interactions
  - You're working with GTK-based authentication dialogs
  - **You need both Chrome AND Firefox ESR in the same image**
  - Image size is less important than compatibility and feature completeness

- **Use `Dockerfile.alpine`** (⭐ Recommended for Serverless):
  - You're deploying to AWS Lambda, Google Cloud Run, Azure Container Instances
  - You need minimal image size for cost optimization (storage, bandwidth, cold starts)
  - You only need headless browser automation
  - You don't need GUI features, VNC, or PJeOffice
  - You're running in containerized environments with resource constraints
  - You want faster container startup times

## Build Cache Optimizations

All Dockerfiles now include build cache optimizations:

- **APT Cache Mounts**: `--mount=type=cache,target=/var/cache/apt` speeds up rebuilds by caching package downloads
- **Pip Cache Mounts**: `--mount=type=cache,target=/root/.cache/pip` speeds up Python package installations
- **Layer Separation**: Downloads and installations are separated into different layers for better caching
- **Multi-stage Builds**: Used where appropriate to keep final images small

To build with BuildKit (required for cache mounts):
```bash
DOCKER_BUILDKIT=1 docker build -f Dockerfile.chrome -t rpa-worker-selenium .
```

All versions include all the same Python packages and provide the same functionality for Selenium automation.
