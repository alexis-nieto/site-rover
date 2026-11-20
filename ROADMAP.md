# Roadmap

Planned work for SiteRover, roughly in the order it will be tackled. Nothing here is promised for a specific release. Ideas and pull requests are welcome; for bigger items, please open an issue to discuss the approach first.

Terminology used below: the main control bar is the **panel**. Everything else that floats over a slide (queue, address bar, labels, and the new types below) is a **widget**.

## 1. Project setup and documentation

- [ ] README, configuration reference and kiosk-setup guide
- [ ] "Sites that refuse to load" guide, in plain language: why some sites forbid being shown inside other pages, what a header-stripping extension does about it, what it costs in security (so: use a browser profile only for SiteRover), and the options per browser: Firefox, Chromium-based browsers (Chrome, Edge, Brave, Vivaldi, …), and browsers without WebExtension support (Pale Moon, surf, …) where no extension route exists
- [ ] "Logins inside slides" guide: whether a site you are logged into elsewhere stays logged in inside SiteRover depends on the browser's third-party cookie rules (Chrome and Edge usually yes, Firefox keeps a separate login for embedded sites, Safari and Brave usually no), plus the privacy risk of logged-in pages on a public screen
- [ ] CONTRIBUTING, CODE_OF_CONDUCT, SECURITY and CHANGELOG
- [ ] CI that runs the in-browser test page on every pull request
- [ ] Release workflow that publishes a single downloadable `siterover.html`
- [ ] GitHub Pages demo

## 2. Split the source, keep one download

Approach: plain files for development, one generated file for users. No bundler, no framework, no npm dependencies.

- [ ] Move the code into `src/`: `css/` (base, panel, widgets, settings) and `js/` (config, storage, layout and fitting, edge tools, slideshow, queue, settings, start-up), each a few hundred lines
- [ ] `index.html` at the repository root loads them with ordinary `<link>` and `<script src>` tags. That works when the file is opened straight from disk, so contributors edit and reload with no tools at all. ES modules are deliberately avoided: browsers refuse to load them from disk
- [ ] `build.mjs`: a dependency-free Node script (about 40 lines) that inlines every CSS and JS file into one `dist/siterover.html`. The release workflow runs it, so most contributors never have to
- [ ] CI runs the test page against both the source version and the built file

## 3. Widget system

Most items in sections 4 and 5 depend on this, so it comes first.

