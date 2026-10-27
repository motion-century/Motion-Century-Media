---
trigger: model_decision
name: codeplay-pr
description: If the user requests scripts/code for the CodePlay plugin in Premiere Pro, read this skill.
---

# Writing CEP Scripts for Premiere Pro (CodePlay)

## 1. Runtime Environment

Scripts are executed through `eval()` inside Adobe Premiere Pro's **ExtendScript (ECMAScript 3)** engine.

**Mandatory limitations:**

- **DO NOT** use ES6+ features: no `let`, `const`, arrow functions (`=>`), or Template Literals (`` ` ``).
- **ONLY USE** `var` for variable declarations.
- Prefer traditional `for` / `while` loops over newer Array methods.

## 2. CodePlay Console Polyfill

CodePlay already injects a polyfill — **use it instead of `alert()`:**

- `console.log(msg)` — Prints results/variables to the panel for the user to view.
- `console.error(msg)` — Prints errors in red (especially useful inside `catch` blocks).

## 3. Premiere Pro API Specifics

- **There is no global `app.beginUndoGroup()`** like in After Effects. Undo support depends on the specific API methods being used.
- **Distinguish between the two DOMs:**
  - `app.project` — Official DOM, preferred whenever possible.
  - `qe.project` — Hidden QE DOM, only use when there is no alternative through the official DOM.
- **Primary object entry points:**
  - `app.project.rootItem` — Contains bins and clips in the Project panel.
  - `app.project.sequences` / `app.project.activeSequence` — Used for timeline operations.
- Always check for `null` / `undefined` before interacting with objects (Premiere's API is strict about data types).

## 4. Standard Code Structure

```javascript
try {
  var proj = app.project;
  if (!proj) {
    throw new Error("Please open a Premiere project before running this script!");
  }

  var activeSeq = proj.activeSequence;
  if (activeSeq) {
    console.log("Active Sequence: " + activeSeq.name);

    var videoTracksCount = activeSeq.videoTracks.numTracks;
    console.log("Number of video tracks: " + videoTracksCount);

    // --- Main logic here ---
  } else {
    console.error("Error: No Sequence is currently open on the Timeline.");
  }
} catch (e) {
  console.error("Error: " + e.message);
}
```

## 5. Core Rules Summary

1. Use pure **ES3** syntax — do not use any ES6+ features.
2. Use `console.log()` / `console.error()` instead of `alert()`.
3. Always wrap code in `try...catch`.
4. Always check for `null/undefined` before interacting with Premiere objects.
5. Prefer `app.project` (official DOM); only use `qe.project` when absolutely necessary.
