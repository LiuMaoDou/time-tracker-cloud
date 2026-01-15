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

## Code Standards and Conventions

### **MANDATORY: All code must include comments**

When writing or modifying code, you **MUST** add comprehensive comments. This is a strict requirement.

### Comment Guidelines

#### 1. Function-Level Comments
Every new or modified function must have a comment block explaining:
- **Purpose**: What the function does
- **Problem**: What issue it solves (if fixing a bug)
- **Solution**: How it solves the problem
- **Parameters**: What inputs it expects (if complex)

**Example**:
```javascript
// 修复：待办勾选后的同步问题
// 问题：勾选后页面消失，但刷新后任务又回来了（删除操作未成功保存到云端）
// 原因：triggerSync() 是异步的，但没有等待完成就执行了删除，导致删除状态未同步
// 解决：改为 async/await 模式，确保每一步都同步完成后再执行下一步
async function toggleTodo(id) {
    // Function body...
}
```

#### 2. Step-by-Step Comments
For multi-step logic, use numbered comments to show execution flow:

```javascript
async function toggleTodo(id) {
    const t = window.todos.find(x => x.id === id);
    if(t) {
        // 1. 先标记为已完成
        t.completed = !t.completed;
        updateTodoList();

        // 2. 等待完成状态保存到云端（关键修复点）
        await triggerSync();

        // 3. 如果是勾选完成，500ms 后执行删除操作
        if(t.completed) {
            setTimeout(async () => {
                // 从数组中移除已完成的任务
                window.todos = window.todos.filter(x => x.id !== id);
                updateTodoList();

                // 等待删除操作保存到云端（关键修复点）
                await triggerSync();
            }, 500);
        }
    }
}
```

#### 3. Inline Comments for Critical Lines
Mark important logic with inline comments:

```javascript
await window.saveToCloud(); // 等待云端保存完成
if (descInput && document.activeElement !== descInput) { // 防止打字时被打断
    descInput.value = window.currentTaskDescription;
}
```

#### 4. Section Comments for CSS
Group related CSS rules with section headers:

```css
/* === 深色模式 === */
[data-theme="dark"] {
    --primary-gradient: #1A202C;
    /* ... */
}

/* === 今日结束按钮样式 === */
.btn-finish {
    background: var(--btn-finish-bg);
    /* ... */
}
```

### JavaScript Style Guidelines

1. **Variable Naming**:
   - Use `camelCase` for variables and functions
   - Use descriptive names: `currentTaskDescription` not `desc`
   - Boolean variables should start with `is`, `has`, `can`: `isDataLoaded`, `isPaused`

2. **Async/Await**:
   - Always use `async/await` for Supabase operations
   - Always `await` `triggerSync()` when data must be saved before continuing
   - Never fire-and-forget async operations that affect data integrity

3. **Error Handling**:
   - Use try-catch for Supabase operations
   - Show user-friendly alerts for errors
   - Log errors to console for debugging

4. **Data Mutations**:
   - Always call the corresponding `update*List()` after modifying data arrays
   - Always call `triggerSync()` after data changes
   - Use `debouncedSave()` for frequent updates (e.g., typing)

### CSS Style Guidelines

1. **CSS Variables**:
   - Use existing CSS variables for colors: `var(--primary-color)`, `var(--text-main)`
   - Define both light and dark mode values when adding new variables

2. **Naming**:
   - Use kebab-case: `.btn-add-icon`, `.todo-group-title`
   - Use descriptive class names that reflect purpose

3. **Responsiveness**:
   - Test changes on mobile (media query at `@media (max-width: 768px)`)

### Supabase Sync Guidelines

**Critical Rules**:
1. Always `await` async sync operations when order matters
2. Never start a new operation before the previous sync completes (if they're related)
3. Use `isDataLoaded` flag to prevent saves before initial load
4. Normalize all dates to ISO format before saving: `new Date().toISOString()`

**Example of Correct Sync Pattern**:
```javascript
// ✅ CORRECT: Wait for each sync to complete
async function deleteItem(id) {
    // 1. Mark as deleted
    item.deleted = true;
    await triggerSync(); // Wait for mark to save

    // 2. Actually remove from array
    items = items.filter(x => x.id !== id);
    await triggerSync(); // Wait for deletion to save
}

// ❌ INCORRECT: Fire-and-forget causes race conditions
function deleteItem(id) {
    item.deleted = true;
    triggerSync(); // Doesn't wait!
    items = items.filter(x => x.id !== id);
    triggerSync(); // Might save before first sync completes!
}
```

### Comment Language

- Use **Chinese** for comments (matching the UI language)
- Use English for technical terms: `async/await`, `Supabase`, `debounce`
- Be clear and concise

### When to Add Extra Comments

Add detailed comments when:
- Fixing a bug (explain the bug, cause, and fix)
- Implementing complex logic (multi-step processes)
- Working with async operations (explain the order and why)
- Adding workarounds (explain why the workaround is needed)
- Modifying data sync logic (explain data flow)

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
