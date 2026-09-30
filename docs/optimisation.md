# Orbis IDE — Optimization & De-bloating Blueprint

## Objective
Transform Orbis from a heavyweight Code OSS baseline into an exceptionally lean, responsive, and privacy-respecting development environment. 

### Core Targets
- **Cold Startup**: Sub-second editor readiness.
- **Memory Footprint**: Substantial reduction in baseline idle RAM.
- **Network Cleanliness**: Zero outbound telemetry, A/B experiments, or unwanted cloud polling.
- **Distribution Size**: Trim unnecessary bundled assets, locales, and legacy extensions.

---

## Action Checklist

### Phase 1: Built-in Extensions & Baggage Pruning
- [ ] **Audit & Prune Legacy Task Runners**: Remove or decouple `grunt`, `gulp`, `jake` from built-in extensions (`extensions/`).
- [ ] **Review Bundled Languages & Grammars**: Audit `extensions/` to keep only core web/system languages; convert non-essential ones to downloadable marketplace extensions.
- [ ] **Enforce Strict Lazy Activation**: Audit remaining extensions' `package.json` activation events to eliminate eager `*` activations so the Extension Host stays idle until files are opened.
- [ ] **Theme & Media Cleanup**: Prune unused upstream color themes and legacy assets now that Praxis Dark is our default design language.

### Phase 2: Telemetry, Experiments & Network Churn Removal
- [ ] **Neutralize Telemetry Pipelines**: Strip or stub `@microsoft/1ds-core-js` and telemetry publisher loops across Main, Shared, and Renderer processes.
- [ ] **Decommission Experimentation (TAS)**: Remove the A/B testing client and dynamic experiment config fetchers.
- [ ] **Disable Cloud Tunnel Auto-Services**: Disable default tunnel background services and mutexes (`vscodeoss-tunnel`, `vscodeoss-tunnelservice`) for standalone local usage.
- [ ] **Purge Survey & Feedback Prompts**: Remove automated NPS/feedback prompt controllers and crash beacon services.

### Phase 3: Workbench Lifecycle & Startup Deferral
- [ ] **Audit `IWorkbenchContributionsRegistry`**: Inspect all workbench contributions registered at `LifecyclePhase.Starting` and `LifecyclePhase.Ready`.
- [ ] **Defer or Remove Onboarding Walkthroughs**: Eliminate welcome walkthrough parsers and release note auto-fetchers from the critical paint path.
- [ ] **Decouple Residual Copilot / Cloud Hooks**: Clean out dead references and unneeded stubs from build and configuration files.
- [ ] **Streamline Session Restoration**: Ensure window restoration restores only the active editor buffer and essential UI chrome first.

### Phase 4: Runtime, Chromium & Electron Tuning
- [ ] **IPC Traffic Audit**: Identify and batch noisy IPC messages between the Main process, Shared process, and Renderer.
- [ ] **V8 & Chromium Flag Optimization**: Tune startup flags in [code.bat](file:///d:/Praxis/Orbis/Orbis/scripts/code.bat) and main process initialization (memory limits, code caching).
- [ ] **Editor DOM & Rendering Defaults**: Evaluate defaults for heavy rendering widgets (minimap canvas overhead, sticky scroll DOM depth, heavy bracket pair match caching).

### Phase 5: Packaging & Distribution Footprint
- [ ] **Prune Locales**: Strip unused language `.pak` files from the Electron distribution (retaining English and active targets).
- [ ] **Dependency Audit**: Review root `package.json` dependencies for redundant or heavy packages.
- [ ] **Benchmark & Validate**: Establish baseline vs. optimized metrics using `npm run perf` and memory profiles.

---

## Queued Small Optimizations (High-Impact & Animation-Preserving)

These candidate optimizations eliminate micro-stutters and cold latency without sacrificing visual polish, caret easing, or smooth scrolling:

1. **Font Measurement Pre-warming (DOM Reflow Elimination)**
   - **Target**: `src/vs/editor/browser/config/charWidthReader.ts` & `src/vs/editor/browser/config/fontMeasurements.ts`
   - **Mechanism**: `DomCharWidthReader.read()` injects 30+ spans with 256 repeated characters into `document.body` and measures `offsetWidth` synchronously. If this runs when the first editor opens, it causes a forced layout / reflow during the initial frame.
   - **Action**: Call `FontMeasurements.readFontInfo()` during idle time right after workbench startup or during splash screen load so the font cache is 100% warm before any file is opened.

2. **Occurrences Highlighting Debounce (Cursor Motion Polish)**
   - **Target**: `src/vs/editor/common/config/editorOptions.ts` (`occurrencesHighlightDelay`)
   - **Mechanism**: Defaults to `0` ms. Every single cursor step immediately queries the language service and text model without debounce, causing micro-stutters during rapid arrow-key or Vim navigation in large files.
   - **Action**: Default `editor.occurrencesHighlightDelay` to `150` ms. Resting on a symbol highlights instantly, but fast cursor movement stays locked at 120fps.

3. **Sticky Scroll Depth & AST Query Trimming**
   - **Target**: `src/vs/editor/common/config/editorOptions.ts` (`stickyScroll.maxLineCount`)
   - **Mechanism**: Defaults to `5` lines, consuming vertical editing height and requiring 5 nested DOM widgets with AST queries against outline/folding models.
   - **Action**: Reduce default `editor.stickyScroll.maxLineCount` to `3` lines. Cuts sticky scroll DOM nodes and AST queries by 40% while preserving context.

4. **CSS Layout Containment on List Rows**
   - **Target**: `src/vs/base/browser/ui/list/list.css`
   - **Mechanism**: `.monaco-list-row` lacks explicit CSS layout containment. Hovering and selecting deep tree items can cause Chromium to recalculate geometry up the DOM tree.
   - **Action**: Add `contain: layout style` to `.monaco-list-row` and `content-visibility: auto` to inactive view containers to prevent layout recalculation bubbling and isolate paint invalidations.

5. **Chromium Out-of-Process (OOP) 2D Canvas Rasterization**
   - **Target**: `src/mainImpl.ts` (`featuresToEnable`)
   - **Mechanism**: Minimap, terminal canvases, and editor decorations can contend with DOM layout on the UI thread during rasterization.
   - **Action**: Add `CanvasOopRasterization` to Chromium `featuresToEnable` switches in `src/mainImpl.ts` to offload 2D canvas drawing to the dedicated GPU raster thread.

6. **V8 Heap Stability — Idle Eviction for Stale Unreferenced Text Models**
   - **Target**: `src/vs/workbench/services/textfile/common/textFileEditorModelManager.ts`
   - **Mechanism**: `mapResourceToModel` retains all resolved models indefinitely. With our 10-tab limit, files pre-warmed or opened hours ago remain in memory.
   - **Action**: Introduce a low-priority idle sweep (e.g. every 10–15 minutes) that disposes clean, non-dirty models not present in the active 10-tab working set.

