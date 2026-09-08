# Research: Making pi (v0.84.4) remember the selected model + thinking level per project, surviving `/new` and new sessions

Research date: 2026-02-14 · Subject: pi coding agent CLI v0.84.4 (installed via mise at `/home/ib/.local/share/mise/installs/pi/0.84.4/`, package `@earendil-works/pi-coding-agent`, repo `earendil-works/pi`, formerly `badlogic/pi-mono`).

**Method note:** The local mise install ships docs (`pi/README.md`, `pi/docs/`) and the built bundle, but not the TypeScript source. All source-code citations below are from the exact matching version tag `v0.84.4` on GitHub (`https://github.com/earendil-works/pi/blob/v0.84.4/...`), verified to exist via the GitHub tags API. Docs cited are the local files under `/home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/` (identical to the `v0.84.4` tag).

## Summary

Yes — pi has a project-level settings file: `.pi/settings.json` in the current working directory (no parent-directory walk). Setting `defaultProvider`, `defaultModel`, `defaultThinkingLevel`, and optionally `modelThinkingLevels` / `enabledModels` there pins the startup model and thinking level for that folder, and those values are re-applied on every `/new` and every fresh `pi` launch in that directory. The catch: the interactive "Ctrl+S save default" in the `/model` and `/thinking` pickers always writes to the **global** `~/.pi/agent/settings.json` — project defaults can only be created by hand-editing `.pi/settings.json` (or via shell-alias flags). Everything hinges on the project being **trusted**: until you answer the trust prompt (or run `/trust`), pi ignores `.pi/settings.json` entirely.

## Findings

1. **Project settings file: `<cwd>/.pi/settings.json` — exact name, cwd only.** Docs: "`.pi/settings.json` | Project (current directory)" (`docs/settings.md`, intro table). Source: `SettingsManager.create()` builds the project path as `join(resolvedCwd, CONFIG_DIR_NAME, "settings.json")`, and `CONFIG_DIR_NAME = ".pi"` comes from the package's `piConfig.configDir` (`src/core/settings-manager.ts`, `src/config.ts`, `package.json` → `"piConfig": {"configDir": ".pi"}`). There is **no parent-directory walk** for settings (unlike `AGENTS.md`/skills, which do walk up) — `.pi/settings.json` is read only from the exact cwd. [docs/settings.md](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/settings.md) · [settings-manager.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/settings-manager.ts) · [config.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/config.ts)

2. **Schema: keys for model + thinking defaults.** From the `Settings` interface and the docs "Model & Thinking" table:
   - `defaultProvider` — string, e.g. `"anthropic"`
   - `defaultModel` — string, the **bare model ID** (not `provider/id`), e.g. `"claude-opus-4-8"`; provider comes from `defaultProvider`
   - `defaultThinkingLevel` — one of `"off" | "minimal" | "low" | "medium" | "high" | "xhigh" | "max"` (`ThinkingLevel`, `THINKING_LEVEL_OPTIONS` in `src/core/defaults.ts`)
   - `modelThinkingLevels` — object mapping `"provider/modelId"` → thinking level; per-model startup thinking override (this is how you pin level per model instead of globally)
   - `enabledModels` — string array of patterns for Ctrl+P cycling, same format as `--models`
   [docs/settings.md §Model & Thinking + Example](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/settings.md) · [settings-manager.ts `Settings`](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/settings-manager.ts) · [defaults.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/defaults.ts)

3. **Project settings override global; merged read model.** "Project settings (`.pi/settings.json`) override global settings. Nested objects are merged" (deep merge, not replace — `deepMergeSettings` in `settings-manager.ts`). Docs show a worked `compaction.reserveTokens` override example; the same merge applies to `defaultModel`/`defaultThinkingLevel`/`modelThinkingLevels`. The getters used at startup (`getDefaultProvider()`, `getDefaultModel()`, `getDefaultThinkingLevel()`, `getModelThinkingLevel()`, `getEnabledModels()`) all read the **merged** settings. [docs/settings.md §Project Overrides](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/settings.md) · [settings-manager.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/settings-manager.ts)

