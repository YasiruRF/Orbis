# ORBIS IDE — Architectural Implementation Plan & Task Tracker

**Document Status**: Active Working Roadmap  
**Companion Documents**: [architecture.md](file:///d:/Praxis/Orbis/Orbis/docs/architecture.md) | [optimisation.md](file:///d:/Praxis/Orbis/Orbis/docs/optimisation.md)

---

## Progress Overview

| Phase | Milestone | Target Window | Status | Completion |
| :--- | :--- | :--- | :--- | :--- |
| **Phase 0** | Baseline Preparation & Hygiene | Day 0 | Complete | 100% |
| **Phase 1** | Shell Consolidation & 26px Micro-HUD (UI) | Days 1–7 | Pending | 0% |
| **Phase 2** | Futuristic Hovers & Notification Quarantine (UI) | Days 8–14 | Pending | 0% |
| **Phase 3** | Scoped Editor Terminals & Diff Staging (UI) | Days 15–21 | Pending | 0% |
| **Phase 4** | Core Optimization & De-bloating | Days 22–28 | Pending | 0% |
| **Phase 5** | Two-Tier Local Intelligence Arbiter | Days 29–35 | Pending | 0% |
| **Phase 6** | Hardening, Benchmarking & Acceptance Sign-off | Days 36–38 | Pending | 0% |

---

## Phase 0: Baseline Preparation & Hygiene

- [x] **0.1. Upstream Snapshot & Core Decoupling**
  - [x] 0.1.1 Establish clean Code OSS baseline.
  - [x] 0.1.2 Author Praxis Dark theme strictly adhering to Praxis palette (Ink, Surface, Raise, Off-white, Mute, 2% Crimson accent).
  - [x] 0.1.3 Brand application as **Orbis** (`nameShort`, `nameLong`, `applicationName`, `win32MutexName`).
  - [x] 0.1.4 Resolve MSBuild Spectre mitigation and `DelayImp.lib` linker resolution.
  - [x] 0.1.5 Decouple Copilot dependencies and restore upstream test suites.
  - [x] 0.1.6 Fix ESLint configuration for dynamic optional extension imports.
  - [x] 0.1.7 Configure Open VSX extension gallery.
  - [x] 0.1.8 Author Praxis Material Icon Theme with dual-tone (#ECECE9 Off-White / #C1121F Crimson) language icons.
  - [x] 0.1.9 Configure tab stacking working set with 10-tab LRU memory pool (`workbench.editor.limit`).


---

## Phase 1: Shell Consolidation & 26px Micro-HUD (Days 1–7)

### Goal:
Strip all peripheral chrome (Titlebar, Menu Bar, Activity Bar, Status Bar) to achieve >96% canvas utilization and deploy the unified 26px Micro-HUD.

### Tasks & Sub-Tasks:

- [ ] **1.1 Peripheral Chrome Deactivation**
  - [ ] 1.1.1 Hide or disable default Activity Bar contribution (`src/vs/workbench/browser/parts/activitybar/activitybarPart.ts`).
  - [ ] 1.1.2 Suppress default bottom Status Bar rendering (`src/vs/workbench/browser/parts/statusbar/statusbarPart.ts`).
  - [ ] 1.1.3 Remove classic Menu Bar and eliminate Titlebar chrome padding (`src/vs/workbench/browser/parts/titlebar/titlebarPart.ts`).
  - [ ] 1.1.4 Enforce borderless layout calculation in workbench layout service (`src/vs/workbench/services/layout/browser/layoutService.ts`).

- [ ] **1.2 Construct Unified 26px Micro-HUD Header**
  - [ ] 1.2.1 Create custom Micro-HUD Part (`src/vs/orbis/workbench/browser/microHud/microHudPart.ts` or integrated into layout).
  - [ ] 1.2.2 Lock header container height strictly to `26px` with zero vertical margin.
  - [ ] 1.2.3 Implement **Left Zone**:
    - [ ] Integrated frameless window controls (Close, Minimize, Maximize) with custom SVG icons.
    - [ ] Electron window drag region (`-webkit-app-region: drag` / `no-drag` on interactives).
    - [ ] Real-time Git branch and status indicator bound to active repository context.
  - [ ] 1.2.4 Implement **Center Zone**:
    - [ ] Compact file path breadcrumb view (workspace / path / activeFile.ext).
    - [ ] Workspace focus and dirty-state indicators.
  - [ ] 1.2.5 Implement **Right Zone**:
    - [ ] Micro-activity status ring (pulsing canvas or SVG ring for background compilation/tests).
    - [ ] Passive alert badge showing quarantined notification count.

- [ ] **1.3 Window Integration & Desktop Snapping**
  - [ ] 1.3.1 Ensure Windows native snapping (Aero Snap, WinKey + Arrows) works seamlessly on frameless window.
  - [ ] 1.3.2 Test multi-monitor dragging, DPI scaling transitions, and fullscreen toggle (`F11`).
  - [ ] 1.3.3 Verify window minimize/restore state persistence across reboots.

---

## Phase 2: Futuristic Hovers & Notification Quarantine (Days 8–14)

### Goal:
Eliminate visual jitter and toast interruptions; replace them with frosted glass tabbed hovers and ambient status cues.

### Tasks & Sub-Tasks:

- [ ] **2.1 Toast Notification Quarantine**
  - [ ] 2.1.1 Intercept `INotificationService` toast creation pipeline (`src/vs/workbench/services/notification/browser/notificationService.ts`).
  - [ ] 2.1.2 Disable bottom-right popup toast overlay rendering entirely (`notificationsToasts.ts`).
  - [ ] 2.1.3 Connect notification events to the Micro-HUD Right Zone:
    - [ ] High-priority / error events: pulse the status ring softly in ambient crimson.
    - [ ] Info / warning events: silent numeric badge increment on alert counter.
  - [ ] 2.1.4 Implement slide-out **Passive History Drawer** triggered by clicking the Micro-HUD alert badge:
    - [ ] Non-modal, slide-in chronological event log.
    - [ ] Filter by source extension, timestamp, and severity.

- [ ] **2.2 Glassmorphic Tabbed Hover Overlay**
  - [ ] 2.2.1 Extend Monaco editor hover widget (`src/vs/editor/contrib/hover/browser/hoverWidget.ts`).
  - [ ] 2.2.2 Introduce resting pause threshold: delay popup by 600ms or bind to dedicated hotkey (`Ctrl + K, Ctrl + I`).
  - [ ] 2.2.3 Apply glassmorphic visual styling:
    - [ ] Hardware-accelerated frosted glass backdrop (`backdrop-filter: blur(12px)`).
    - [ ] Translucent dark surface background (`rgba(...)` from Praxis Ink/Surface tokens).
    - [ ] 1px subtle border outline and soft elevation drop shadow.
  - [ ] 2.2.4 Implement structured three-tab navigation layout:
    - [ ] **Tab 1: Signature** (concise symbol definition, parameters, return type).
    - [ ] **Tab 2: Code References** (peek call-sites / usages).
    - [ ] **Tab 3: Documentation** (Markdown docstrings, remarks, JSDoc).
  - [ ] 2.2.5 Implement flicker prevention and mouse-leave cleanup disposables.

---

## Phase 3: Scoped Editor Terminals & Diff Staging (Days 15–21)

### Goal:
Confine terminals strictly to active editor split groups without disrupting overall reading flow; introduce pre-save diff staging for AI edits.

### Tasks & Sub-Tasks:

- [ ] **3.1 Scoped Editor Split Terminals**
  - [ ] 3.1.1 Modify terminal service / instance management to support column-scoped attachment.
  - [ ] 3.1.2 Implement **Tile Docking Mode**:
    - [ ] Anchor the terminal as a sub-pane locked strictly within the focused Editor Column / Tile.
    - [ ] Keep adjacent editor columns at 100% full vertical height.
    - [ ] Retain independent resize handles per split column.
  - [ ] 3.1.3 Implement **Floating Overlay Drawer Mode**:
    - [ ] Hotkey toggle (`Ctrl + \`` or custom) opens a floating terminal overlay directly over active editor.
    - [ ] Does not trigger grid reflow or re-layout of open editor buffers.

- [ ] **3.2 Generative AI In-Memory Diff Staging Layer**
  - [ ] 3.2.1 Create an in-memory buffer staging layer for incoming AI code modifications.
  - [ ] 3.2.2 Provide a side-by-side or inline preview before applying changes to disk.
  - [ ] 3.2.3 Integrate single-click accept/reject controls directly in the staged view.

---

## Phase 4: Core Optimization & De-bloating (Days 22–28)

### Goal:
Strip out obsolete built-in baggage, eliminate all telemetry/experimentation loops, prune startup contributions, and optimize runtime performance.

### Tasks & Sub-Tasks:

- [ ] **4.1 Built-in Extensions & Baggage Pruning**
  - [ ] 4.1.1 Audit and remove or decouple legacy task runners (`grunt`, `gulp`, `jake`) from `extensions/`.
  - [ ] 4.1.2 Review bundled language extensions and trim non-essential grammars/tools (making them on-demand via Open VSX).
  - [ ] 4.1.3 Audit all `extensions/*/package.json` activation events to purge eager `*` activations and enforce strict lazy activation.
  - [ ] 4.1.4 Prune legacy upstream theme definitions and unused media assets.

- [ ] **4.2 Telemetry, Experiments & Network Churn Removal**
  - [ ] 4.2.1 Strip or stub `@microsoft/1ds-core-js` and telemetry upload loops across Main, Shared, and Renderer processes.
  - [ ] 4.2.2 Decommission the Experimentation Service (TAS client / A/B experiments) and remote config fetchers.
  - [ ] 4.2.3 Disable automatic background update checkers, survey prompts, and crash reporter beacons.
  - [ ] 4.2.4 Strip or stub cloud tunnel service autostarts (`vscodeoss-tunnel`, `vscodeoss-tunnelservice`) for clean local operation.

- [ ] **4.3 Workbench Lifecycle & Startup Contribution Deferral**
  - [ ] 4.3.1 Audit `IWorkbenchContributionsRegistry` contributions registered at `LifecyclePhase.Starting` and `LifecyclePhase.Ready`.
  - [ ] 4.3.2 Eliminate welcome walkthrough parsers, onboarding wizards, and release note fetchers from the critical paint path.
  - [ ] 4.3.3 Remove dead Copilot / cloud hooks and residual stubs from core services.
  - [ ] 4.3.4 Streamline session restoration to only hydrate active visible editor buffers and core UI chrome on first paint.

- [ ] **4.4 Runtime, Chromium & Electron Tuning**
  - [ ] 4.4.1 Audit IPC channels across Main, Shared, and Renderer processes; batch noisy messages and eliminate synchronous IPC.
  - [ ] 4.4.2 Tune Electron startup flags in `scripts/code.bat` and main process initialization (memory limits, code caching).
  - [ ] 4.4.3 Optimize editor rendering defaults (reduce minimap canvas overhead, throttle smooth scrolling, optimize bracket pair caching).
    - [x] 4.4.3a Default minimap to block mode (`editor.minimap.renderCharacters: false`) eliminating canvas glyph layout overhead.
    - [x] 4.4.3b Implement predictive text model pre-warming in Explorer (`onMouseOver` / `onDidChangeFocus`) to eliminate cold-switch latency.
    - [x] 4.4.3c Expose `onMouseOver` and `onMouseOut` event forwarders on `AsyncDataTree`.

- [ ] **4.5 Packaging & Distribution Footprint**
  - [ ] 4.5.1 Strip unused language `.pak` files from the Electron distribution (retaining English and active targets).
  - [ ] 4.5.2 Prune unused root `node_modules` devDependencies from distribution packaging scripts.
  - [ ] 4.5.3 Establish benchmark metrics using `npm run perf` to quantify startup and memory improvements.

---

## Phase 5: Two-Tier Local Intelligence Arbiter (Days 29–35)

### Goal:
Deploy an isolated, offline, sub-40ms System One arbiter using quantized ModernBERT (~210MB) to sanitize AI diffs, suppress cascading LSP errors, and triage alerts.

### Tasks & Sub-Tasks:

- [ ] **5.1 Background Utility Process Architecture**
  - [ ] 5.1.1 Scaffold dedicated Electron Utility Process (`src/vs/platform/orbisArbiter/node/arbiterProcess.ts`).
  - [ ] 5.1.2 Establish high-performance, asynchronous IPC communication channel between Renderer and Utility Process.
  - [ ] 5.1.3 Enforce bounded query timeouts (<50ms) with immediate pass-through fallback to keep UI locked at 60fps.
  - [ ] 5.1.4 Integrate ONNX Runtime / candle runtime with INT8 quantized ModernBERT model payload (~210MB RAM ceiling).

- [ ] **5.2 Anti-Slop Diff Arbiter**
  - [ ] 5.2.1 Intercept incoming generative AI patch stream against current document buffer.
  - [ ] 5.2.2 Detect and strip:
    - [ ] Unrequested formatting or indentation modifications on untouched lines.
    - [ ] Redundant boilerplate comments (e.g., repeating method names or obvious descriptions).
    - [ ] Hallucinated imports or deleted functional code.
  - [ ] 5.2.3 Output strictly minimal, verified functional logic delta into editor staging layer.

- [ ] **5.3 LSP Diagnostic Firewall**
  - [ ] 5.3.1 Intercept Language Server Protocol diagnostic stream (`src/vs/workbench/services/diagnostics/`).
  - [ ] 5.3.2 Construct diagnostic correlation tree for multi-error spikes.
  - [ ] 5.3.3 Isolate root-cause syntax/token failure and suppress cascading downstream secondary errors.

- [ ] **5.4 Ambient Notification Classifier**
  - [ ] 5.4.1 Feed raw notification events and origin metadata into the local arbiter.
  - [ ] 5.4.2 Fast binary classification: `CRITICAL_ACTION_REQUIRED` vs `PASSIVE_INFORMATIONAL`.
  - [ ] 5.4.3 Route to Micro-HUD ambient indicator or silent history drawer accordingly.

---

## Phase 6: Hardening, Benchmarking & Acceptance Sign-off (Days 36–38)

### Goal:
Validate all acceptance criteria, memory bounds, and latency budgets.

### Tasks & Sub-Tasks:

- [ ] **6.1 Acceptance Criteria Verification**
  - [ ] 6.1.1 **Canvas Utilization**: Verify active code editing area exceeds 96% of total window area on clean startup.
  - [ ] 6.1.2 **Zero Toast Interruptions**: Confirm zero bottom-right popup toasts mount across builds, tests, or notifications.
  - [ ] 6.1.3 **Inference Latency**: Benchmark local arbiter; verify deterministic classification response in <40ms on standard CPU.
  - [ ] 6.1.4 **Memory Footprint**: Profile idle workbench + utility process; ensure combined memory remains under 550MB RAM.

- [ ] **6.2 Performance & Regression Testing**
  - [ ] 6.2.1 Run `npm run perf` and record startup timing comparisons against baseline.
  - [ ] 6.2.2 Execute full test suite (`scripts/test.bat`) to verify zero core regressions.
  - [ ] 6.2.3 Package release build and verify distribution payload size reduction.
