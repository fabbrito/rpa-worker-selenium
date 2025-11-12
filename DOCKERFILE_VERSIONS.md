# Dockerfile Versions

This repository provides five Dockerfile versions:

## 1. Dockerfile (Default - Chromium from Debian repos)

**Recommended for: Testing, restricted networks, simple use cases**

- Uses Chromium from Debian repositories
- No external downloads required during build
- Easier to build in restricted environments
- Slightly older Chrome version
- Fully functional for most automation tasks

**Build command:**
```bash
docker build -t rpa-worker-selenium .
```

## 2. Dockerfile.chrome (Production - Google Chrome with matched ChromeDriver)

**Recommended for: Production use, latest Chrome features, maximum compatibility**

- Uses multi-stage build for optimization
- Downloads specific Chrome version (142.0.7444.59)
- Downloads matched ChromeDriver from Chrome for Testing
- Latest Chrome features and updates
- Requires internet access to dl.google.com and storage.googleapis.com during build

**Build command:**
```bash
docker build -f Dockerfile.chrome -t rpa-worker-selenium .
```

## 3. Dockerfile.brave (Brave Browser)

**Recommended for: Privacy-focused automation, ad-blocking, Brave-specific features**

- Uses Brave browser from official Brave repository
- Built on Chromium, fully compatible with Selenium/ChromeDriver
- Enhanced privacy features and built-in ad-blocking
- Latest stable Brave browser version
- **Requires internet access to brave-browser-apt-release.s3.brave.com and storage.googleapis.com during build**

**Build command:**
```bash
docker build -f Dockerfile.brave -t rpa-worker-selenium-brave .
```

**Example usage:**
```bash
docker run --rm rpa-worker-selenium-brave example_script_brave.py
```

**Note:** If building behind a corporate firewall or in a restricted network environment, ensure that the following domains are accessible during build:
- `brave-browser-apt-release.s3.brave.com` (for Brave browser installation)
- `storage.googleapis.com` (for ChromeDriver download)
- `googlechromelabs.github.io` (for ChromeDriver version lookup)

## 4. Dockerfile.firefox (Firefox Browser)

**Recommended for: Firefox-specific testing, Gecko engine compatibility, Mozilla standards**

- Uses multi-stage build for optimization
- Downloads specific Firefox version (136.0)
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

## Which Should You Use?

- **Use `Dockerfile`** if:
  - You're just getting started
  - You're behind a corporate firewall
  - You don't need the absolute latest Chrome version
  - You want faster, simpler builds

- **Use `Dockerfile.chrome`** if:
  - You're deploying to production
  - You need a specific Chrome version
  - You need maximum compatibility
  - You have unrestricted internet access during builds

- **Use `Dockerfile.brave`** if:
  - You need privacy-focused browsing automation
  - You want built-in ad-blocking capabilities
  - You're testing websites with Brave browser specifically
  - You prefer Brave's Chromium-based features

- **Use `Dockerfile.firefox`** if:
  - You need to test with Firefox/Gecko engine specifically
  - You need Mozilla-specific WebDriver features
  - You're validating cross-browser compatibility
  - You prefer Firefox's rendering and standards compliance

## 5. Dockerfile.ubuntu (Ubuntu Desktop - Enhanced GUI/Window Management)

**Recommended for: PJeOffice certificate dialogs, complex window interactions, GUI automation, maximum Ubuntu compatibility**

- Uses Ubuntu 22.04 LTS as base image (instead of Debian slim)
- Comprehensive desktop environment and GUI libraries
- Enhanced window management tools (wmctrl, xdotool, xautomation)
- Full GTK2/GTK3 support for certificate and authentication dialogs
- D-Bus and PolicyKit support for system dialogs
- AT-SPI accessibility for complex GUI interactions
- Audio support (PulseAudio) for multimedia dialogs
- More closely matches native Ubuntu development environment
- Larger image size but maximum compatibility
- **Requires internet access to dl.google.com and storage.googleapis.com during build**

**Build command:**
```bash
docker build -f Dockerfile.ubuntu -t rpa-worker-selenium-ubuntu .
```

**Build with PJeOffice support:**
```bash
docker build -f Dockerfile.ubuntu --build-arg BUILD_PJEOFFICE=1 -t rpa-worker-selenium-ubuntu-pje .
```

**Example usage:**
```bash
# Basic usage
docker run --rm rpa-worker-selenium-ubuntu example_script.py

# With GUI services enabled for PJeOffice
docker run --rm \
  -e USE_XVFB=1 \
  -e USE_OPENBOX=1 \
  -e USE_PJEOFFICE=1 \
  rpa-worker-selenium-ubuntu-pje my_pjeoffice_script.py
```

**Note:** This image is specifically designed to handle:
- PJeOffice certificate password dialogs
- Complex window interactions requiring full desktop environment
- GTK-based authentication prompts
- Applications requiring complete Ubuntu environment compatibility

- **Use `Dockerfile.ubuntu`** if:
  - You need to handle PJeOffice certificate password dialogs
  - You're experiencing window management issues with other images
  - You need maximum Ubuntu environment compatibility
  - Your Python code runs perfectly on Ubuntu but has issues in slim containers
  - You need full desktop environment support for complex GUI interactions
  - You're working with GTK-based authentication dialogs
  - Image size is less important than compatibility

All versions include all the same Python packages and provide the same functionality for Selenium automation.