- [ ] One rule for what is per slide and what is global, applied everywhere: every layout and appearance setting has a global default, and a slide can override it. Every editing action says which one it changes ("this slide" / "all slides", like the queue menu does), and Settings shows overridden values with a way to reset them to the global default
- [ ] Each slide holds a list of widgets (type, position, size, style, content) instead of one fixed label plus fixed panels. New config format; no migration needed since there are no existing users, and the prototype carry-over code goes away
- [ ] Every widget has a scope. **All slides**: it stays on screen through slide changes without fading (a ticker keeps scrolling instead of restarting), and a slide can hide it or move it, the way the queue can be hidden per slide today. **This slide only**: it fades in and out with its slide. Defaults: countdown, ticker, clock, slide number and position bar live on all slides; text boxes, shapes, images and quick links on one slide
- [ ] Colors per widget (background, text, border), per slide or global
- [ ] Nothing can be placed even partly off screen: positions are kept inside the viewport while dragging and resizing, on load, and when the window size changes
- [ ] Store positions relative to the screen size, so one config looks right on both 1080p and 4K screens
- [ ] Close (X) button in the hover toolbar of every widget (not the panel). Built-in widgets are hidden (the queue's X works like its show/hide toggle); widgets you created are deleted, with a few seconds to undo

## 4. New widgets

- [ ] Slide number: the current position alone or with the total (`3` or `3 / 6`)
- [ ] Position bar: a thin bar showing where the current slide sits in the queue (one segment per slide, the current one highlighted)
- [ ] More than one text box per slide
- [ ] Rich text: bold, italic, underline, strikethrough and clickable links. Needs a strict allow-list sanitizer (only those formats and `http(s)` links) on save, load and import. Single click follows a link, double click edits; a small formatting toolbar appears on text selection
- [ ] Quick link button (opens an address in a new tab)
- [ ] Floating image, for reference. By URL or a file next to the config file (keeps configs portable); local uploads stored in IndexedDB, embedded on export
- [ ] Shapes: line, arrow, rectangle and ellipse, with color, thickness and rotation. Freehand drawing in a later step
- [ ] Clock and date
- [ ] Countdown: time remaining until a date you set ("Opening in 2 d 04:13:22"). At zero it shows a message of your choice, hides itself, or counts up ("+00:03:12"). Format automatic (days when far away, minutes and seconds when close) or fixed. Example uses: a maintenance window over monitoring dashboards, a storm's expected arrival next to weather maps, a sale or exam ending
- [ ] Ticker: one or more messages that scroll continuously across the screen, joined by a separator. A full-width strip along the bottom or top by default, movable and resizable like any widget; text scales to the strip's height; adjustable speed and direction; pauses while hovered. Example uses: incident status in an operations room, notices in a lobby, specials in a shop. Text is typed in; live news or RSS feeds are mostly blocked by the same cross-site rules as other embedded content

## 5. Panel, playback and settings

- [ ] Ask before starting: the first time a config is loaded (the default one, a changed config file, or an import), show a small "Ready to start?" dialog instead of rotating right away. Once confirmed, later loads of the same config start on their own, so a screen still recovers unattended after a power cut. `?autostart=1` in the page address skips the question
- [ ] Time per slide, with a global default and per-slide overrides. In Settings: a duration for each slide plus "set for all"; on the panel: an up/down picker that changes the current slide or all slides
- [ ] Choose which buttons are shown on the panel (Settings always stays)
- [ ] "Reset" next to Import and Export: put everything (slides, order, layout, widgets, settings) back to the last import, or to the config file if nothing was imported. Asks for confirmation in the dialog itself
- [ ] Duplicate the current slide, including its layout and widgets, to build a different arrangement of the same page
- [ ] Linked and separate duplicates. Slides with exactly the same address are **linked** by default: they share one loaded page (one iframe), which saves memory and makes switching between them seamless, since the page stays put and only the widgets change. Navigating inside a linked page, reloading it or refreshing it on a timer affects every slide linked to it. A linked slide shows a small link icon in the queue and in Settings
  - "Detach" turns a linked slide into a **separate** copy with its own page (costs the memory of a second page); "Duplicate as separate copy" does this from the start
  - Changing the address of a linked slide detaches it automatically
  - The config only stores the exception (`"detached": true`)

## 6. Automatic refresh

- [ ] Refresh (normal reload) or reset (reload bypassing the cache) on a timer, for one slide or for all slides, every N minutes
- [ ] Hidden slides refresh in the background. The visible slide loads a fresh copy behind the current one and crossfades to it, so a refresh never flashes a blank page
- [ ] A refresh also clears any navigation done inside the page, like Reload Current does today

## 7. Sites that forbid embedding

Instead of the browser's "refused to connect" page, show a card that explains why the slide is empty and how to fix it.

A page cannot detect this by itself. Tested in Chrome 157 and Firefox 160: a blocked frame and a working frame from another site look identical to the page that embeds them (the same load event, the same access errors, no status code). Detection therefore has to come from somewhere else:

- [ ] Built-in list of popular sites known to forbid embedding (Google, YouTube watch pages, X, Facebook, LinkedIn, GitHub, Steam, zoom.earth, …). Their slides show the explanation card instead of loading: what happened, what a header-stripping extension does and why it is needed, how to get one for the browser in use, and a link to the guide (section 1). Not tied to one browser or one extension
- [ ] "Load anyway" button on the card, and a setting "I use a header-stripping extension" that turns the card off, for people who already strip the headers
- [ ] Suggest the embeddable address when a site has one (YouTube `watch?v=` → `embed/`, OpenStreetMap → `export/embed.html`, published Google Calendar/Docs/Slides links), on the card and as soon as the address is typed in Settings
- [ ] Companion extension of our own (one WebExtension for Firefox, Chrome, Edge, Brave, Vivaldi, …) that removes the block only for frames inside SiteRover, instead of for every site you visit. It can also see the response headers, so with it installed SiteRover knows exactly which sites refused

## 8. Sound

Browsers do not let a page control the audio of embedded pages from other sites, so a mute button can only ever work for a few video players.

- [ ] Remove the mute button: it does nothing for almost every site, which is worse than not having it
- [ ] Per-slide option "Load only while visible": the page is unloaded when its slide is hidden. This stops hidden slides from playing sound (today every slide is loaded at once, so a video on a hidden slide can be heard) and saves memory, at the cost of that slide loading each time it appears
- [ ] Document how common players start muted from their address (YouTube and PeerTube `?mute=1` / `?muted=1`)
- [ ] Later, if there is demand: tab-wide mute through the companion extension (section 7), which works for any site

## 9. Kiosk and signage

- [ ] Lock mode for public screens: no dragging, editing or settings; unlock with a PIN or a URL parameter
- [ ] Hide all controls after a period without mouse movement
- [ ] Zoom and crop a slide: show only one part of a page, or scale a 1080p dashboard up for a 4K screen
- [ ] Schedules: show some slides only at certain hours or on certain days
- [ ] TV-remote and keyboard shortcuts (arrows, space, number keys to jump to a slide)
- [ ] Screens follow config changes on their own: when SiteRover is served from a web server, each screen re-checks `config.json` every few minutes and applies changes without a reload, so editing one file updates every screen in the building
- [ ] Snap-to-grid, alignment guides, undo/redo for layout changes
- [ ] Transition choices: crossfade, slide, none
- [ ] Translations, starting with Spanish

## 10. Split-screen slides (ambitious)

- [ ] A slide can show several pages at once: side by side, top and bottom, one large plus two small, 2×2, and so on
- [ ] Start from a layout template, then drag the dividers between pages to reshape the split; dividers snap to common ratios
- [ ] Each region keeps its own address, zoom/crop and refresh timer
- [ ] Trade-offs: every visible region is a live page (more memory), and a page shared between linked slides (section 5) cannot appear in two regions at once

## 11. Pointer and presentation tools

- [ ] Custom cursor (dot, ring, crosshair, …) with a chosen size and color
- [ ] Crosshair guides: a vertical and a horizontal line spanning the whole screen and meeting at the pointer, for reading values off charts
- [ ] Both work as a "pointer mode" switched from the panel or a key. Over a page from another site, the browser lets that page choose the cursor and does not tell SiteRover where the pointer is, so pointer mode puts a transparent layer over the slide that draws the cursor and guides; clicks don't reach the page while it is on

## Known limitations

These come from browser security rules, not from SiteRover, and are unlikely to change without the companion extension:

- After you navigate inside a page from another site, SiteRover cannot see its new address.
- SiteRover cannot mute or change the volume of pages from other sites.
- Sites that forbid being embedded stay blank unless a header-stripping extension is installed, and SiteRover cannot tell on its own that this happened (see section 7 for the workarounds).
- Whether you stay logged in to a site inside SiteRover is decided by the browser's cookie rules, not by SiteRover (see the guide in section 1).
- Over a page from another site, SiteRover cannot change the cursor or follow the pointer without blocking clicks to that page (see section 11).
