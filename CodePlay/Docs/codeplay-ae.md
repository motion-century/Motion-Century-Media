---
trigger: model_decision
name: codeplay-ae
description: If the user requests scripts/code for the CodePlay plugin in After Effects, read this skill.
---

# Writing CEP Scripts for After Effects (CodePlay)

## 1. Runtime Environment

Scripts are executed through `eval()` inside Adobe After Effects' **ExtendScript (ECMAScript 3)** engine via the CEP platform.

**Mandatory limitations:**

- **DO NOT** use ES6+ features: no `let`, `const`, arrow functions (`=>`), `Promise`, `async/await`, or Template Literals (`` ` ``).
- **ONLY USE** `var` for variable declarations.
- Avoid `.map()`, `.filter()`, and `.reduce()` because they may not exist in older versions of After Effects. Use traditional `for` loops instead.

## 2. CodePlay Console Polyfill

CodePlay already includes a polyfill — **use it instead of `alert()`:**

- `console.log(msg)` — Prints results/messages to the panel.
- `console.error(msg)` — Prints errors in red.

> Do not log circular objects (objects containing infinite reference loops), as stringifying them may freeze the UI.

## 3. Undo Group Rules

Any operation that modifies the project **must** be wrapped inside `app.beginUndoGroup()` / `app.endUndoGroup()` so users can undo the entire action with a single Ctrl+Z.

## 4. Standard Code Structure

```javascript
try {
  app.beginUndoGroup("Action Name");

  var proj = app.project;
  if (!proj) {
    throw new Error("No project is currently open!");
  }

  var activeItem = proj.activeItem;
  if (activeItem != null && activeItem instanceof CompItem) {
    console.log("Working on Comp: " + activeItem.name);

    // --- Main logic here ---
  } else {
    console.log("No Comp is open. Creating a new Comp...");
    var newComp = proj.items.addComp("CodePlay Comp", 1920, 1080, 1, 10, 30);
    console.log("Created Comp: " + newComp.name);
  }

  app.endUndoGroup();
} catch (e) {
  console.error("Error: " + e.message);
}
```

## 5. Core Rules Summary

1. Use pure **ES3** syntax — do not use any ES6+ features.
2. Use `console.log()` / `console.error()` instead of `alert()`.
3. Always wrap code in `try...catch`.
4. Always wrap project modifications in `app.beginUndoGroup()` and `app.endUndoGroup()`.

## 6. Shape Layer Quirk (Object is Invalid Error)

When adding properties to Shape Layers (e.g., Rect, Fill) via `addProperty`, the returned object might become detached or invalid when you try to modify it directly. Always retrieve the newly created property by its index from its parent before modifying its sub-properties.

**CORRECT APPROACH:**
```javascript
var shapeGroup = boxLayer.property("Contents").addProperty("ADBE Vector Group");
shapeGroup.property("Contents").addProperty("ADBE Vector Shape - Rect");

// MUST retrieve the property by index to avoid "Object is invalid" error:
var myRect = shapeGroup.property("Contents").property(1); 
myRect.property("ADBE Vector Rect Size").expression = "...";
```

Also, always use exact Match Names (e.g., `ADBE Vector Rect Size`, `ADBE Vector Rect Position`, `ADBE Vector Fill Color`).
