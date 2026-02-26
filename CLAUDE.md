# Canvas Grade Mapper

## Project Overview

A single-page web application for university instructors to manage Canvas LMS grades. Built as a static HTML file with no build system or framework. The UI is in Thai language.

### Two Modes

1. **Score Mapping (Map คะแนน)** — Import external score files and map them to Canvas assignments by matching students via Email (priority) or SIS User ID. Outputs a CSV ready for Canvas Grades > Import.
2. **Student Status Check (ตรวจสอบสถานะนักศึกษา)** — Compare Canvas enrollment against registrar CSV files to find mismatches (students in Canvas but not registered, or registered but missing from Canvas).

## Architecture

- **Single-file app**: Everything lives in `index.html` — HTML, CSS (inline `<style>`), and JavaScript (inline `<script>`).
- **No build system**: No package.json, no bundler, no framework. Open the file directly in a browser.
- **External dependencies** (loaded via CDN):
  - [SheetJS (xlsx)](https://cdn.sheetjs.com/xlsx-0.20.1/package/dist/xlsx.full.min.js) — for parsing Excel files
  - [Google Fonts - Sarabun](https://fonts.googleapis.com/css2?family=Sarabun) — Thai-friendly font

## Key Patterns

- **Step wizard UI**: Both modes use numbered step navigation (step dots + card show/hide via `.hidden` class).
- **State management**: Plain global variables (`canvasData`, `scoreData`, `mappingResult`, `chkResults`, etc.).
- **File parsing**: The `parseFile()` function handles both CSV (custom parser with quote handling) and Excel (via SheetJS).
- **Canvas file validation**: Checks that the first column header is "Student" and second/third columns match Canvas patterns (ID, SIS).
- **Student matching**: Email match takes priority over ID match when both are available.
- **Registrar filename convention**: `{courseCode:6}{lecSection:3}{labSection:3}.csv` (e.g., `261111802000.csv`).
- **CSS**: Dark gradient theme with glassmorphism cards, teal accent color (#4fd1c5), responsive with mobile breakpoint at 600px.

## File Structure

```
index.html          # The entire application
test-data/          # Sample data files for testing
  canvas_export.csv   # Sample Canvas gradebook export
  261111802000.csv    # Sample registrar file (Lec 802, Lab 000)
  261111803000.csv    # Sample registrar file (Lec 803, Lab 000)
```

## Data Formats

### Canvas Export CSV
```
Student, ID, SIS User ID, SIS Login ID, Integration ID, Section, [Assignment columns...]
```
- Row after headers may be a "Points Possible" row (starts with lowercase "point") — this is skipped during processing.
- Assignments are identified from column index 6 onward, filtered by having a numeric ID in parentheses like `(123456)`.

### Registrar CSV
```
ID, NAME, SNAME, Column1_1, FACID, FACNAME
```

### Score File (user-provided)
Must contain at least one of: `ID`, `Student ID`, `SIS User ID`, or an email-like column.

## Development Notes

- To test: open `index.html` in a browser and use files from `test-data/`.
- All CSV exports include a UTF-8 BOM (`\uFEFF`) for proper Thai character display in Excel.
- The app is fully client-side — no server, no API calls, no data leaves the browser.
