# Roadmap

Planned work for SiteRover, roughly in the order it will be tackled. Nothing here is promised for a specific release. Ideas and pull requests are welcome; for bigger items, please open an issue to discuss the approach first.

Terminology used below: the main control bar is the **panel**. Everything else that floats over a slide (queue, address bar, labels, and the new types below) is a **widget**.

## 1. Project setup

- [ ] README, configuration reference and kiosk-setup guide
- [ ] CONTRIBUTING, CODE_OF_CONDUCT, SECURITY and CHANGELOG
- [ ] CI that runs the in-browser test page on every pull request
- [ ] Release workflow that publishes a single downloadable `siterover.html`
- [ ] GitHub Pages demo

## 2. Split the source, keep one download

- [ ] Move the code into `src/` (config, widgets, edge tools, settings, …), loaded by `index.html` as classic scripts so development still works by opening the file from disk
- [ ] Zero-dependency build script that inlines everything into one `siterover.html` for releases

## 3. Widget system

Most items in section 4 depend on this, so it comes first.

- [ ] Each slide holds a list of widgets (type, position, size, style, content) instead of one fixed label plus fixed panels; config format v2 with automatic upgrade of v1 files
- [ ] Nothing can be placed even partly off screen: positions are kept inside the viewport while dragging and resizing, on load, and when the window size changes
- [ ] Store positions relative to the screen size, so one config looks right on both 1080p and 4K screens
- [ ] Close (X) button in the hover toolbar of every widget (not the panel). Built-in widgets are hidden (the queue's X works like its show/hide toggle); widgets you created are deleted, with a few seconds to undo

## 4. New widgets

- [ ] Slide counter (`3 / 6`)
- [ ] Minimal progress bar showing time until the next slide; pauses with rotation, restarts on manual navigation
- [ ] More than one text box per slide
- [ ] Rich text: bold, italic, underline, strikethrough and clickable links. Needs a strict allow-list sanitizer (only those formats and `http(s)` links) on save, load and import. Single click follows a link, double click edits; a small formatting toolbar appears on text selection
- [ ] Quick link button (opens an address in a new tab)
- [ ] Floating image, for reference. By URL or a file next to the config file (keeps configs portable); local uploads stored in IndexedDB, embedded on export
- [ ] Shapes: line, arrow, rectangle and ellipse, with color, thickness and rotation. Freehand drawing in a later step
- [ ] Clock and date, countdown, scrolling ticker

## 5. Main panel

- [ ] Up/down picker for the time per slide
- [ ] Choose which buttons are shown (Settings always stays)
- [ ] Duplicate the current slide, including its layout and widgets, to build a different arrangement of the same page
- [ ] Slides with exactly the same address share one loaded page (a single iframe) instead of loading it once per slide. Switching between them keeps the page and only changes the widgets. Because the page is shared, navigating inside it or reloading it affects every slide using it, so each slide gets an option to keep its own separate copy

## 6. Automatic refresh

- [ ] Refresh (normal reload) or reset (reload bypassing the cache) on a timer, for one slide or for all slides, every N minutes
- [ ] Hidden slides refresh in the background. The visible slide loads a fresh copy behind the current one and crossfades to it, so a refresh never flashes a blank page
- [ ] A refresh also clears any navigation done inside the page, like Reload Current does today

## 7. Sites that forbid embedding

Instead of the browser's "refused to connect" page, show a card that explains why the slide is empty and how to fix it.

A page cannot detect this by itself. Tested in Chrome 157 and Firefox 160: a blocked frame and a working frame from another site look identical to the page that embeds them (the same load event, the same access errors, no status code). Detection therefore has to come from somewhere else:

- [ ] Built-in list of popular sites known to forbid embedding (Google, YouTube watch pages, X, Facebook, LinkedIn, GitHub, zoom.earth, …). Their slides show the explanation card instead of loading: what happened, how to install a header-stripping extension in a browser profile used only for SiteRover, and a link to the docs
- [ ] "Load anyway" button on the card, and a setting "I use a header-stripping extension" that turns the card off, for people who already strip the headers
- [ ] Suggest the embeddable address when a site has one (YouTube `watch?v=` → `embed/`, OpenStreetMap → `export/embed.html`, published Google Calendar/Docs/Slides links), on the card and as soon as the address is typed in Settings
- [ ] With the companion extension installed: exact detection for every site, since the extension can see the response headers. The card then appears only when a site really refused and the extension is switched off for it

## 8. Sound

Browsers do not let a page control the audio of embedded pages from other sites, which is why the mute button only works for some players.

- [ ] Player adapters for YouTube, Vimeo and PeerTube: mute current, mute all, and a volume slider
- [ ] Only the visible slide plays sound. Today every slide is loaded at once, so a video on a hidden slide can be heard; pause or mute hidden players, with optional unloading of hidden slides
- [ ] Companion browser extension: tab-wide mute that works for any site, and optional removal of embedding blocks for SiteRover's own frames only

## 9. Kiosk and signage

- [ ] Different duration per slide
- [ ] Lock mode for public screens: no dragging, editing or settings; unlock with a PIN or a URL parameter
- [ ] Hide all controls after a period without mouse movement
- [ ] Zoom and crop a slide: show only one part of a page, or scale a 1080p dashboard up for a 4K screen
- [ ] Schedules: show some slides only at certain hours or on certain days
- [ ] TV-remote and keyboard shortcuts (arrows, space, number keys to jump to a slide)
- [ ] Re-read the config file every N minutes and apply changes without reloading, so one edited file updates every screen
- [ ] Snap-to-grid, alignment guides, undo/redo for layout changes
- [ ] Transition choices: crossfade, slide, none
- [ ] Translations, starting with Spanish

## Known limitations

These come from browser security rules, not from SiteRover, and are unlikely to change without the companion extension:

- After you navigate inside a page from another site, SiteRover cannot see its new address.
- SiteRover cannot mute or change the volume of pages from other sites, except players that offer a remote-control API.
- Sites that forbid being embedded stay blank unless a header-stripping extension is installed, and SiteRover cannot tell on its own that this happened (see section 7 for the workarounds).
