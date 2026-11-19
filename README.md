# SiteRover 🚙

> A lightweight, zero-dependency, local-first URL rotator and iframe dashboard for TV displays and kiosks.

SiteRover allows you to turn any web browser into a seamless, automated digital signage dashboard. It rotates through multiple URLs using cross-fading iframes to prevent loading flashes, and features a fully movable, drag-and-drop GUI for on-the-fly configuration. 

All settings are saved locally to your browser, with full support for JSON import/export to easily deploy the same dashboard across multiple TVs or mini PCs.

## ✨ Features

* **Zero Dependencies:** A single, lightweight HTML/JS/CSS file. No Node.js, no bundlers, no servers required.
* **Seamless Transitions:** URLs load in the background before displaying, utilizing smooth CSS opacity crossfades.
* **Persistent GUI:** Draggable, resizable UI panels (Main Controls, Settings, Loop Queue) that remember their positions.
* **Live Customization:** Click any on-screen label to edit it inline. Change opacity, timing, and toggle the queue visibility instantly.
* **JSON Portability:** Export your entire layout (panel positions, labels, URLs, timers) to a `.json` file and import it on another device.
* **Advanced Reloading:** Soft reload or hard cache-busting reset for individual pages or the entire dashboard.
* **Baseline Memory:** Restore panels to their original imported coordinates at any time.

## ⚠️ Prerequisites (Crucial for Map/Weather Dashboards)

Many modern websites (like Zoom Earth, Nullschool, Google Calendar, etc.) block other sites from embedding them using `X-Frame-Options` or Content Security Policy (CSP) headers. 

Because SiteRover runs in a local browser, you **must** install a browser extension to bypass these headers for the iframes to load correctly.

**For Firefox (Recommended for Linux Mini PCs):**
Install [Ignore X-Frame-Options Header](https://addons.mozilla.org/en-US/firefox/addon/ignore-x-frame-options-header/) or a similar extension.

## 🚀 Quick Start

1. Download the `dashboard.html` file.
2. Install the header-stripping extension in your browser (see Prerequisites).
3. Double-click `dashboard.html` to open it in your browser.
4. Press `F11` for full-screen kiosk mode.

## ⚙️ Configuration (JSON)

SiteRover saves your settings automatically to `localStorage`. You can also configure your dashboard by creating a `config.json` file and importing it via the Settings panel.

Example configuration structure:
```json
{
  "interval": 10000,
  "opacity": 0.85,
  "pages": [
    {
      "url": "[https://zoom.earth/](https://zoom.earth/)...",
      "label": "Satellite Map",
      "style": { "top": "20px", "left": "20px", "width": "200px" }
    },
    {
      "url": "[https://earth.nullschool.net/](https://earth.nullschool.net/)...",
      "label": "Wind Patterns",
      "style": { "top": "20px", "left": "20px", "width": "200px" }
    }
  ],
  "layout": {
    "controlPanel": { "top": "90%", "left": "80%" },
    "settingsPanel": { "top": "20%", "left": "30%" },
    "queuePanel": { "top": "10%", "left": "10%" }
  }
}