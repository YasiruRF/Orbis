# ORBIS IDE: Architecture, Design & Technical Specification

**Project Codename**: Orbis | **Base Platform**: Code - OSS (Open Source Workbench Core)  
**Architecture**: Decoupled Renderer Workbench with Local On-Device System One Arbiter (Quantized ModernBERT, ~210MB)

---

## 1. Executive Summary & Design Thesis

Modern code editors and generative AI programming assistants suffer from cognitive fatigue and visual congestion. Standard VS Code and its generative AI extensions (Cursor, Copilot, Windsurf) exacerbate cognitive fatigue through:

- **Peripheral Chrome Creep**: Excessive toolbars, activity strips, title bars, and bottom status panels consume up to 25% of vertical screen space.
- **Notification Cacophony**: Competing toast banners pop up over editor text, terminal sessions, and navigation trees.
- **Tooltip Jitter**: Immediate hover tooltips obscure active code and display dense, unstructured type dumps.
- **Rigid Panel Ergonomics**: Opening a terminal horizontally slices across all open split editors, breaking reading flow.
- **Generative AI Slop**: AI models reformat untouched lines, inject verbose comments, and cause cascading linting and type errors.

### The Orbis Resolution

- **The Zen Canvas**: A borderless, minimalist workspace allocating over 96% of screen real estate to code, featuring a single 26px Micro-HUD, scoped editor terminals, and glassmorphic hover overlays.
- **The Local Intelligence Arbiter**: An offline, sub-40ms decision model running in an isolated process that intercepts AI code patches to strip slop, filters cascading compiler diagnostics, and quarantines intrusive notifications into a calm status indicator.

---

## 2. Complete Workbench Visual Layout

Orbis decouples workbench rendering from background decision-making to maintain locked 60fps UI performance while running local intelligence.

```
+----------------------------------------------------------------------------------------------------+
|  [x] [-] [+]  branch:main*   |    orbis-core / engine / arbiter.ts     |   (o) indexing   [!] 2    |  <-- 26px Micro-HUD
+----------------------------------------------------------------------------------------------------+
|                                  |                                                                 |
|  EDITOR COLUMN A (Primary)       |  EDITOR COLUMN B (Secondary Split)                              |
|                                  |                                                                 |
|  export async function process() |  describe("Processor Suite", () => {                            |
|    const session = init();       |    it("handles payload correctly", () => {                      |
|                                  |                                                                 |
|    +-----------------------------+-----------------------------------+                             |
|    | GLASS HOVER OVERLAY         | [ Tab: Signature ] [ Tab: Docs ]  |                             |
|    | function process(): Session | Returns active worker context.    |                             |
|    +-----------------------------+-----------------------------------+                             |
|                                  |                                                                 |
|    return session.run();         |  +------------------------------------------------------------+ |
|  }                               |  | SCOPED TERMINAL DRAWER (Locked to Column B)                | |
|                                  |  | $ pnpm test:unit                                            | |
|                                  |  | PASS tests/processor.spec.ts (1.2s)                         | |
|                                  |  +------------------------------------------------------------+ |
|                                  |                                                                 |
+----------------------------------+-----------------------------------------------------------------+
```

---

## 3. Detailed UI & Ergonomic Specifications

### 3.1. Unified 26px Micro-HUD Header
Replaces the Titlebar, Menu Bar, Activity Bar, and Status Bar with a single 26px utility header.

```
+----------------------------------------------------------------------------------------------------+
| LEFT ZONE                      | CENTER ZONE                         | RIGHT ZONE                  |
| [x] [-] [+]   git:(main)*      | workspace / src / runtime.ts        | (o) Ready     [!] 2 alerts  |
| Window Controls & Branch       | Path Breadcrumbs & Workspace Focus  | Background State & Notifs   |
+----------------------------------------------------------------------------------------------------+
```

- **Left Zone**: Native window controls (Close, Minimize, Maximize) unified with a non-intrusive Git branch indicator.
- **Center Zone**: Clean breadcrumb path showing current file hierarchy and active project workspace.
- **Right Zone**: Micro-activity status ring (indicating background compilation or test runs) and a subtle passive alert badge.

---

### 3.2. Futuristic Glassmorphic Hover Window
Addresses hover popup fatigue with an intentional delay and clean tabbed layout.

