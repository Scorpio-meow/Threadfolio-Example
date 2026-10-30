<div align="center">

# Threadfolio

**Pure front-end web application for displaying, paginating, and organizing featured Threads posts with multi-column masonry layouts and automated rate-limiting protection**

[![License](https://img.shields.io/badge/license-MIT-yellow?style=for-the-badge)](./LICENSE)
[![PWA](https://img.shields.io/badge/PWA-Ready-brightgreen?style=for-the-badge&logo=pwa&logoColor=white)](./manifest.json)
[![Tech Stack](https://img.shields.io/badge/tech--stack-Vanilla--JS-blue?style=for-the-badge&logo=javascript&logoColor=white)](#tech-stack--specifications)
[![Dependencies](https://img.shields.io/badge/dependencies-0-success?style=for-the-badge)](#tech-stack--specifications)
[![Companion Repo](https://img.shields.io/badge/Companion_Repo-Threadkeep-8A2BE2?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Scorpio-meow/Threadkeep)
[![Live Demo](https://img.shields.io/badge/Live_Demo-GitHub_Pages-D12F35?style=for-the-badge&logo=githubpages&logoColor=white)](https://scorpio-meow.github.io/Threadfolio-Example/)

---

English | [繁體中文](./README.md)

Pure front-end vanilla architecture, zero heavy external frameworks, ready out of the box.  
Equipped with HTTP 429 rate-limiting exponential backoff, iframe rendering anomaly monitoring with graceful fallback, deterministic seeded shuffling,  
and Progressive Web App (PWA) offline caching for a seamless browsing and searching experience.

</div>

---

## Table of Contents

- [Project Overview & Core Values](#project-overview--core-values)
- [Ecosystem & Integration](#ecosystem--integration)
- [Quick Start](#quick-start)
  - [Prerequisites & Environment](#prerequisites--environment)
  - [1. Clone Repository](#1-clone-repository)
  - [2. Configure Post Data](#2-configure-post-data)
  - [3. Start Local Development Server](#3-start-local-development-server)
- [Core Features](#core-features)
  - [1. Real-Time Search & Tag Filtering](#1-real-time-search--tag-filtering)
  - [2. Dual Theme System & CSS Design Tokens](#2-dual-theme-system--css-design-tokens)
  - [3. Responsive CSS Masonry & Single-Column Layout](#3-responsive-css-masonry--single-column-layout)
  - [4. Automated Rate-Limiting Detection & Exponential Backoff](#4-automated-rate-limiting-detection--exponential-backoff)
  - [5. Chunked Non-Blocking Rendering & Shimmer Skeletons](#5-chunked-non-blocking-rendering--shimmer-skeletons)
  - [6. Deterministic Seeded Random Shuffling](#6-deterministic-seeded-random-shuffling)
  - [7. PWA Support & 32-bit djb2 Content Hash Caching](#7-pwa-support--32-bit-djb2-content-hash-caching)
  - [8. Iframe Health Observer, Background Tab Adaptation & Graceful Fallback](#8-iframe-health-observer-background-tab-adaptation--graceful-fallback)
  - [9. Single Post Isolated Preview Mode](#9-single-post-isolated-preview-mode)
  - [10. Live Loading Progress Panel](#10-live-loading-progress-panel)
  - [11. Global Console Noise Filter](#11-global-console-noise-filter)
  - [12. FAQ & Diagnostic Accordion Modal](#12-faq--diagnostic-accordion-modal)
- [Tech Stack & Specifications](#tech-stack--specifications)
- [Project Directory & File Structure](#project-directory--file-structure)
  - [File Listing](#file-listing)
  - [Module Responsibilities](#module-responsibilities)
- [System Architecture & Lifecycle Flows](#system-architecture--lifecycle-flows)
  - [System Module Architecture Diagram](#system-module-architecture-diagram)
  - [Post Load & Rate-Limiting Sequence Diagram](#post-load--rate-limiting-sequence-diagram)
- [Configuration & Data Specifications](#configuration--data-specifications)
  - [Runtime Configuration Table (RuntimeConfig)](#runtime-configuration-table-runtimeconfig)
  - [Post Data Schema (PostItem)](#post-data-schema-postitem)
  - [URL Query Parameters Specification (URLParams)](#url-query-parameters-specification-urlparams)
- [Key Algorithms & Technical Deep Dive](#key-algorithms--technical-deep-dive)
  - [1. Exponential Backoff Rate-Limiting Algorithm](#1-exponential-backoff-rate-limiting-algorithm)
  - [2. 32-bit djb2 Content Hash Cache Invalidation](#2-32-bit-djb2-content-hash-cache-invalidation)
  - [3. Deterministic Pseudo-Random Seeded Shuffle](#3-deterministic-pseudo-random-seeded-shuffle)
  - [4. Dual Iframe Observers & Fallback Substitution](#4-dual-iframe-observers--fallback-substitution)
- [FAQ & Troubleshooting](#faq--troubleshooting)
- [Development & Deployment Guide](#development--deployment-guide)
  - [Local Development Commands](#local-development-commands)
  - [Static Hosting Deployment](#static-hosting-deployment)
- [Changelog](#changelog)
- [AI-Friendly Documentation (llms.txt)](#ai-friendly-documentation-llmstxt)
- [License & Disclaimer](#license--disclaimer)

---

## Project Overview & Core Values

Official Threads embed iframes provide rich social interactivity. However, when rendering multiple embeds simultaneously, web pages frequently suffer from cross-origin authentication failures, HTTP 429 rate-limiting (Too Many Requests), main-thread blocking, and layout jitter.

**Threadfolio** is engineered to eliminate these bottlenecks:

| Core Value | Description |
| :--- | :--- |
| **Rate-Limit Resilience** | Intercepts HTTP 429 errors globally, applying exponential backoff and automated queue resumption. |
| **Graceful Degradation Guarantee** | Seamlessly substitutes broken or blocked iframes (height under 120px) with fallback author cards and direct links. |
| **Smooth Chunked Rendering** | Leverages `requestIdleCallback` and shimmer skeleton screens to mount posts progressively, optimizing INP. |
| **Instant Search & Tag Filtering** | Fast real-time fuzzy search matching authors, content, and hashtags with two-way URL query synchronization. |
| **Offline Cache & PWA** | Automated version invalidation via 32-bit djb2 content hashing, supporting full standalone app installation on desktop and mobile. |

---

## Ecosystem & Integration

This project acts as the **Presentation Layer (Downstream)** in tandem with the companion Chrome extension **[Threadkeep](https://github.com/Scorpio-meow/Threadkeep)**:

```mermaid
flowchart LR
    subgraph Upstream ["Upstream Collector"]
        Collector["Threadkeep\n(Chrome Extension)"]
        Parser["Post DOM Parsing & Noise Sanitization"]
        Export["Export Featured Data (threads-featured-data-*.js)"]
        Collector --> Parser --> Export
    end

    subgraph Downstream ["Downstream Presentation"]
        Config["config.js\n(Posts Dataset & Runtime Constants)"]
        Viewer["Threadfolio\n(Static Web App)"]
        Engine["Rate Limit Backoff / Tag Search / Masonry"]
        UI["Modern Presentation UI\n(Dark/Light Theme / PWA / Diagnostic Modal)"]
        Config --> Viewer --> Engine --> UI
    end

    Export -- "Copy data to overwrite posts array" --> Config
```

1. **Upstream Collector**: Use [Threadkeep](https://github.com/Scorpio-meow/Threadkeep) to bookmark posts while browsing Threads, clicking "Export Featured Data" to produce structured objects without the `@` prefix.
2. **Downstream Viewer**: Paste the exported array into [config.js](./config.js) in this project to instantly update your showcase site.

---

## Quick Start

### Prerequisites & Environment

- Any modern browser supporting modern web standards (Chrome, Edge, Safari, Firefox, Brave, Arc, etc.).
- A local or remote static web server (Bun, Node.js, Python, or static hosting providers).
- Zero compilation steps, zero bundlers, zero npm runtime dependencies.

### 1. Clone Repository

```bash
git clone https://github.com/Scorpio-meow/Threadfolio-Example.git
cd Threadfolio-Example
```

### 2. Configure Post Data

Open [config.js](./config.js) and update the `posts` array:

```javascript
const posts = [
    {
        embedCode: '<blockquote class="text-post-media" data-text-post-permalink="https://www.threads.com/@username/post/xxx">...</blockquote>',
        postLink: 'https://www.threads.com/@username/post/xxx',
        author: 'username',
        content: 'Plain text content of the post...',
        tags: ['JavaScript', 'WebDev']
    }
];
```

### 3. Start Local Development Server

You can choose any of the following methods to preview the project locally:

**Method A: VS Code / IDE "Live Server" Extension (Easiest)**
- Install the "Live Server" extension in VS Code.
- Right-click `index.html` and select "**Open with Live Server**" (default: `http://127.0.0.1:5500`).

**Method B: Start with Bun (Recommended CLI)**
```bash
bunx http-server -p 3000
```
Open `http://localhost:3000` in your browser.

**Method C: Start with Python (Fallback CLI)**
```bash
python -m http.server 3000
```

---

## Core Features

### 1. Real-Time Search & Tag Filtering
- **Real-Time Fuzzy Matching**: Instantly searches author handles, post text, and hashtags as you type.
- **Frequency-Sorted Tag Bar**: Automatically ranks tags by frequency in descending order with expand/collapse toggle support.
- **Two-Way URL State Sync**: Search terms (`?search=`) and active tags (`?tag=`) synchronize with URL query parameters for direct link sharing and bookmarking.

### 2. Dual Theme System & CSS Design Tokens
- **Dark / Light Mode Switching**: Instant header toggle with preference persistence in `localStorage`.
- **Threads Embed Theme Sync**: Each post card writes the current theme into its embed `blockquote`'s `data-theme` when it renders, so Threads embeds match the site; after a theme switch, already loaded embeds follow on the next page change or reload.
- **Centralized CSS Variables**: Palette, font weights, border radiuses, and easing curves are managed via CSS Custom Properties in `styles.css`.
- **Borderless Lightness Layering**: Surfaces are separated by 1%–3% lightness steps instead of borders (light `#f5f5f3` / `#fcfcfb`, dark `#0b0b0a` / `#151515` / `#1a1a19` / `#20201e`); pure black is never used.
- **Dark-Mode Weight Compensation**: Every font weight drops by 40 in dark mode (e.g. 400 → 360) so light text does not bloom; Manrope and Noto Sans TC are loaded as variable fonts for this.
- **Interaction Details**: Focused inputs get a double-ring `box-shadow` instead of an outline; the checkmark shown after copying a post link pops in on an underdamped spring (stiffness 380, damping 18) and fades back after 2.5 seconds; numbers use `tabular-nums`, and body text uses `text-wrap: pretty` to avoid orphans.
- **Scroll Entrance Motion**: Cards fade up as they scroll into view via a CSS scroll timeline (`animation-timeline: view()`); browsers without it (e.g. Firefox) and viewers who prefer reduced motion keep the plain load-in.

### 3. Responsive CSS Masonry & Single-Column Layout
- **Single-Column Focus Mode (default)**: Posts open in a centered single-column list for in-depth code reading.
- **Native CSS `columns` Masonry**: One-click toggle in the toolbar. Employs CSS multi-column layouts (`columns: 340px` inside a 1160px max-width container) to eliminate JavaScript reflow lag; the column count follows the available width (up to 3), and screens 576px wide or narrower always get 1 column.
- **Remembered Preference**: The chosen layout is stored in `localStorage` under the `layout` key.

### 4. Automated Rate-Limiting Detection & Exponential Backoff
- **HTTP 429 Error Interception**: Two detection paths:
  - `console-filter.js` traps console messages containing 429 / rate limit / too many requests and dispatches the global `threads:rate-limit` event.
  - `threads-loader.js` wraps `window.fetch` and `XMLHttpRequest`; a 429 response triggers the backoff directly, honoring the `Retry-After` response header when present.
- **Countdown Banner & Queue Pause**: Renders a live countdown banner on the progress panel and halts subsequent embed rendering.
- **Exponential Scaling**: Waits `RATE_LIMIT_BACKOFF × 1.5^(consecutive errors − 1)` (60 seconds by default, capped at 300s), safely resuming after timer expiry.

### 5. Chunked Non-Blocking Rendering & Shimmer Skeletons
- **requestIdleCallback Scheduling**: Mounts post card DOM elements during browser idle periods to prevent main-thread UI jank and improve Interaction to Next Paint (INP).
- **Shimmer Placeholder Animations**: Queued post cards are covered by an avatar, name, text and media skeleton (`.post-skeleton`) with a shimmer, hiding Threads' raw white placeholder; once Threads reports the height via `postMessage`, the card gets `post-item--ready` and the skeleton fades out to the real post. Failed posts switch straight to the fallback card.

### 6. Deterministic Seeded Random Shuffling
- **Timestamp Seed Generation**: Generates a timestamp seed stored in the URL (`?random=<seed>`).
- **Pagination & Refresh Consistency**: Uses a seeded pseudo-random formula ensuring identical post ordering across pagination clicks and browser reloads.

### 7. PWA Support & 32-bit djb2 Content Hash Caching
- **Progressive Web App**: Complete with `manifest.json` and Service Worker for desktop and mobile home screen installation.
- **Automatic djb2 Cache Invalidation**: The Service Worker reads core source files and hashes their content using a 32-bit djb2 algorithm, automatically clearing stale caches when code changes.

### 8. Iframe Health Observer, Background Tab Adaptation & Graceful Fallback
- **MutationObserver + ResizeObserver Dual Monitoring**: Real-time tracking of dynamically generated iframe node insertion and rendered height.
- **Background Tab Adaptation & Anti-Collapse Defense**:
  - Global CSS sets `.post-item iframe { min-height: 280px; }` as a placeholder until the real height arrives (Threads `embed.js` creates each iframe at `height: 0`), eliminating layout collapse into thin bars when the browser pauses background rendering.
  - Ignores initial `about:blank` load events, ensuring validation only executes once the genuine Threads URL (`threads.com`/`threads.net`) has loaded.
  - Automatically extends timeout to >= 25s when the tab is hidden (`document.hidden`); immediately triggers a `window.resize` broadcast and re-measurement upon returning to the foreground (`visibilitychange`).
  - Global `window.addEventListener('message', ...)` listener receives the content height Threads embed iframes post as a plain number (accepted only from `threads.com`/`threads.net` origins; legacy `MEASURE`/`HEIGHT` objects also supported) and pins both the iframe's `height` and `min-height` to it so cards hug their content, then adds `post-item--ready` to the card to remove the skeleton; reports of 0 are skipped to prevent collapse.
- **Automated Fallback Replacement**: If an iframe fails to render, times out (exceeding `IFRAME_TIMEOUT`), is rejected by `X-Frame-Options`, or stays under 120px in height, it is replaced with a graceful fallback card showing the failure reason and a direct Threads link.
- **Error Page Detection**: For posts Threads refuses to embed (e.g. sensitive or login-only posts), the embed endpoint returns 500 and shows Facebook's "Sorry, something went wrong" page. Rules:
  - A working embed has Threads post a height greater than 0 via `postMessage`, while the error page posts nothing; only that height report counts as success.
  - Every other message is ignored, including custom messages that browser extensions post after injecting into the iframe (they also come from `threads.com`, so they cannot be trusted as Threads signals).
  - Without a report, `IFRAME_TIMEOUT` usually fails the embed first; with a longer `IFRAME_TIMEOUT`, the embed also fails if no report arrives within 8 seconds of loading (`EMBED_REPORT_TIMEOUT`).
  - The timer only runs in the foreground; hidden tabs let the queue continue and re-judge the embed once the tab is visible again.

### 9. Single Post Isolated Preview Mode
- **Direct URL Parameter Routing**: Navigate via `?post=<URL>` or `?post=<Index>` to isolate an individual post while hiding header controls, tag bars, and pagination.
- **One-Click Share URL**: Built-in button to copy the direct sharing link.

### 10. Live Loading Progress Panel
- **Telemetry Metrics**: Real-time display of page load percentage and 3 essential time metrics:
  1. **Next Post Countdown**: Uniformly fixed at a 3.6s countdown (except during Meta rate-limit cooldown where a 60s timer is shown).
  2. **Estimated Total Duration**: Smoothly estimated based on remaining unstarted posts and average load time.
  3. **Total Load Elapsed Time**: Dynamic ticking during progress, locking the final duration upon 100% completion.
- **Tabular Figures**: The panel refreshes every 0.1 seconds, and `tabular-nums` keeps every digit the same width so the countdowns do not jitter.

### 11. Global Console Noise Filter
- **Noise Suppression**: Outside debug mode, `console-filter.js` filters out `cdninstagram.com` 404 resource errors, favicon 404s, and `threads.com` "Failed to load resource" errors.
- **Event Forwarding**: Rate-limit messages dispatch `threads:rate-limit` and `X-Frame-Options: deny` refusals dispatch `threads:xframe-block` for `threads-loader.js` to handle; with `?debug=1` these messages are still printed.

### 12. FAQ & Diagnostic Accordion Modal
- **Embedded FAQ Dialog**: Accordion modal covering why posts fail to display and rate limit best practices.
- **Grid Collapse Animation**: Answers expand and collapse via `grid-template-rows: 0fr → 1fr` with no JavaScript height measuring; links and buttons inside a collapsed answer are skipped by the Tab key.
- **One-Click Debug Mode**: Toggle `?debug=1` directly from the UI to unmask all console warnings and timing logs.

---

## Tech Stack & Specifications

```
+-----------------------------------------------------------------------+
|                         Technical Standards                           |
+-----------------------------------------------------------------------+
|  Frontend Core    | Vanilla JavaScript (ES6+), HTML5, CSS3 Variables  |
|  Typography       | Google Fonts, variable (Manrope, Noto Sans TC)    |
|  Layout Engine    | Native CSS Columns Masonry + CSS Grid + Flexbox   |
|  Offline Cache    | Service Worker API (32-bit djb2 Auto Invalidation)|
|  App Delivery     | Progressive Web App (Web App Manifest)            |
|  Dependencies     | 0 External Runtime Dependencies                   |
|  Hosting Target   | Any static web host (GitHub Pages, Vercel, etc.)  |
+-----------------------------------------------------------------------+
```

---

## Project Directory & File Structure

### File Listing

```
Threadfolio-Example/
├── index.html            # Main semantic HTML structure & modals
├── config.js             # Posts dataset (posts array) & runtime constants
├── threads-loader.js     # Core runtime controller (queue, backoff, DOM, metrics)
├── console-filter.js     # Error interceptor (suppress 404 noise, trap 429 events)
├── styles.css            # Design token system (CSS variables, masonry, themes)
├── sw.js                 # PWA Service Worker (djb2 hash invalidation, offline cache)
├── manifest.json         # PWA Manifest specification
├── assets/               # Static icon assets (Favicon, App Icons)
├── llms.txt              # AI-friendly architecture & RAG indexing specification
├── README_EN.md          # English documentation (This file)
└── README.md             # Traditional Chinese documentation
```

### Module Responsibilities

| File Path | Layer | Key Responsibilities |
| :--- | :--- | :--- |
| `index.html` | View Layer | Declares semantic HTML layout, search inputs, theme toggles, pagination, and modals. |
| `config.js` | Data & Config Layer | Stores the posts dataset array and 11 runtime configuration constants. |
| `threads-loader.js` | Controller Layer | Manages URL routing, pagination, queue scheduling, backoff timers, DOM rendering, and fallbacks. |
| `console-filter.js` | Interceptor Layer | Intercepts console logs, suppresses 404 warnings, and fires custom events on rate limits and X-Frame-Options refusals. |
| `styles.css` | Styling Layer | Defines CSS custom properties, dark/light themes, masonry columns, and animations. |
| `sw.js` | Service Worker Layer | Implements djb2 hash versioning for cache busting and offline asset serving. |
| `llms.txt` | AI Specification Layer | Provides structured project metadata and algorithm specifications for AI agents. |

---

## System Architecture & Lifecycle Flows

### System Module Architecture Diagram

```mermaid
flowchart TD
    subgraph Browser ["Browser Client Context"]
        HTML["index.html\n(Semantic UI & Modal)"]
        CSS["styles.css\n(Theme Tokens & Masonry)"]
        Manifest["manifest.json\n(PWA Config)"]
    end

    subgraph Scripts ["Runtime Engine"]
        Filter["console-filter.js\n(Error Trapping & Noise Filter)"]
        Config["config.js\n(Dataset & Constants)"]
        Loader["threads-loader.js\n(Scheduler, Backoff & DOM Rendering)"]
    end

    subgraph ServiceWorker ["Background Worker"]
        SW["sw.js\n(Asset Cache & djb2 Hash Versioning)"]
    end

    HTML --> Filter
    HTML --> Config
    HTML --> Loader
    HTML --> CSS
    HTML --> Manifest

    Filter -- "Dispatch threads:rate-limit / threads:xframe-block events" --> Loader
    Config -- "Inject posts & constants" --> Loader
    Loader -- "Render cards & progress bar" --> HTML
    SW -- "Cache static assets & intercept fetch" --> HTML
```

### Post Load & Rate-Limiting Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    participant UI as Page UI / DOM
    participant TL as threads-loader.js
    participant CF as console-filter.js
    participant API as Threads Official Server

    UI->>TL: Initialize page (calculate page from URL query)
    TL->>UI: Render Skeleton Shimmer Placeholders
    TL->>UI: Initialize Progress Tracker Panel

    loop Staggered Queue Loading (BATCH_SIZE = 1)
        TL->>UI: Mount blockquote embed node
        TL->>API: Dynamic load /embed.js to process iframe
        alt Load Succeeded
            API-->>UI: iframe rendered (height > 120px)
            TL->>UI: Update progress bar & completion timer
        else HTTP 429 Encountered
            API-->>CF: Trigger 429 Too Many Requests
            CF->>TL: Dispatch threads:rate-limit event
            TL->>UI: Display countdown banner & pause queue
            Note over TL,UI: Exponential backoff delay (default 60s x 1.5)
            TL->>TL: Backoff elapsed, resume queue
        else Timeout / Height Block (< 120px) / No Height Report (Error Page)
            TL->>UI: Graceful fallback to direct link card
        end
    end
    TL->>UI: All posts rendered, display total duration
```

---

## Configuration & Data Specifications

### Runtime Configuration Table (RuntimeConfig)

Customizable constants in [config.js](./config.js):

| Setting Constant | Type | Default | Description & Best Practice |
| :--- | :---: | :---: | :--- |
| `LOAD_DELAY` | number | `4000` | Base load delay between embed requests (ms). The code enforces a 4000ms floor; smaller values are raised to 4000ms. |
| `BATCH_SIZE` | number | `1` | Maximum concurrent iframe loads. The code caps it at 1; values above 1 have no effect. |
| `EMBED_STAGGER_DELAY` | number | `3600` | Stagger interval between adjacent embed loads (ms). The code enforces a 3600ms (3.6s) floor; smaller values are raised to 3600ms. |
| `IFRAME_TIMEOUT` | number | `3600` | Timeout threshold for individual iframe renders (ms). Triggers fallback on expiry. |
| `MIN_IFRAME_TIMEOUT` | number | `8000` | Initial presence check safety duration (ms). |
| `RATE_LIMIT_BACKOFF` | number | `60000` | Base backoff on 429 error (ms). Escalates by 1.5x up to 300s ceiling. |
| `MAX_DELAY` | number | `60000` | Upper ceiling for dynamic delay calculation (ms). |
| `MIN_DELAY_BETWEEN_REQUESTS` | number | `3600` | Reserved; not read by the code (the request interval is governed by `EMBED_STAGGER_DELAY`). |
| `MAX_VISIBLE_QUEUE` | number | `30` | Reserved; not read by the code. |
| `PAGE_SIZE_OPTIONS` | number[] | `[1, 5, 10, 25, 50, 100, 1000]` | Selectable options in the page size dropdown. |
| `PAGE_SIZE` | number | `10` | Default number of items rendered per page. Must be one of `PAGE_SIZE_OPTIONS`; otherwise the first option is used. |

### Post Data Schema (PostItem)

TypeScript interface for items in the `posts` array:

```typescript
interface PostItem {
  /** Threads official blockquote HTML string */
  embedCode: string;

  /** Full original URL to the Threads post */
  postLink: string;

  /** Author username handle without @ */
  author: string;

  /** Plaintext post content */
  content: string;

  /** Array of hashtag keywords */
  tags: string[];

  /** Whether the post embeds for logged-out viewers (written by the browser extension's check); when false the iframe is still attempted and only a failure shows the "login-only" explanation; absent means unchecked */
  embeddable?: boolean;
}
```

### URL Query Parameters Specification (URLParams)

| Parameter | Type | Example | Description |
| :--- | :---: | :--- | :--- |
| `page` | integer | `?page=2` | Target page index (1-based). |
| `page_size` | integer | `?page_size=25` | Number of items per page. |
| `random` | string | `?random=1717750000000` | Timestamp seed for deterministic random shuffle. |
| `search` | string | `?search=frontend` | Search keyword matching author, content, or tags. |
| `tag` | string | `?tag=React` | Exact hashtag filter (excluding `#`). |
| `post` | string | `?post=https://www.threads.com/@username/post/xxx` | Single post preview mode (URL or index). |
| `debug` | string | `?debug=1` | Enable debug logging in console. |

---

## Key Algorithms & Technical Deep Dive

### 1. Exponential Backoff Rate-Limiting Algorithm

When Meta returns HTTP 429, `handleRateLimit` in `threads-loader.js` pauses the queue, removes in-flight iframes, and computes the backoff (excerpt):

```javascript
function handleRateLimit(source, overrideBackoffMs) {
    if (rateLimitDetected) return;
    rateLimitDetected = true;
    paused = true;
    consecutiveErrors++;
    // Use Retry-After when present, otherwise scale by consecutive errors; both capped at 300s
    var backoffTime = typeof overrideBackoffMs === 'number' && overrideBackoffMs > 0 ?
        Math.min(overrideBackoffMs, 300000) :
        Math.min(RATE_LIMIT_BACKOFF * Math.pow(1.5, consecutiveErrors - 1), 300000);
    rateLimitEndTime = Date.now() + backoffTime;
    currentDelay = Math.min(currentDelay * 2, MAX_DELAY);
    showRateLimitBanner(backoffTime);
    setTimeout(function () {
        rateLimitDetected = false;
        paused = false;
        pumpEmbeds();
        hideRateLimitBanner();
    }, backoffTime);
}
```

Each successful `fetch` or XHR response decrements `consecutiveErrors` by 1, so the backoff gradually falls back.

### 2. 32-bit djb2 Content Hash Cache Invalidation

In `sw.js`, a 32-bit hash is computed across core source files:

```javascript
function djb2Hash(str) {
  let hash = 5381;
  for (let i = 0; i < str.length; i++) {
    hash = ((hash << 5) + hash) ^ str.charCodeAt(i);
    hash = hash & hash; // Convert to 32-bit integer
  }
  return (hash >>> 0).toString(16);
}
```

### 3. Deterministic Pseudo-Random Seeded Shuffle

Ensures consistent post ordering across page reloads and pagination. A seeded linear congruential generator (LCG) drives a Fisher–Yates shuffle that returns a new array and leaves the source data untouched:

```javascript
function shuffleWithSeed(array, seed) {
    var m = array.length, t, i;
    var copy = array.slice();
    while (m) {
        seed = (seed * 9301 + 49297) % 233280;
        i = Math.floor((seed / 233280) * m--);
        t = copy[m];
        copy[m] = copy[i];
        copy[i] = t;
    }
    return copy;
}
```

### 4. Dual Iframe Observers & Fallback Substitution

To handle blocked iframes without breaking page layouts:

1. **MutationObserver** detects iframe DOM element creation.
2. **ResizeObserver** monitors rendered height; if height remains under 120px upon timeout, it is swapped for a graceful fallback card.
3. **postMessage Report Check**: Once the height passes, the loader confirms Threads has posted a height report greater than 0 for the iframe; no report within the timeout swaps in the fallback card.

---

## FAQ & Troubleshooting

> [!WARNING]
> **Issue 1: Why do some post cards show "貼文載入失敗" (post failed to load) with only a "在 Threads 上查看" (View on Threads) button?**
> - Public posts need neither a Threads login nor Do Not Track turned off: they display while logged out and with Do Not Track on, and `embed.js` never reads that setting.
> - Posts Threads does not embed publicly: deleted posts, private accounts, or posts limited to logged-in viewers (e.g. marked as sensitive). For logged-out viewers their `/embed` returns HTTP 500. Login-only posts display only when the viewer is logged into Threads in the same browser and third-party cookies are allowed (Safari and incognito windows block them by default); an expired Threads session also fails until you log in again. Everyone else can only view them on Threads.
> - When an ad or privacy blocker (e.g. uBlock Origin, AdGuard) blocks `threads.com/embed.js`, every post times out and turns into a fallback card; whitelist `threads.com` and `cdninstagram.com`.
> - Slow connections or rate limiting can also make individual posts time out (see Issue 2).

> [!NOTE]
> **Issue 2: What is "Rate Limiting" and how does the application handle it?**
> - Rapid embed requests trigger HTTP 429 from Meta.
> - The application automatically displays a countdown banner, pauses the queue, and resumes after exponential backoff.
> - To minimize rate limits, decrease `PAGE_SIZE` (e.g. 3 or 5) or increase `LOAD_DELAY` and `EMBED_STAGGER_DELAY` in [config.js](./config.js).

> [!TIP]
> **Issue 3: How to enable debug mode?**
> - Append `?debug=1` to the URL (e.g., `http://localhost:3000/?debug=1`).
> - Unmasked cross-origin warnings and scheduling diagnostics will display in the developer console (F12).

---

## Development & Deployment Guide

### Local Development Commands

Pure front-end static architecture with zero build steps:

```bash
# Clone repository
git clone https://github.com/Scorpio-meow/Threadfolio-Example.git
cd Threadfolio-Example
```

**Server Execution Options:**
1. **VS Code / IDE Live Server**: Right-click `index.html` and select "Open with Live Server".
2. **Bun Static Server**:
   ```bash
   bunx http-server -p 3000
   ```
3. **Python Static Server**:
   ```bash
   python -m http.server 3000
   ```

### Static Hosting Deployment

Deploy directly to any static web host:

- **GitHub Pages**: Set repository Settings -> Pages to deploy from root of `main` branch.
- **Cloudflare Pages**: Connect Git repository, leave Build command empty, set Output directory to `/`.
- **Vercel / Netlify**: Drag-and-drop folder or connect Git repository for instant deployment.

---

## Changelog

Follows the [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format.

### [Unreleased]

#### Added
- Support for the post `embeddable` field (written by the Threadkeep extension's check): posts with `embeddable: false` are still loaded, so viewers logged into Threads with third-party cookies allowed can see them; on failure the fallback card explains that Threads limits the post to logged-in viewers.
- Comprehensive Exponential Rate-Limiting Backoff defense against HTTP 429 limits.
- Live loading progress tracker showing percentage, elapsed time, and ETA.
- 32-bit djb2 content hash automated Service Worker cache invalidation.
- Single post isolated preview mode (`?post=`) and deterministic seeded shuffle (`?random=`).
- Footer FAQ accordion diagnostic modal.
- Queued post cards show a shimmering skeleton that fades out to the real post once Threads reports its height.
- Header shows the site name Threadfolio and the total post count; cards fade up as they scroll into view via a CSS scroll timeline.

#### Changed
- Refactored masonry layout to native CSS `columns` for smoother scrolling performance.
- Upgraded dual theme system with dynamic CSS custom properties; the theme is written into each `blockquote`'s `data-theme` at render time.
- Switched the interface to borderless lightness layering (light `#f5f5f3` / `#fcfcfb`, dark `#0b0b0a` to `#20201e`) while keeping the colored background and glows; every font weight drops by 40 in dark mode, and fonts load as variable fonts.
- Focused inputs get a double-ring glow; copy-link feedback pops in on a spring and fades back after 2.5 seconds; numbers use tabular figures and body text avoids orphans.
- The FAQ collapses with `grid-template-rows: 0fr → 1fr`, with no JavaScript and no `max-height` cap.
- Unified the site name as Threadfolio (tab title, iOS home screen, PWA); pagination Prev/Next became arrow icons with "上一頁 / 下一頁" accessible labels; the card top gradient strip shows only on hover or while loading.
- Enhanced `console-filter.js` interceptors to suppress 404 noise and cross-origin postMessage warnings.

#### Fixed
- Removed forced minimum height constraints for embedded posts on mobile viewports for cleaner presentation.
- Fixed abnormal bottom whitespace margin in single post isolated preview mode.
- Fixed large blank space below embeds shorter than 280px (e.g. the "Cannot display thread" notice, short text posts): the `postMessage` listener now accepts the plain numeric height Threads actually sends and pins the iframe's `min-height` to it.
- Fixed posts Threads refuses to embed (the embed endpoint returns 500 and the iframe shows Facebook's "Sorry, something went wrong") being counted as successful loads: success now requires Threads to post a height report greater than 0 via `postMessage`, and a missing report shows the "View on Threads" fallback card and counts the post as failed.
- Fixed the fallback card's specific failure reason (X-Frame, 500, embedding refused) being overwritten by the queue's generic "timed out" message.
- Corrected the FAQ: public posts need neither a Threads login nor Do Not Track turned off. It now lists the actual causes (embedding refused by Threads, ad/privacy blockers, timeouts) and quotes the fallback card's actual "貼文載入失敗 / 在 Threads 上查看" text. Login-only posts do display in browsers logged into Threads with third-party cookies allowed.
- Fixed the skeleton shimmer never showing: queued posts temporarily lack the `text-post-media` class, so the old rule never matched them.
- Fixed links and buttons inside a collapsed FAQ answer still being reachable with the Tab key.
- Fixed white-on-white text on the "開啟偵錯模式" button in dark mode, and white text on the light gradient of the selected tag.

#### Security
- Resolved CodeQL static analysis alerts for DOM XSS by sanitizing dynamic text nodes and using safe property assignments.
- Fixed XSS vulnerability in `threads-loader.js` caused by re-parsing DOM text nodes as raw HTML.

---

## AI-Friendly Documentation (llms.txt)

This project provides a standardized **[llms.txt](./llms.txt)** file designed for AI agents, LLM search engines, and RAG indexing pipelines:

- File location: `llms.txt`
- Contents: System architecture, runtime configuration schema, data models, and detailed algorithm specifications.

---

## License & Disclaimer

### License

This project is licensed under the **[MIT License](./LICENSE)**. You are free to use, modify, distribute, and integrate it into private or commercial projects.

### Disclaimer

This is an independent open-source project and is not affiliated with, authorized, or endorsed by Meta or Threads. Intellectual property rights for embedded content belong to their respective authors and Meta.

---

<div align="center">

**Threadfolio**  
Developed and maintained by [Scorpio-meow](https://github.com/Scorpio-meow)

</div>