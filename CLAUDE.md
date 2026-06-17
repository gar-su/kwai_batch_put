# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-file vanilla JS/HTML prototype for the Kwai batch ad campaign management system. No build system, no external dependencies, no tests. Open `index.html` directly in a browser.

## Architecture

Two modules, all in `index.html` (~2950 lines):

- **Module 1 — Batch Task List** (lines ~340-466, JS from ~2840): Main page. Dual-header table showing campaign/adgroup/creative submission status per batch task. Supports search/filter, retry (per-level), delete, detail edit.
- **Module 2 — Configuration Area** (lines ~468-1395, JS scattered): Full-screen drawer (`drawer-batch`) opened by "批量添加" or row "详情". Horizontal card layout with 5 config cards (Account / Campaign / Adgroup / Targeting / Creative), each opening a sub-drawer. Bottom preview table shows cartesian product of configured items, with submit buttons.

## Component System

- **Drawers**: `openDrawer(id)` / `closeDrawer(id)`. z-index stacks dynamically (base 2000, +2 per open). Esc key closes all open drawers.
- **Dialogs**: `openDialog(id)` / `closeDialog(id)`. z-index fixed at 5000. Overlay click-to-close: `if(event.target===this)closeDialog(id)`.
- **Radio button tabs**: `selectRadioBtn(input)` toggles `.is-active`, `getRadioVal(groupId)` reads checked value. Used for all config selections (no native `<select>` in config forms).
- **Toast**: `showMsg(msg, type)` — auto-dismiss 2s, types: success/warning/info.

## Key Global State

| Variable | Scope | Purpose |
|----------|-------|---------|
| `tableData` | Module 1 | Task list rows (taskId, accountName, planStatus, unitStatus, creativeStatus, etc.) |
| `accountListData` | Module 2 | Available Kwai accounts (id, name) |
| `targetingPackages` | Module 2 | Saved targeting packages with full config |
| `templateListData` | Module 2 | Saved config templates |
| `creativeAdgroups` | Module 2 | Dynamic adgroups with materials in creative drawer |
| `_campaignSaved` | Module 2 | Lock flag — adgroup/targeting/creative cards disabled until campaign saved |
| `_creativeSaved` | Module 2 | Track whether creative has been saved |
| `scheduleData` | Module 2 | 7×24 time schedule grid, `[[day1_hours], [day2_hours], ...]` |
| `drawerZIndex` | Global | Current drawer z-index base, starts at 2000 |

## Field Linkage Rules

- `onStrategyChange()` — Delivery strategy: Target Cost(2) hides CBO row; Lowest Cost(3)/Cost Cap(4) show CBO but lock budget type to "unlimited"
- `onMarketingTypeChange()` — App promotion(1) vs Website(2): toggles app selector vs landing page/Pixel fields
- `onOcpxChange()` — Conversion goal: ROAS types(123/130) show ROI% input; others show bid(USD) input
- `onCboChange()` — CBO: disabled → campaign budget type limited; linked → adgroup budget type limited
- `onUnitBudgetTypeChange()` — Adgroup budget: daily(2) shows single input; date(3) shows 7-day inputs; CBO=1 forces unlimited

## Important CSS Rules

- `[hidden]{display:none!important}` — overrides inline `display:flex` on hidden elements
- Element Plus CSS variables defined in `:root`, primary color `#1677ff`
- Radio button selected state: blue background, white text

## Security

- `escHtml(s)` — DOM-based XSS prevention for template data in HTML
- `escJs(s)` — Escape for template names embedded in JS strings

## No Build/Tooling

This is a pure HTML file. There are no linting, testing, or build commands. The global CLAUDE.md Python tooling (mypy, ruff, etc.) does not apply to this project.
