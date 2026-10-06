# Libris

A professional desktop reader for EPUB and PDF files. Built with Electron, React, and TypeScript.

## Features

- **Library** — import books via a native file picker or drag-and-drop, then browse them in a
  responsive cover grid with search and sorting (title, author, recently added, recently read).
- **Reading** — open EPUBs with [epubjs](https://github.com/futurepress/epub.js) (paginated
  rendering, chapter navigation) and PDFs with [pdfjs-dist](https://mozilla.github.io/pdf.js/)
  (continuous scroll, fit-to-width, page navigation).
- **Themes** — dark and light, toggled from the toolbar.
- **DRM-aware** — EPUBs protected by DRM are detected and refused with a clear message; only
  font-obfuscation encryption is permitted.

## Getting started

```bash
npm install
npm run dev
```

## Scripts

| Script            | Description                                        |
| ----------------- | -------------------------------------------------- |
| `npm run dev`     | Start the app in development mode (Vite HMR)       |
| `npm run build`   | Build the main, preload, and renderer bundles      |
| `npm run typecheck` | Type-check the Node and web TypeScript projects  |
| `npm run start`   | Run the built app (no packaging)                   |
| `npm run dist`    | Build and package a Windows installer (NSIS)       |

## Architecture

Three processes, kept strictly separated:

- **Main** (`src/main`) — window lifecycle and all privileged work: native dialogs, reading file
  bytes, and reading/writing the library JSON store.
- **Preload** (`src/preload`) — a typed, minimal `window.api` bridge over IPC. No Node exposed
  to the renderer.
- **Renderer** (`src/renderer`) — React UI. Parsing and rendering (epubjs, pdfjs) run here,
  where the DOM and canvas are available.

Security posture: `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`, and a
Content-Security-Policy applied as a production-only response header (dev needs relaxed rules
for Vite HMR).

### Persistence

The library is stored as JSON at `app.getPath('userData')/library.json`, written atomically
(write to a temp file, then rename). Covers are stored inline as resized data URLs. Books are
referenced **by path** rather than copied; if a file moves, its card shows a "file missing"
state.

## Project layout

```
src/
  main/       Electron main process (window, IPC handlers, library store)
  preload/    contextBridge API + type declarations
  renderer/   React application
  shared/     Types shared across processes
test-fixtures/  Sample EPUBs: valid, malformed, and DRM-protected
```

## Roadmap

Feature work is incremental. Milestone 1 (this one) is the library plus basic reading. Planned
next: reading polish (font controls, bookmarks, resume position), annotations/highlights, and
two-page/column layout.
