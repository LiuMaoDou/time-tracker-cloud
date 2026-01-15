# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-page time tracking application built with vanilla HTML, CSS, and JavaScript. It uses Supabase for cloud synchronization and real-time updates across devices. The app is designed for personal productivity tracking with features for task timing, daily planning, todos, and insight capture.

**Title**: 任务时间记录系统 - Cloud Pro (Supabase 最终版)

## Architecture

### Single-File Structure

The entire application is contained in `index.html` (1918 lines). It includes:
- HTML structure
- Embedded CSS (lines 14-669)
- Core JavaScript logic (lines 884-1737)
- Supabase integration module (lines 1739-1916)

### Key Components

**Main Sections** (left-to-right layout):
1. **今日足迹 (Daily Records)** - Time tracking with start/pause/stop controls
2. **今日规划 (Daily Plans)** - Day-specific plans that can be converted to todos or started as tasks
3. **待办清单 (Todo List)** - Todos organized by project with deadline tracking
4. **灵感洞察 (Insights/Questions)** - Capture and develop ideas

### Data Model

All data is stored in `window` global variables:
- `window.records[]` - Completed task records with duration, timestamp, description
- `window.dailyPlans[]` - Daily plans (cleared on "今日结束")
- `window.todos[]` - Persistent todos with project, deadline, subtitle
- `window.questions[]` - Insights/questions with optional answers
- Timer state: `currentTask`, `startTime`, `pausedTime`, `isPaused`, `currentTaskDescription`
- Relationships: `currentPlanId`, `currentTodoId` track which plan/todo is being executed

### Cloud Synchronization

**Supabase Integration** (lines 1739-1916):
- Table: `time_tracker_cloud`
- Row ID: `my_daily_data` (single-row design)
- Real-time subscription updates UI when data changes from other devices
- Debounced saves (1000ms) prevent excessive sync during typing
- Loading overlay ensures data is loaded before user interaction

**Important**:
- `isDataLoaded` flag prevents saves before initial load completes
- `handleDataLoad()` carefully preserves input focus to avoid interrupting typing (line 1842)
- All dates are normalized to ISO format before saving

### Theme System

CSS variables-based theming with `data-theme="dark"` attribute:
- Light mode (default)
- Dark mode with muted colors and adapted shadows
- Theme persisted in `localStorage`

## Common Development Tasks

### Adding New Features

When adding features that modify data:
1. Update the relevant `window` array/variable
2. Call the corresponding `update*List()` function to refresh UI
3. Call `triggerSync()` or `debouncedSave()` to sync to Supabase

### Modifying UI Sections

Each section has its own:
- Modal for adding items (e.g., `addPlanModal`, `addTodoModal`)
- Update function (e.g., `updatePlanList()`, `updateTodoList()`)
- Add function (e.g., `addDailyPlan()`, `addTodo()`)

### Working with Todos

Todos support:
- Project grouping with collapsible headers
- Deadline-based sorting within groups
- Visual indicators for overdue items
- "开始执行" removes todo and starts timer

### Flomo Integration

Webhook URL hardcoded (line 1064, 1641): `https://flomoapp.com/iwh/MjMyNTE1/896a528a72e341a87584f81234f5d2eb/`

Used for:
- Daily review (`sendDailyReviewToFlomo`)
- Task records (`sendRecordToFlomo`)
- Insights/questions (`sendToFlomo`)

### Excel Export

Uses SheetJS (XLSX.js) loaded via CDN (line 13). Exports all records with:
- Task name
- Start/end times
- Duration
- Description

## Key Functions

**Timer Management**:
- `startTask()` - Starts timer, disables input
- `pauseTask()` - Toggle pause/resume
- `stopTask()` - Saves record, completes related plan/todo, resets state

**Data Synchronization**:
- `triggerSync()` - Immediate sync
- `debouncedSave()` - Debounced sync (use for frequent updates)
- `saveToCloud()` - Async Supabase upsert

**UI Updates**:
- `refreshAllUI()` - Refreshes all sections
- `updateRecordsList()`, `updateTodoList()`, `updateQuestionList()`, `updatePlanList()`

**Day Completion**:
- `finishToday()` - Exports Excel, clears records and plans (keeps todos and questions)

## Important Notes

- **No build process** - This is a static HTML file, no compilation needed
- **No package manager** - All dependencies loaded via CDN
- **Supabase credentials** - Hardcoded in index.html (lines 1742-1743)
- **Single-user design** - Uses fixed row ID for all data
- **Daily workflow** - Records and plans are meant to be cleared daily via "今日结束"
- **Git history** - Recent commits mainly update Flomo integration and UI tweaks

## Testing Locally

Simply open `index.html` in a browser. The app will:
1. Show loading overlay while fetching from Supabase
2. Load existing data or start fresh
3. Sync changes automatically with 1-second debounce

No local server required unless testing CORS-related changes.