4. **Trust gate: `.pi/settings.json` is ignored until the project is trusted.** "Trusting a project allows pi to load `.pi/settings.json` and `.pi` resources" (`docs/settings.md` §Project Trust; same in README §Project Trust). Interactive startup prompts once; `/trust` persists the decision to `~/.pi/agent/trust.json` (restart required to apply). Non-interactive modes (`-p`, `--mode json`, `--mode rpc`) never prompt: they use `defaultProjectTrust` from global settings (`"ask"` default and `"never"` = ignore project settings; `"always"` = trust), overridable per run with `-a`/`--approve` or `-na`/`--no-approve`. In source, the runtime settings manager is created with `{ projectTrusted }`, and `loadFromStorage` returns `{}` for an untrusted project (`src/core/settings-manager.ts` `loadFromStorage`, `src/main.ts` `createRuntime`). **Practical consequence: after creating `.pi/settings.json`, launch `pi` in the project and answer the trust prompt (or run `/trust`, then restart).** [docs/settings.md §Project Trust](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/settings.md) · [main.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/main.ts)

5. **Ctrl+S "save default" writes to the GLOBAL file only — there is no interactive way to save project defaults.** In `settings-manager.ts`, every setter invoked by the pickers — `setDefaultModelAndProvider()`, `setDefaultThinkingLevel()`, `setModelThinkingLevel()`, `setEnabledModels()` — mutates `this.globalSettings` and enqueues a write to the `"global"` scope (`~/.pi/agent/settings.json`). In the UI: `/model` + Enter switches model for the session only (`setModel(model, { persist: false })`, status `Model: <id>`); Ctrl+S in the picker persists (`persist: true`, status `Default model: <provider>/<id>` → `setDefaultModelAndProvider` → global). Same for `/thinking` (`selectThinkingLevel(level, persist)` → `setDefaultThinkingLevel`). So the docs' "saved with Ctrl+S in `/model`, or edited manually" is accurate: **to get project-scoped defaults you must edit `.pi/settings.json` manually.** [docs/settings.md](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/settings.md) · [settings-manager.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/settings-manager.ts) · [interactive-mode.ts `showModelSelector`/`showThinkingSelector`](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/modes/interactive/interactive-mode.ts) · [agent-session.ts `setModel`/`setThinkingLevel`](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/agent-session.ts)

6. **What `/new` does: brand-new session, model/thinking re-resolved from scratch — with project settings honored.** `/new` maps to `handleClearCommand()` → `runtimeHost.newSession()` (`interactive-mode.ts`), which tears down the current session and calls the same `createRuntime` factory used at startup (`agent-session-runtime.ts` `newSession()`; `main.ts` `createRuntime`). That factory re-creates a cwd-bound `SettingsManager` (merging trusted project settings) and resolves the model exactly like startup. Resolution order (`main.ts buildSessionOptions` + `sdk.ts createAgentSession` + `model-resolver.ts findInitialModel`):
   1. **CLI flags of the current launch** — `--model [provider/]id[:level]`, `--provider`, `--thinking` (these are re-applied on `/new` because the factory closes over the parsed args);
   2. **Scoped models** — from `--models` flag or `enabledModels` setting (merged: project wins); if the saved default (merged `defaultProvider`+`defaultModel`) is inside the scope it is preferred, else the first scoped model; a pattern-suffix `:level` on the scoped pattern sets thinking;
   3. **Saved default from settings** — `defaultProvider`+`defaultModel` from **merged** settings (project `.pi/settings.json` overrides global), used when it exists and the provider has auth;
   4. **First available model** with configured auth;
   5. Nothing (no model).
   Thinking level for a new session: explicit CLI `--thinking` → scoped-pattern `:level` → `modelThinkingLevels["provider/modelId"]` → `defaultThinkingLevel` → built-in default `"medium"` (`DEFAULT_THINKING_LEVEL` in `src/core/defaults.ts`; `sdk.ts` doc: "Default: from settings, else 'medium'"), then clamped to the model's supported levels (`clampThinkingLevel`). **So a project-level `.pi/settings.json` default survives `/new` indefinitely, while a session-only `/model` change does not carry into the new session.** [agent-session-runtime.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/agent-session-runtime.ts) · [main.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/main.ts) · [sdk.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/sdk.ts) · [model-resolver.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/model-resolver.ts)

