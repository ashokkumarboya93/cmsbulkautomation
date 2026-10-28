# MCN CMS Tool

A browser-based utility that turns a folder of ordinary images into a ZIP package structured for the **Salesforce Marketing Cloud Next CMS** bulk image import workflow.

Select your images, review them, and download a ready-made ZIP. No manual folder creation, no hand-written JSON.

> **Privacy:** everything runs locally in your browser. Images, filenames and metadata are never uploaded anywhere. There is no backend, database, login or API key.

---

## Table of contents

1. [Features](#features)
2. [Quick start](#quick-start)
3. [How to use](#how-to-use)
4. [Output format](#output-format)
5. [Naming rules](#naming-rules)
6. [Supported formats](#supported-formats)
7. [Error handling and warnings](#error-handling-and-warnings)
8. [Project structure](#project-structure)
9. [Deployment](#deployment)
10. [Browser support and limitations](#browser-support-and-limitations)
11. [Troubleshooting](#troubleshooting)
12. [Verification status](#verification-status)
13. [Developer](#developer)

---

## Features

- **Multiple input methods:** select several image files, select a whole folder, or drag and drop files or folders (nested folders are read too).
- **Add more at any time:** new selections are appended to the existing list, and exact duplicates are ignored.
- **Live preview list:** thumbnail, filename, format, file size, pixel dimensions and relative folder path for each image.
- **Validation:** unsupported files are skipped with a message, unreadable images are flagged and excluded from the export, and duplicate filenames are highlighted.
- **Per-item control:** remove individual images or clear the entire list.
- **Automatic package generation:** numbered folders, `content.json` metadata and `_media` directories are created for you.
- **Original files preserved:** image bytes are never resized, recompressed or converted.
- **Progress indicator** during packaging, plus a summary (file name, entry count, ZIP size) when finished.
- **Interface:** white and sky-blue Salesforce-style theme with an optional dark mode, inline icons, a responsive layout and a built-in **How it works** guide.

## Quick start

The tool is a single static file: `mc-next-bulk-image-import.html`.

**Option A: open directly**

1. Download `mc-next-bulk-image-import.html`.
2. Double-click it to open it in a modern browser.

**Option B: serve locally**

```bash
# from the folder containing the file
python3 -m http.server 8080
# then open http://localhost:8080/mc-next-bulk-image-import.html
```

**Requirement:** an internet connection the first time the page loads, because [JSZip 3.10.1](https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js) is loaded from cdnjs. To work fully offline, download that script, save it next to the HTML file, and change the `<script src=...>` tag to point to the local copy.

## How to use

1. **Select images.** Click **Select images** or **Select folder**, or drag files or folders onto the drop area.
2. **Review the list.** Check thumbnails, sizes and dimensions. Remove anything you do not want.
3. **Choose title style.** The *Capitalize first letter of title* checkbox is on by default (see [Naming rules](#naming-rules)).
4. **Generate.** Click **Generate & download ZIP**. The browser downloads `MCN_CMS_Import_YYYY-MM-DD.zip`.
5. **Import.** Use the ZIP in your Salesforce CMS import workflow for Marketing Cloud Next. This tool prepares the package only; it does not connect to Salesforce.

The **How it works** button in the header opens the same flow inside the app.

## Output format

For every readable image, one entry folder is created:

```text
MCN_CMS_Import_2026-10-28.zip
├── 001_Bigbasketseeklogo/
│   ├── content.json
│   └── _media/
│       └── bigbasketseeklogo.png
├── 002_Download117/
│   ├── content.json
│   └── _media/
│       └── download117.jpg
└── 003_Epiclion/
    ├── content.json
    └── _media/
        └── epiclion.jpg
```

### `content.json`

```json
{
  "type": "sfdc_cms__image",
  "title": "Bigbasketseeklogo",
  "contentBody": {
    "sfdc_cms:media": {
      "source": {
        "type": "file",
        "mimeType": "image/png"
      }
    }
  }
}
```

Only `title` and `mimeType` vary per image. No other fields or files (for example a `_meta.json`) are added.

## Naming rules

| Item | Rule |
|---|---|
| **Entry folder** | `<sequence>_<title>`. The sequence starts at `001` and widens automatically (for example `0001`) when a batch has more than 999 images. |
| **Title** | The filename without its extension. By default the first letter is capitalized (`download117.jpg` → `Download117`). Untick the checkbox to keep the filename's original capitalization. |
| **Folder-name cleanup** | Characters invalid in paths (`\ / : * ? " < > |` and control characters) are removed, whitespace becomes `_`, leading and trailing dots are stripped, and names are capped at 80 characters. Unicode letters are kept. |
| **Duplicate folder names** | The sequence number already makes folders unique. If a collision still occurs, a numeric suffix is appended. |
| **Media file** | The original filename (with its original extension) is kept inside `_media/`. Unsafe path characters are replaced with `_`. |
| **ZIP name** | `MCN_CMS_Import_YYYY-MM-DD.zip`, using the current local date. |

The `title` inside `content.json` is the unsanitized title; only the folder name is cleaned up.

## Supported formats

| Extension | MIME type written to JSON |
|---|---|
| `.jpg`, `.jpeg` | `image/jpeg` |
| `.png` | `image/png` |
| `.gif` | `image/gif` |
| `.webp` | `image/webp` |

Extensions are matched case-insensitively (`.PNG` works). The MIME type is derived from the extension. Files with any other extension are skipped and reported.

## Error handling and warnings

| Situation | Behavior |
|---|---|
| Unsupported file type | Skipped; a message lists the first few skipped names. |
| Image that cannot be decoded | Tagged *Unreadable*; excluded from the ZIP; the export summary reports how many were skipped. |
| Same file selected twice | Ignored (matched by relative path, size and modified time). |
| Same filename from different folders | Both kept; each is tagged as a duplicate name and placed in its own numbered folder, so nothing is overwritten. |
| Nothing selected | Export is disabled; no empty ZIP is produced. |
| ZIP library not loaded | A clear message asks you to check your connection and reload. |
| Packaging failure | An error message appears and the interface stays usable so you can retry, ideally with fewer images. |

## Project structure

```text
mc-next-bulk-image-import.html   # entire app: markup, styles, scripts
README.md                        # this document
```

Inside the HTML file:

- **Styles:** CSS variables control the light and dark themes.
- **Icons:** an inline SVG sprite; no icon font or image files.
- **Logic:** plain JavaScript (no framework or build step) covering file intake, drag and drop, preview rendering, JSON generation, ZIP packaging and the how-it-works dialog.
- **Dependency:** JSZip 3.10.1 (MIT) via cdnjs.

## Deployment

Because the app is one static file, it can be hosted anywhere that serves static content: GitHub Pages, Netlify, Vercel, Cloudflare Pages, an S3 bucket or an internal web server. Upload `mc-next-bulk-image-import.html` (optionally renamed to `index.html`) and no server configuration is needed.

## Browser support and limitations

- Works in current versions of Chrome, Edge, Firefox and Safari.
- **Folder selection** relies on the `webkitdirectory` attribute (widely supported). **Folder drag and drop** relies on `webkitGetAsEntry`; if your browser lacks it, use the **Select folder** button.
- Large batches are limited by browser memory because the ZIP is assembled in memory. For hundreds of large images, export in several smaller batches.
- Image dimensions and thumbnails are generated by the browser; very large or unusual images may take a moment to appear.
- Only the formats in the table above are accepted, even if your browser can display others.

## Troubleshooting

| Problem | Fix |
|---|---|
| Export button is disabled | Add at least one readable image. |
| "ZIP library failed to load" | Check your internet connection or ad blocker and reload, or host JSZip locally (see [Quick start](#quick-start)). |
| Folder drop does nothing | Use **Select folder** instead. |
| An image is tagged *Unreadable* | The file is corrupt or has an extension that does not match its content. Re-export it from an image editor. |
| Download did not start | Allow downloads for the site in your browser, then click **Generate & download ZIP** again. |
| Titles look different from filenames | The first-letter capitalization option is on by default; untick it to keep the original case. |

## Verification status

- The `content.json` schema, the `NNN_Title/_media/` layout and the title-capitalization behavior are modeled on a sample package (`MCN_CMS_Import_2026-10-28.zip`) provided by the project owner.
- The app has **not** been tested against a live Salesforce org, so the final import behavior is unconfirmed. Try a small batch of 2 or 3 images in your org before running a large import.
- Rules for edge cases the sample does not show (unusual characters, more than 999 images, duplicate names) are implementation choices documented above.

## Developer

**Ashok Kumar**

- Email: [ashokkumarboya93@gmail.com](mailto:ashokkumarboya93@gmail.com)
- LinkedIn: [linkedin.com/in/ashokkumarboya](https://www.linkedin.com/in/ashokkumarboya/)
- GitHub: [github.com/ashokkumarboya93](https://github.com/ashokkumarboya93)

*Salesforce, Marketing Cloud and related marks are trademarks of Salesforce, Inc. This is an independent tool and is not affiliated with or endorsed by Salesforce.*
