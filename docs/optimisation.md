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
