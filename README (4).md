# Form Filler — Word Form Extractor & Generator

A single-file, zero-backend web tool that reads a blank Word (`.docx`) questionnaire, turns every question into a clean on-screen input field, and writes your answers back into the original document — exporting a completed `.docx` with the layout and formatting intact.

Everything runs locally in your browser. No server, no uploads, no build step, no install.

![Made with Vanilla JS](https://img.shields.io/badge/made%20with-vanilla%20JS-f7df1e?logo=javascript&logoColor=black)
![No build step](https://img.shields.io/badge/build-none-brightgreen)
![Client-side only](https://img.shields.io/badge/runs-100%25%20client--side-blue)

---

## Why

Filling out table-based Word forms (compliance questionnaires, vendor onboarding forms, KYC requests) usually means scrolling through a document, clicking into cramped table cells, and fighting Word's formatting. This tool flips that: it extracts the questions into a focused, distraction-free form UI, then injects your answers back into the exact cells they belong in.

## Features

- **Drag & drop a `.docx`** — the tool unzips it in the browser and parses `word/document.xml` directly
- **Automatic question detection** — every table row after the header becomes a question card, with the last column treated as the answer cell
- **Document details section** — `Label:` / value rows above the header (e.g. *Company Name:*, *Date:*) are detected and made editable too
- **Smart input types** — questions whose guidance starts with `Yes/No` get a dropdown; detail fields get a text input; everything else gets an auto-growing textarea
- **Preset answers** — pin standard answers to specific question numbers so they load pre-filled (and stay editable), marked with a `PRESET` badge
- **Progress tracking** — a sticky bottom bar shows how many questions are answered, with the export button enabled once you've started
- **Format-preserving export** — answers are written into the original XML, keeping the cell's paragraph styling; multi-line answers become proper Word line breaks
- **Accessible & responsive** — keyboard-operable dropzone, mobile layout, `prefers-reduced-motion` support

## Getting started

### Option 1 — just open it

1. Download `index.html` (or clone the repo)
2. Double-click it — it opens in any modern browser
3. Drop in your blank `.docx` form, answer the questions, click **Generate filled Word file**

> The only external dependency is [JSZip](https://stuk.github.io/jszip/) loaded from cdnjs, so you need an internet connection on first load. Vendor the script locally if you want fully offline use.

### Option 2 — GitHub Pages

1. Name the file `index.html` in the repo root
2. Enable **Settings → Pages → Deploy from branch**
3. Share the URL — since nothing ever leaves the browser, it's safe to host publicly

## Expected form layout

The tool is built for forms structured as a Word **table**, in this shape:

| | |
|---|---|
| Company Name: | *(value cell)* |
| Date: | *(value cell)* |
| **Question / Request** | **Company Comments** |
| 1- First question text | *(answer cell)* |
| 2- Second question · Yes/No | *(answer cell)* |

Detection rules:

- **Header row** — the first row containing “Company Comments”, or containing both “Question” and “Request”, marks the start of the Q&A section
- **Detail fields** — rows *above* the header where a cell ends in `:` (max 40 chars) become editable document details, with the cell to the right as the value
- **Questions** — rows *below* the header with 2+ cells; the first cell is the question, middle cells become guidance text, and the **last cell** is where the answer is written
- **Question numbers** — a leading `1-`, `2.` or `3)` in the question text becomes the `Q1` / `Q2` chip
- Single-cell rows (full-width notes, spacers) are skipped automatically

If no questions are found, the tool explains the expected structure instead of failing silently.

## Preset answers

Answers that are always the same for a given form can be hardcoded so they load pre-filled. Edit the `PRESET_ANSWERS` object near the top of the `<script>` block — keys are question numbers, `\n` creates line breaks:

```js
const PRESET_ANSWERS = {
  3: "ACME EXPORTS PVT LTD\nIEC : 1234567",
  7: "PHARMACEUTICALS"
};
```

Preset values are still fully editable in the UI before export.

## How it works

1. **Read** — [`JSZip`](https://stuk.github.io/jszip/) opens the `.docx` (which is just a ZIP), and `DOMParser` parses `word/document.xml` with proper WordprocessingML namespaces
2. **Extract** — table rows (`w:tr`) are scanned; question text, guidance, and answer cells are mapped into field objects that keep live references to their XML nodes
3. **Fill** — the form UI binds each input to its field; progress updates as you type
4. **Write back** — for each answer cell, the first paragraph (and its formatting properties) is kept, existing runs are replaced with a single new run, and newlines become `<w:br/>` elements with `xml:space="preserve"`
5. **Export** — the modified XML is re-serialized into the ZIP and downloaded as `«original name» - filled.docx`

## Privacy

Your document never leaves your machine. There is no backend, no analytics, and no network request other than fetching the JSZip library from a CDN. All parsing, editing, and file generation happens in the browser via the File, DOMParser, and Blob APIs.

## Limitations

- **Table-based forms only** — free-flowing paragraph questionnaires, content controls, and legacy form fields aren't detected
- **`.docx` only** — `.doc` and PDF forms are rejected with a friendly error (convert them to `.docx` first)
- **Answer cells are rewritten as plain text** — the paragraph style survives, but any rich formatting inside the placeholder answer text is replaced
- **Header keywords are English** — detection looks for “Company Comments” / “Question” + “Request”; tweak the regex in `buildFields()` for other templates or languages

## Tech stack

- Vanilla HTML / CSS / JavaScript — no framework, no bundler, one file
- [JSZip 3.10.1](https://stuk.github.io/jszip/) for reading and rebuilding the `.docx` container
- Native `DOMParser` / `XMLSerializer` for WordprocessingML manipulation

## Contributing

Issues and PRs welcome. Useful directions: support for content-control (`w:sdt`) forms, configurable header detection, multi-table documents, and saving in-progress answers to a JSON file.

## License

MIT — see [LICENSE](LICENSE). (Add a license file if you haven't yet, or swap this line for your preferred license.)
