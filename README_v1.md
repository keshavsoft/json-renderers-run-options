# json-renderers-run-options

> Pure declarative feature and option instruction runner for JSON renderers in the KeshavSoft ecosystem.

## Overview
Applies optional dynamic UI enhancements (serial numbering, row action buttons, search bars) to a base JSON DOM AST using pure single instructions defined in `spec-repo`.

## Usage
```javascript
import runOptions from "json-renderers-run-options";

const finalSpec = runOptions({
    base,
    options: { showSerial: true, showOptions: true, showSearch: true },
    optionDefs: definitions.table.simple.options,
    showLog: false
});
```