7. **Model/thinking selections are session-scoped and restored on resume — not on `/new`.** Model switches and thinking changes are recorded as `model_change` / `thinking_level_change` entries in the session JSONL (`agent-session.ts`: `sessionManager.appendModelChange(...)`, `appendThinkingLevelChange(...)`; `docs/sessions.md`: "Session files ... contain message entries, model changes, thinking-level changes ..."). Resuming (`pi -c`, `pi -r`, `--session`, `/resume`) restores the session's saved model (`sdk.ts` restores `existingSession.model`; `restoreModelFromSession` falls back with a warning if the model is gone or unauthenticated). The v0.84.3 changelog makes this explicit: model/thinking controls "**keep selections session-scoped, and persist them explicitly with Ctrl+S**". A `/new` session therefore starts from defaults (project-over-global), never from the previous session's ad-hoc selection. [CHANGELOG.md §0.84.3](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/CHANGELOG.md) · [sdk.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/sdk.ts) · [docs/sessions.md](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/sessions.md)

8. **Thinking-level suffix syntax `provider/id:level`.** Patterns split on the **last** colon; the suffix is used as the thinking level only if it is a valid level (`off|minimal|low|medium|high|xhigh|max`), otherwise (interactive scope mode) it warns "Invalid thinking level ... Using default instead", and in strict CLI mode it fails rather than silently resolving to another model (`parseModelPattern`, `resolveCliModel` in `model-resolver.ts`). Docs examples: `pi --model sonnet:high`, `pi --model openai/gpt-4o` (README/usage §Examples). The same `pattern:level` grammar works in `enabledModels` / `--models` (glob patterns too, e.g. `provider/*:high`). Colons inside real model IDs (e.g. OpenRouter `:exacto`, Ollama `llama3.1:8b`) are handled by trying an exact match of the full pattern first. [model-resolver.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/model-resolver.ts) · [docs/usage.md §Examples](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/usage.md) · [README §Model Options](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/README.md)

9. **CLI flags and env vars.** Relevant flags: `--model <pattern>` (supports `provider/id` and `:thinking`), `--provider <name>`, `--thinking <level>`, `--models <patterns>` (Ctrl+P scope). Env vars: there is **no env var that sets the default model/thinking**; `PI_PROVIDER`, `PI_MODEL`, `PI_REASONING_LEVEL` are *outputs* injected into shell-tool commands for introspection, and `PI_CODING_AGENT_DIR` relocates the whole config dir (global settings included) (`docs/environment-variables.md`; `docs/usage.md` §Model Options). A shell alias therefore pins model+level per shell, but unlike `.pi/settings.json` it doesn't vary by directory unless you wrap it: e.g. `alias pi-strict='pi --model anthropic/claude-opus-4-8:high'` — and because the factory re-applies launch flags on `/new`, the alias' pin survives `/new` within that run. [docs/environment-variables.md](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/environment-variables.md) · [README §CLI Reference](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/README.md)

10. **Startup default must resolve to a real, authenticated model.** `findInitialModel` looks up `modelRuntime.getModel(defaultProvider, defaultModelId)` and requires `hasConfiguredAuth(provider)`; otherwise it silently falls through to "first available model" (thinking falls back to defaults). Use the model ID exactly as shown in `/model` (alias-style IDs are preferred over dated variants when matching patterns). [model-resolver.ts `findInitialModel`](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/model-resolver.ts)

11. **Version-specific caveats.**
    - **0.84.3** introduced the `/thinking` selector, searchable defaults, session-scoped selections, and Ctrl+S persistence ("Model and thinking controls" + "Added a `/thinking` selector ...; Ctrl+S saves the selected model as the global default"). On older versions behavior differs.
    - **0.84.4** fixed "saving a default model from a non-empty model scope so it remains available in that scope": persisting now also adds the model to the in-session Ctrl+P scope and appends `provider/id` to `enabledModels` when a non-empty scope exists (`_addPersistedDefaultToNonEmptyScope` in `agent-session.ts`).
    - The repo moved: `badlogic/pi-mono` → `earendil-works/pi` (the v0.84.4 tag and changelog PR links live under `earendil-works/pi`); old `badlogic/pi-mono` raw URLs 404. [CHANGELOG.md §0.84.3–0.84.4](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/CHANGELOG.md) · [agent-session.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/agent-session.ts)

