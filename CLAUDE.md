# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A pure client-side document preview tool implemented as a single HTML file (`index.html`). No build system, no dependencies to install — open `index.html` directly in a browser.

## Running the App

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8080
# then visit http://localhost:8080
```

There are no build steps, no linting, and no test suite.

## Architecture

The entire application lives in `index.html` as inline CSS + vanilla JavaScript. There is no framework, no module system, and no external files (all libraries are loaded from CDN).

### CDN Libraries

| Library | Version | Purpose |
|---|---|---|
| mammoth.js | 1.8.0 | `.docx` → HTML conversion |
| PDF.js | 3.11.174 | PDF rendering to `<canvas>` |
| SheetJS (xlsx) | 0.18.5 | `.xlsx`/`.xls` → HTML table |

### Core Data Flow

1. **Input**: file drag-and-drop, file picker, or URL fetch → `ArrayBuffer`
2. **Dispatch**: `getFileType()` detects extension → `renderByType()` routes to the correct renderer
3. **Render**:
   - `renderDocx()` — calls `mammoth.convertToHtml()`, injects result into `#docxContent`
   - `renderPdf()` — loads PDF via `pdfjsLib.getDocument()`, renders each page sequentially onto `<canvas>` elements in `#pdfContent`
   - `renderExcel()` — parses with `XLSX.read()`, builds sheet tab UI, renders active sheet via `XLSX.utils.sheet_to_html()` into `#excelContent`
4. **Post-render**: toolbar (zoom, scroll-to-top, print) becomes visible

### Key State Variables

```js
currentZoom       // integer percent (50–200), default 100
currentFileType   // "docx" | "pdf" | "excel" | null
pdfDoc            // pdfjsLib document object, kept for re-render on zoom
excelWorkbook     // XLSX workbook object, kept for sheet switching
```

Zoom for DOCX/Excel changes `fontSize` via CSS; zoom for PDF re-renders all canvas pages at the new scale.

### `code.html`

An older dark-themed variant that only supports `.docx` (uses mammoth 1.6.0). Not the primary file.