```
+---------------------------------------------------------------------------------------+
| Symbol: authenticateUser(request, response)                                           |
+---------------------------+-------------------------------+---------------------------+
| [ Tab 1: Signature ]      | [ Tab 2: Code References ]    | [ Tab 3: Documentation ]  |
+---------------------------+-------------------------------+---------------------------+
|                                                                                       |
| async function authenticateUser(                                                      |
|   request: IncomingAuthRequest,                                                       |
|   response: ServerResponse                                                            |
| ): Promise<AuthenticatedSession>                                                      |
|                                                                                       |
| Summary: Verifies bearer tokens, hydrates user session, and enforces authorization.   |
| Return Type: Active session token or rejects with UnauthorizedException.              |
+---------------------------------------------------------------------------------------+
```

- **Intentional Trigger Threshold**: 600ms resting pause or explicit hotkey toggle to prevent accidental popup flash during normal cursor navigation.
- **Glassmorphic Aesthetic**: Frosted glass backing with hardware-accelerated background blur, translucent borders, and soft elevation shadows.
- **Structured Tabs**: One-click switching between concise type definitions, active call-site references, and human documentation.

---

### 3.3. Toast Notification Quarantine & Ambient Triage
Completely eliminates bottom-right floating toast banners.

```
[ Incoming Alert / Notification Event ]
                   |
                   v
      [ Local Intelligence Filter ]
                   |
        +----------+----------+
        |                     |
        v                     v
[ Critical System Failure ]   [ Informational / Build Nit ]
        |                     |
        v                     v
Micro-HUD Status Ring         Silent Increment on Badge
Pulses Ambient Red            Logged to Non-Modal Drawer
(Zero Viewport Blocking)      (Zero Viewport Blocking)
```

- **Zero Bottom-Right Toasts**: No floating popups are allowed to mount over editor code or terminal panes.
- **Ambient Indicator**: High-priority alerts cause the Micro-HUD status ring to pulse softly in ambient red.
- **Passive History Drawer**: Clicking the notification badge slides out a non-intrusive drawer displaying chronological notices and warnings on demand.

---

### 3.4. Scoped Editor Split Terminals
Prevents terminal panels from breaking the editor grid.

```
+---------------------------------------+---------------------------------------+
| Editor Tile 1 (routes/auth.ts)        | Editor Tile 2 (tests/auth.test.ts)    |
| Full Vertical Screen Height           | Upper Half: Active Test File Code     |
| Uninterrupted Reading Flow            |                                       |
|                                       +---------------------------------------+
|                                       | Scoped Tile Terminal                  |
|                                       | Locked strictly under Tile 2          |
|                                       | $ pnpm test auth                      |
|                                       | Tests: 4 passed, 4 total              |
+---------------------------------------+---------------------------------------+
```

- **Tile Anchoring**: Terminals spawn as child panes strictly within the focused editor column, preserving full vertical height for adjacent split files.
- **Floating Overlay Option**: Hotkey toggles an overlay drawer that floats smoothly over the active editor without resizing or triggering editor reflow.

---

## 4. System Architecture & Two-Tier Intelligence Loop

Orbis separates user interface rendering from local decision-making to maintain 60fps performance while executing sub-40ms machine learning classifications.

```
+-----------------------------------------------------------------------------------+
|                           WORKBENCH RENDERER PROCESS                              |
|                                                                                   |
|   [ Micro-HUD Header ]    [ Active Monaco Editor ]    [ Scoped Pane Terminal ]    |
|             ^                         ^                         ^                 |
|             |                         |                         |                 |
|             +-------------------------+-------------------------+                 |
|                                       |                                           |
|                        Internal Interceptor Proxy                         |
+---------------------------------------+-------------------------------------------+
                                        | Asynchronous IPC Channel
                                        v
+-----------------------------------------------------------------------------------+
|                        ISOLATED BACKGROUND UTILITY PROCESS                        |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   | Local System One Arbiter Engine                                           |   |
|   | - Low-latency classification model (INT8 quantized, ~210MB memory)        |   |
|   | - Sub-40ms deterministic evaluation on standard CPU                       |   |
|   +---------------------------------------------------------------------------+   |
|             |                              |                             |        |
|             v                              v                             v        |
|   +-------------------+          +-------------------+         +---------------+  |
|   | Anti-Slop Diff    |          | LSP Diagnostic    |         | Notification  |  |
|   | Arbiter           |          | Firewall          |         | Gatekeeper    |  |
|   | Validates AI code |          | Suppresses        |         | Quarantines   |  |
|   | patches & strips  |          | cascading compiler|         | noisy toasts  |  |
|   | boilerplate       |          | error storms      |         | into HUD ring |  |
|   +-------------------+          +-------------------+         +---------------+  |
+-----------------------------------------------------------------------------------+
```