12. **Ctrl+P model cycling scope can also be project-defined.** `enabledModels` in `.pi/settings.json` (merged, project wins) seeds the Ctrl+P cycle each startup/`/new`; `/scoped-models` + Ctrl+S writes it — but again to the **global** file (`setEnabledModels` → global scope). [docs/settings.md §Model Cycling](file:///home/ib/.local/share/mise/installs/pi/0.84.4/pi/docs/settings.md) · [settings-manager.ts](https://github.com/earendil-works/pi/blob/v0.84.4/packages/coding-agent/src/core/settings-manager.ts)

## Recommended setup (practical takeaway)

Pin model + thinking per project by hand-writing `.pi/settings.json` at the project root, then trust the project once.

**1. Create `<project>/.pi/settings.json`:**

```json
{
  "defaultProvider": "anthropic",
  "defaultModel": "claude-opus-4-8",
  "defaultThinkingLevel": "high",
  "modelThinkingLevels": {
    "anthropic/claude-opus-4-8": "high",
    "openai/gpt-5.5": "medium"
  },
  "enabledModels": [
    "anthropic/claude-opus-4-8:high",
    "openai/gpt-5.5:medium"
  ]
}
```

- `defaultModel` is the bare model ID (as shown in `/model`); the provider goes in `defaultProvider`.
- `modelThinkingLevels` keys are `"provider/modelId"`. It overrides `defaultThinkingLevel` for that model — include an entry for your default model if you want the level pinned regardless of which model is default.
- `enabledModels` is optional: it makes Ctrl+P cycle exactly those models with those levels. Delete keys you don't need — any key you omit simply inherits from `~/.pi/agent/settings.json` (nested objects merge; project wins per key).

**2. Trust the project once:** run `pi` in the project → answer the trust prompt with trust (or type `/trust`, then restart pi). Headless runs: set `"defaultProjectTrust": "always"` in `~/.pi/agent/settings.json`, or pass `-a` per run.

**3. Result:** every `pi` launch in this directory and every `/new` (and `/fork`/`/clone`, which go through the same factory) starts on `anthropic/claude-opus-4-8` at thinking `high`; a `pi -c`/`--session` resume of an old session instead restores that session's recorded model/level. Other projects keep using your global defaults, e.g.:

```json
// ~/.pi/agent/settings.json (global fallback)
{
  "defaultProvider": "openai",
  "defaultModel": "gpt-5.5",
  "defaultThinkingLevel": "medium"
}
```

**One-off / alias alternative** (when you don't want a file): `alias pi-hard='pi --model anthropic/claude-opus-4-8:high'` — launch flags outrank settings and are re-applied after `/new` within the same run, but they don't persist across separate `pi` invocations the way `.pi/settings.json` does.

## Sources

- Kept: `pi/docs/settings.md` (local install; settings schema, project overrides, project trust) — the authoritative doc for `.pi/settings.json`.
- Kept: `pi/README.md` (local install; settings table, `/model` Ctrl+S, CLI reference, env vars) — cross-reference hub.
- Kept: `pi/docs/sessions.md`, `pi/docs/usage.md`, `pi/docs/environment-variables.md`, `pi/docs/keybindings.md` (local install) — `/new` semantics, flags, `PI_*` vars, Ctrl+S keybinding ids (`app.models.save`).
- Kept: `pi/CHANGELOG.md` (local install, §0.84.3/0.84.4) — session-scoped selection + Ctrl+S persist introduction; scope-persist fix.
- Kept: `src/core/settings-manager.ts` @ v0.84.4 — proves project path (`cwd/.pi/settings.json`), deep merge, global-only writes for all default setters.
- Kept: `src/core/sdk.ts` + `src/core/model-resolver.ts` + `src/main.ts` + `src/core/agent-session-runtime.ts` + `src/core/agent-session.ts` + `src/core/defaults.ts` + `src/config.ts` @ v0.84.4 — `/new` flow, default resolution order, thinking fallback chain (`medium`), `:level` parsing, trust gating.
- Dropped: `badlogic/pi-mono` raw GitHub URLs — 404 (repo moved to `earendil-works/pi`).
- Dropped: `docs/models.md` (local) — covers custom providers/`models.json` and `thinkingLevelMap` only; not needed for default selection beyond level vocabulary.
- Dropped: web search results — not needed; local docs + exact-version source were sufficient and primary.

## Gaps

- I did not run `pi` to observe the trust prompt or verify runtime behavior end-to-end (no shell tool available); conclusions about `/new` re-resolution are from reading the v0.84.4 source path `interactive-mode.ts → agent-session-runtime.ts → main.ts createRuntime → sdk.ts`, which is unambiguous.
- Minor: I did not trace every fallback branch for edge cases like `--session` from a different cwd combined with `/new` (the factory reuses the session's cwd); unlikely to affect the standard per-project setup.
- Suggested next step if desired: empirically confirm by creating `.pi/settings.json` in a scratch dir, running `pi`, pressing `/new`, and checking the footer model.
