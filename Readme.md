# @shoutem/deep-diff

This lib is copied from (https://github.com/flitbit/diff , version 0.3.8) and modified to the Shoutem needs.

**deep-diff** is a javascript/node.js module providing utility functions for determining the structural differences between objects and includes some utilities for applying differences across objects.

## Features

- Get the structural differences between two objects.
- Observe the structural differences between two objects.
- When structural differences represent change, apply change from one object to another.
- When structural differences represent change, selectively apply change from one object to another.

## Installation

```
npm install deep-diff
```

## Tests

```bash
npm install
```

```bash
npm test
```

... or ...

```bash
mocha -R spec
```

### Importing

**nodejs**

```javascript
var deep = require("@shoutem/deep-diff");
```

## Simple Examples

In order to describe differences, change revolves around an `origin` object. For consistency, the `origin` object is always the operand on the `left-hand-side` of operations. The `comparand`, which may contain changes, is always on the `right-hand-side` of operations.

```javascript
var diff = require("deep-diff").diff;

var lhs = {
  name: "my object",
  description: "it's an object!",
  details: {
    it: "has",
    an: "array",
    with: ["a", "few", "elements"],
  },
};

var rhs = {
  name: "updated object",
  description: "it's an object!",
  details: {
    it: "has",
    an: "array",
    with: ["a", "few", "more", "elements", { than: "before" }],
  },
};

var differences = diff(lhs, rhs);
```

The code snippet above would result in the following structure describing the differences:

```javascript
// Versions < 0.2.0
[
  { kind: "E", path: ["name"], lhs: "my object", rhs: "updated object" },
  {
    kind: "A",
    path: ["details", "with"],
    index: 2,
    item: { kind: "E", path: [], lhs: "elements", rhs: "more" },
  },
  {
    kind: "A",
    path: ["details", "with"],
    index: 3,
    item: { kind: "N", rhs: "elements" },
  },
  {
    kind: "A",
    path: ["details", "with"],
    index: 4,
    item: { kind: "N", rhs: { than: "before" } },
  },
];
```

_v 0.2.0 and above_ The code snippet above would result in the following structure describing the differences:

```javascript
[
  { kind: "E", path: ["name"], lhs: "my object", rhs: "updated object" },
  { kind: "E", path: ["details", "with", 2], lhs: "elements", rhs: "more" },
  {
    kind: "A",
    path: ["details", "with"],
    index: 3,
    item: { kind: "N", rhs: "elements" },
  },
  {
    kind: "A",
    path: ["details", "with"],
    index: 4,
    item: { kind: "N", rhs: { than: "before" } },
  },
];
```

### Differences

Differences are reported as one or more change records. Change records have the following structure:

- `kind` - indicates the kind of change; will be one of the following:
  - `N` - indicates a newly added property/element
  - `D` - indicates a property/element was deleted
  - `E` - indicates a property/element was edited
  - `A` - indicates a change occurred within an array
- `path` - the property path (from the left-hand-side root)
- `lhs` - the value on the left-hand-side of the comparison (undefined if kind === 'N')
- `rhs` - the value on the right-hand-side of the comparison (undefined if kind === 'D')
- `index` - when kind === 'A', indicates the array index where the change occurred
- `item` - when kind === 'A', contains a nested change record indicating the change that occurred at the array index

Change records are generated for all structural differences between `origin` and `comparand`. The methods only consider an object's own properties and array elements; those inherited from an object's prototype chain are not considered.

Changes to arrays are recorded simplistically. We care most about the shape of the structure; therefore we don't take the time to determine if an object moved from one slot in the array to another. Instead, we only record the structural
differences. If the structural differences are applied from the `comparand` to the `origin` then the two objects will compare as "deep equal" using most `isEqual` implementations such as found in [lodash](https://github.com/bestiejs/lodash) or [underscore](http://underscorejs.org/).

### Changes

When two objects differ, you can observe the differences as they are calculated and selectively apply those changes to the origin object (left-hand-side).

```javascript
var observableDiff = require("@shoutem/deep-diff").observableDiff,
  applyChange = require("@shoutem/deep-diff").applyChange;

var lhs = {
  name: "my object",
  description: "it's an object!",
  details: {
    it: "has",
    an: "array",
    with: ["a", "few", "elements"],
  },
};

var rhs = {
  name: "updated object",
  description: "it's an object!",
  details: {
    it: "has",
    an: "array",
    with: ["a", "few", "more", "elements", { than: "before" }],
  },
};

observableDiff(lhs, rhs, function (d) {
  // Apply all changes except those to the 'name' property...
  if (d.path.length !== 1 || d.path.join(".") !== "name") {
    applyChange(lhs, rhs, d);
  }
});
```

## API Documentation

A standard import of `var diff = require('deep-diff')` is assumed in all of the code examples. The import results in an object having the following public properties:

- `diff(lhs, rhs, prefilter, acc)` &mdash; calculates the differences between two objects, optionally prefiltering elements for comparison, and optionally using the specified accumulator.
- `observableDiff(lhs, rhs, observer, prefilter)` &mdash; calculates the differences between two objects and reports each to an observer function, optionally, prefiltering elements for comparison.
- `applyDiff(target, source, filter)` &mdash; applies any structural differences from a source object to a target object, optionally filtering each difference.
- `applyChange(target, source, change)` &mdash; applies a single change record to a target object. NOTE: `source` is unused and may be removed.
- `revertChange(target, source, change)` reverts a single change record to a target object. NOTE: `source` is unused and may be removed.

### `diff`

The `diff` function calculates the difference between two objects.

**Arguments**

- `lhs` - the left-hand operand; the origin object.
- `rhs` - the right-hand operand; the object being compared structurally with the origin object.
- `prefilter` - an optional function that determines whether difference analysis should continue down the object graph.
- `acc` - an optional accumulator/array (requirement is that it have a `push` function). Each difference is pushed to the specified accumulator.

#### Pre-filtering Object Properties

The `prefilter`'s signature should be `function(path, key)` and it should return a truthy value for any `path`-`key` combination that should be filtered. If filtered, the difference analysis does no further analysis of on the identified object-property path.

## License

This library is released under the [MIT License](./LICENSE).
