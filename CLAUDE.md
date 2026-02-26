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

## Code Organization

The `<script>` section is organized into labeled sections:

```
// ========== CONSTANTS ==========       — Magic values, status enums, regex patterns
// ========== STATE ==========           — Centralized `state` object + MODE_REGISTRY
// ========== UTILITIES ==========       — Shared functions (parseFile, escapeCSV, downloadCSV, etc.)
// ========== MODE MANAGEMENT ==========  — switchMode(), updateMapSteps(), updateChkSteps()
// ========== SCORE MAPPING: Data =====  — findCanvasColumnIndices(), performStudentMatching(), etc.
// ========== SCORE MAPPING: Render ==== — renderMappingResults()
// ========== SCORE MAPPING: Export ==== — exportMappingCSV()
// ========== SCORE MAPPING: Reset ===== — resetMapMode()
// ========== STATUS CHECK: Data =======  — extractCanvasStudents(), compareSection(), etc.
// ========== STATUS CHECK: Render =====  — renderCheckResults(), renderSectionTable(), etc.
// ========== STATUS CHECK: Export =====  — exportCheckCSV()
// ========== STATUS CHECK: Reset ======  — resetCheckMode()
// ========== EVENT BINDING ============  — All addEventListener calls grouped here
// ========== INITIALIZATION ===========  — updateMapSteps()
```

### State Management

All mode state is centralized in a single `state` object:

```javascript
const state = {
    activeMode: 'map',
    map: { canvasData, scoreData, assignments, mappingResult, currentStep, mappingMode },
    check: { canvasData, registrarFiles, currentStep, results, currentFilter }
};
```

### Mode Registry (for extensibility)

```javascript
const MODE_REGISTRY = {
    map:   { containerId, btnId, totalSteps, dotPrefix },
    check: { containerId, btnId, totalSteps, dotPrefix }
};
```

To add a new mode: add an entry to `MODE_REGISTRY`, add `state.newMode`, add HTML container with `{prefix}Dot{N}` / `{prefix}Step{N}` IDs.

### HTML ID Convention

Step dots and cards follow the pattern `{prefix}Dot{N}` and `{prefix}Step{N}`:
- Map mode: `mapDot1`–`mapDot4`, `mapStep1`–`mapStep4`
- Check mode: `chkDot1`–`chkDot3`, `chkStep1`–`chkStep3`

## Key Patterns

- **Step wizard UI**: Both modes use `updateStepIndicators(prefix, totalSteps, currentStep)` for shared step navigation.
- **Student matching**: Uses `Map`-based O(n+m) lookups (email priority > ID) via `buildScoreLookup()` + `performStudentMatching()`.
- **Canvas file validation**: Shared `validateCanvasFile(data)` function checks column structure.
- **CSV export**: Shared `escapeCSV()` and `downloadCSV()` utilities, all exports include UTF-8 BOM.
- **Registrar filename convention**: `{courseCode:6}{lecSection:3}{labSection:3}.csv` — parsed by `parseRegFilename()`.
- **Event handlers**: All use `addEventListener` (no inline `onclick` in HTML).
- **CSS**: Uses CSS custom properties (`--color-accent`, `--color-success`, etc.) defined in `:root`. Dark gradient theme with glassmorphism cards, responsive at 600px breakpoint.
- **Error handling**: Uses `showToast()` for transient errors and `showInlineError()` for persistent form errors.

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
- Row after headers may be a "Points Possible" row (starts with lowercase "point") — this is skipped via `getPointsRowStart()`.
- Assignments are identified from column index `CANVAS_FIXED_COLS` (6) onward, filtered by `ASSIGNMENT_ID_REGEX`.

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
- Constants are defined at the top of `<script>` (`CANVAS_FIXED_COLS`, `ASSIGNMENT_ID_REGEX`, `EXCLUDE_PATTERNS`, `STATUS`, etc.) — update these when Canvas format changes.
