---
paths:
  - 'WandEnhancer/Core/EnhancerConfig.cs'
  - 'WandEnhancer/Core/Js/**'
  - 'WandEnhancer/Patches/*.js'
  - 'scripts/test-patch-locators.ps1'
---

# Patch engine

- Patches are located structurally, not by shape. Anchor on something Wand does not rename between builds (an API endpoint, an IPC channel name, a public method name), then walk the delimiters with `Resolve` and targeted `slice`s. The actual JAR/ASAR content changes between releases, but the semantic anchor remains.
- No regex across a whole method body or bundle. Scope patterns to a located `JsFunction` via `Resolve`, so they run on a few hundred characters.
- A patch is one `PatchEntry` in `WandEnhancer/Core/EnhancerConfig.cs`; its `Locate` delegate returns the edits. Return `null` when the anchor is absent: that means "not my file". Throw only when the anchor is definitely in the wrong bundle, not just missing.
- Payload JS lives in `WandEnhancer/Patches/*.js`, embedded as `patches/<name>.js`, and is loaded with `PatchPayload.Load(name, "key", value, ...)`, which fills `${key}` placeholders in one pass. Keep shared payload terms in the same names.
- Edits from one patch apply highest offset first. Keep them non-overlapping.
- A patch that exists only on some builds sets `CapabilityHints`: an absent capability logs a skip, a detected but unpatchable one still fails the run.
- When Wand restructures something, add a fallback branch inside that patch's `Locate`, not a version table. The old branch keeps old builds working.
- Verify on real bundles, minified and prettified: locating must succeed on both and the patched files must pass `node --check`.

## Pro activation

- `EPatchType.ActivatePro` is independent of the remote panel and the bridge. It wraps the returned promise of `getUserAccount`, `setAccountWandBrandExperience` and `setAccountLanguage` so `subscription` is only used when the account is actually present.
- Pro is `!!account.subscription`; flags and `512` are irrelevant. `setAccountWandBrandExperience` is missing on some builds, so it is optional through `CapabilityHints`.

## DevTools on F12

- The `DevToolsOnF12` patch anchors on the main-process `<app>.whenReady().then(` and attaches a `before-input-event` hook to every `BrowserWindow.webContents`. Do not patch the renderer keydown listener or the browser window will no longer behave consistently.
