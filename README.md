# json-renderers-run-options

> **Pure declarative feature and option instruction runner for JSON-to-DOM renderers in the KeshavSoft ecosystem.**

---

[![LIVE DOCUMENTATION](https://img.shields.io/badge/LIVE-DOCUMENTATION-blue?style=for-the-badge)](https://keshavsoft.github.io/json-renderers-run-options/)
[![NPM](https://img.shields.io/badge/NPM-PACKAGE-CB3837?style=for-the-badge&logo=npm)](https://www.npmjs.com/package/json-renderers-run-options)
[![GITHUB](https://img.shields.io/badge/GITHUB-REPOSITORY-181717?style=for-the-badge&logo=github)](https://github.com/keshavsoft/json-renderers-run-options)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

---

Part of the **KeshavSoft Declarative UI Architecture** (Feature & Optional AST Enhancer).

## 📖 What is the Purpose of this Repo?

In imperative UI architectures, enabling optional features on components (such as a **Search Bar**, a **Serial Number (#)** column, or **Row Action Buttons** in a data table) requires writing conditional `if/else` logic, DOM loops, and manual element insertions.

`json-renderers-run-options` completely eliminates this complexity by providing a **pure declarative option engine**:

1. **Zero Imperative Code:** You declare simple option toggles like `{ showSearch: true, showSerial: true, showOptions: true }`.
2. **Declarative Single Instructions:** It uses single instructions from `spec-repo` / `declarative-definitions` and dispatches them via `json-instruction-single`.
3. **Safe Target Detection (V2):** Before applying any instruction, it evaluates the target destination with `jsonpathly` (`query(base, destinationPath.jsonPath).length > 0`). If the mount point does not exist in the base AST, it safely skips it without throwing errors.
4. **Decoupled Architecture:** Keeps the base component AST minimal and clean, while allowing optional features to be dynamically plugged in at runtime.

---

## ⚙️ How It Works (V2 Architecture)

```
┌────────────────────────────────┐
│   Base AST (e.g. Table Spec)   │
└──────────────┬─────────────────┘
               │
               ▼
┌────────────────────────────────────────────────────────┐
│                   runOptions(config)                   │
│                                                        │
│  1. Check feature flags: showSearch, serial, options   │
│  2. Filter instructions via JSONPath (jsonpathly)       │
│  3. Execute single instructions via                    │
│     json-instruction-single                            │
└──────────────┬─────────────────────────────────────────┘
               │
               ▼
┌────────────────────────────────┐
│   Enhanced Final DOM Spec      │
│   (with Search, Serial, etc.)  │
└────────────────────────────────┘
```

---

## 🎛️ Supported Options

| Option Key        | Aliases                  | Description                                                                              | Target Injection                              |
| :---------------- | :----------------------- | :--------------------------------------------------------------------------------------- | :-------------------------------------------- |
| **Search**        | `showSearch`             | Conditionally mounts search frames, search labels, helper text, and search input fields. | Base table wrapper / header container         |
| **Serial Number** | `showSerial`, `serial`   | Adds a serial number (`#`) column header and corresponding row counter cells.            | `<thead>` header row & `<tbody>` rows         |
| **Row Options**   | `showOptions`, `options` | Adds action columns containing buttons (e.g., View, Edit, Delete).                       | `<thead>` header row & `<tbody>` action cells |

---

## 📦 Installation

```bash
npm install json-renderers-run-options
```

### Dependencies

- `json-instruction-single` – Executes atomic JSON transformation instructions.
- `jsonpathly` – Evaluates JSONPath queries on the AST.
- `spec-repo` – Provides declarative specifications and templates.

---

## 🚀 Usage

### 1. Default Import (Latest - V2)

```javascript
import runOptions from "json-renderers-run-options";
// or: import { runOptions } from "json-renderers-run-options";

const baseTableAst = {
  tag: "table",
  attributes: { "data-role": "tableRoot" },
  children: [
    { tag: "thead", children: [{ tag: "tr", children: [] }] },
    { tag: "tbody", children: [] },
  ],
};

const finalAst = runOptions({
  base: baseTableAst,
  options: {
    showSearch: true,
    showSerial: true,
    showOptions: true,
  },
  optionDefs: {
    search: [
      /* single instruction objects with destinationPath */
    ],
    serial: [
      /* single instruction objects with destinationPath */
    ],
    options: [
      /* single instruction objects with destinationPath */
    ],
  },
  showLog: false, // Set to true to trace target instructions in console
});
```

### 2. Integration with `declarative-definitions`

```javascript
import runOptions from "json-renderers-run-options";
import definitions from "declarative-definitions";
import { spec } from "spec-repo";

const base = structuredClone(spec.tableSimple);

const enhancedTable = runOptions({
  base,
  options: {
    showSearch: true,
    showSerial: true,
    showOptions: true,
  },
  optionDefs: definitions.table.simple.options,
  showLog: false,
});
```

### 3. Browser CDN Import via ESM

```html
<script type="module">
  import runOptions from "https://cdn.jsdelivr.net/npm/json-renderers-run-options@1.2.1/src/index.js";

  const finalSpec = runOptions({
    base: myAst,
    options: { showSearch: true, showSerial: true },
  });
</script>
```

### 4. Explicit Version Imports

```javascript
// Import Latest V2 (Default)
import { runOptions } from "json-renderers-run-options/src/v2/index.js";

// Import Legacy V1
import { runOptions as runOptionsV1 } from "json-renderers-run-options/src/v1/index.js";
```

---

## 🏛️ Instruction Target Resolution: V1 vs V2

| Feature              | Version 1 (Legacy)                                     | Version 2 (Latest - Recommended)                                 |
| :------------------- | :----------------------------------------------------- | :--------------------------------------------------------------- |
| **Target Mechanism** | `destinationPath.dataRole` attribute match             | `destinationPath.jsonPath` expression evaluation                 |
| **Target Finder**    | Recursive loop checking `node.attributes['data-role']` | Standard JSONPath query via `jsonpathly`                         |
| **Flexibility**      | Restricted only to elements with explicit `data-role`  | Supports any JSONPath selector (tags, classes, roles, positions) |
| **Safety**           | Returns false if tree lacks exact string match         | Returns matches count `query(base, jsonPath).length > 0`         |

---

## 📋 Parameters (`runOptions`)

| Parameter    | Type      | Default    | Description                                                                                   |
| :----------- | :-------- | :--------- | :-------------------------------------------------------------------------------------------- |
| `base`       | `Object`  | _required_ | The base JSON DOM AST to be modified.                                                         |
| `options`    | `Object`  | `{}`       | Flags enabling features (`showSearch`, `showSerial` / `serial`, `showOptions` / `options`).   |
| `optionDefs` | `Object`  | `{}`       | Instruction definitions containing target instructions for `search`, `serial`, and `options`. |
| `showLog`    | `Boolean` | `false`    | When `true`, logs debug information and resolved targets to the console.                      |

---

## 📄 License

MIT © [KeshavSoft](https://github.com/keshavsoft)