### 4.1. Anti-Slop Diff Arbiter (Generative Patch Sanitizer)
- **Input**: Original buffer content paired with the incoming generative AI patch stream.
- **Analysis**: Evaluates semantic drift and checks whether the patch modifies untouched lines, alters indentation arbitrarily, or introduces unsolicited boilerplate comments.
- **Output**: Strips cosmetic and unrequested changes, applying strictly the functional logic delta to the editor buffer.

### 4.2. LSP Diagnostic Firewall (Noise Suppression Engine)
- **Input**: Stream of compiler and language server diagnostics (type errors, warnings, syntax checks).
- **Analysis**: Determines whether a cascade of multiple errors stems from a single upstream syntax or token failure.
- **Output**: Identifies the single root-cause error and suppresses downstream secondary squiggles, keeping the code view clean.

### 4.3. Ambient Notification Classifier
- **Input**: Alert metadata, severity tier, and originating extension identity.
- **Analysis**: Categorizes alerts into critical interruptions versus passive informational notes.
- **Output**: Routes non-essential alerts to the background drawer while reflecting critical status via the Micro-HUD indicator.

---

## 5. Development Roadmap & Milestones

### Phase 1: Shell Consolidation & Micro-HUD (Days 1–7)
- Configure the open-source workbench core and build pipeline.
- Deactivate default peripheral chrome (Activity Bar and Status Bar).
- Construct the 26px Micro-HUD header with window drag functionality, Git branch display, and breadcrumbs.
- Validate multi-monitor positioning, window snapping, and high-DPI scaling.

### Phase 2: Futuristic Hovers & Notification Quarantine (Days 8–14)
- Neutralize toast notification containers to prevent lower-right popup mounting.
- Connect notification routing to the Micro-HUD ambient indicator.
- Build frosted glass hover overlays with background blur and three-tab navigation.
- Implement hover timing controls to prevent flicker during fast cursor movements.

### Phase 3: Scoped Terminal Docking & Diff Staging (Days 15–21)
- Reconfigure terminal creation to bind within active editor split groups rather than opening a global bottom panel.
- Implement the slide-out terminal overlay mode for quick shell commands.
- Build the in-memory diff preview layer to hold generative AI modifications prior to saving.

### Phase 4: Local Intelligence Engine Integration (Days 22–28)
- Bundle the quantized INT8 System One model inside the background utility process.
- Establish the low-latency IPC pipeline between the renderer and background process.
- Connect the local engine to the compiler diagnostic stream for root-cause filtering.
- Connect the local engine to generative patch streams for automated slop removal.

---

## 6. AI Collaborator Prompting & Review Instructions

### Instructions for Code Authoring (Antigravity IDE / Gemini)
- **Respect the workbench architecture**: Use internal service registration and dependency injection rather than ad-hoc global singletons.
- **Maintain native workbench rendering**: Avoid injecting heavy third-party UI runtimes into core surfaces; utilize native element construction and disposable resource management.
- **Ensure all styles are dynamic**: Inherit theme variables from the workbench theme engine.

### Instructions for Architecture Review & Auditing (Claude Code)
- **Lifecycle & Resource Cleanup**: Verify that all event listeners, hover overlays, and terminal sessions register with cleanup disposables on teardown.
- **Layout Robustness**: Confirm that custom header and terminal modifications do not disrupt editor split operations, grid layouts, or window state restoration.
- **Inference Latency Budget**: Ensure all local intelligence queries run asynchronously off the UI thread with bounded timeouts under 50ms and immediate fallbacks if processing pauses.

---

## 7. Verification & Acceptance Criteria

1. **Maximum Canvas Utilization**: The active code editing canvas occupies greater than 96% of total window pixels on startup.
2. **Zero Popup Interruptions**: No toast notifications mount in the bottom-right corner during build steps, compiler warnings, or background tasks.
3. **Sub-40ms Classification Loop**: The local intelligence arbiter returns structured classifications in under 40ms on standard multi-core CPUs.
4. **Stable Memory Footprint**: The complete editor workbench and bundled local model maintain an idle memory footprint under 550MB RAM.
