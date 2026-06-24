---
name: googlesheets
description: >-
  How to use the Google Sheets connector's tools — reading and writing cell
  values by A1 range, appending rows, clearing data, and creating spreadsheets
  or tabs. Use when the user works with Google Sheets, a spreadsheet, tabs/
  worksheets, cells, rows, columns, or wants to read/append/update tabular data.
---

You are using the **Google Sheets** connector (via Composio). Its tools act on the
connected user's Google Sheets. Apply this guidance whenever you touch a sheet.

## Identify the spreadsheet and its tabs first

- Almost every tool needs a **spreadsheet id** — the long token in the sheet URL
  (`docs.google.com/spreadsheets/d/<ID>/edit`), *not* the title. Extract it from
  a pasted URL rather than guessing.
- A spreadsheet contains one or more **sheets** (tabs/worksheets), each with its own
  name. Before reading or writing a specific tab, list the tabs (`GET_SHEET_NAMES`)
  or pull structure/metadata (`GET_SPREADSHEET_INFO`) so you reference a real tab
  name and know its dimensions. A wrong tab name errors.

## Ranges use A1 notation

- Ranges are **A1 notation**, qualified by tab: `Sheet1!A1:C10`, `Sheet1!A:A`
  (whole column), `Sheet1!2:2` (whole row). An unqualified range hits the first tab.
- Read with `BATCH_GET` — it takes **multiple ranges** at once, so fetch everything
  you need in one call instead of looping. Values come back as a grid of rows.

## Writing: update vs. append

- **Update** (`VALUES_UPDATE`) overwrites the cells in the given range in place —
  the values you send map onto the range top-left to bottom-right. Use it when you
  know exactly where the data goes.
- **Append** (`SPREADSHEETS_VALUES_APPEND`) adds rows **after the last row of the
  existing table** near the given range — use it to log/add rows without computing
  the next empty row yourself. Don't use update for "add a new row"; you'll clobber.
- **`valueInputOption`** controls parsing: `USER_ENTERED` interprets input as if
  typed in the UI (so `=SUM(...)` becomes a formula, `5/1` may become a date),
  while `RAW` stores strings verbatim. Default to `USER_ENTERED` for human-style
  data and formulas; use `RAW` when you must preserve exact text (ids, codes).
- Send values as **rows of cells** (a list of lists). A flat list is one row.
- Prefer `VALUES_UPDATE` over `BATCH_UPDATE` — the latter is **deprecated** here and
  also writes only a single range despite the name.

## Creating structure

- `CREATE_GOOGLE_SHEET1` creates a **new spreadsheet** (returns a new id); `ADD_SHEET`
  adds a **tab** to an existing spreadsheet. Pick based on whether the container
  already exists — don't create a whole new file when the user wanted a new tab.

## Care

- Writes are **immediate and shared** with everyone who has access, and there is no
  undo through the API. Confirm the target tab and range before writing; never
  overwrite a populated range you haven't read.
- `CLEAR_VALUES` removes cell **content** (formatting and notes survive) over the
  given range — it is destructive to data. Scope the range tightly; never clear a
  whole tab without explicit confirmation.
- Read before you overwrite: pull the current values so you can preserve headers and
  existing rows, and so you can report what changed.
- Large sheets: request only the ranges you need and summarize rather than dumping
  every cell back to the user.
