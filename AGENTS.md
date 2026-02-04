# AGENTS.md - Coding Agent Instructions for pzprjs

## Project Overview

pzprjs is a JavaScript library for creating and editing pencil puzzles (Sudoku, Slitherlink, Yajilin, etc.). It runs in browsers (HTML5 Canvas) and Node.js. This is a pure ES5 JavaScript project with no TypeScript.

## Build/Lint/Test Commands

### Primary Commands

```bash
npm run build        # Lint and build to dist/
npm run test         # Run all Mocha tests
npm run lint         # Run ESLint on all source
npm run format       # Format code with Prettier
npm run check-format # Check formatting (CI uses this)
npm run coverage     # Run tests with coverage report
```

### Makefile Shortcuts

```bash
make             # Build the project
make test        # Run tests
make serve       # Serve dist/ on localhost:8000
make format      # Format code
```

### Running a Single Test

```bash
# Run a specific test file
npx mocha -r source-map-support/register test/variety/slither_test.js

# Run tests matching a pattern (grep)
npx mocha -r source-map-support/register --recursive test --grep "slither"

# Run a specific puzzle script test
npx mocha -r source-map-support/register test/script/slither.js
```

### CI Pipeline Order

The CI runs these steps in order:

1. `npm run check-format` - Formatting check
2. `npm run build` - Build (includes lint)
3. `npm run lint` - Full lint
4. `npm run coverage` - Tests with coverage

## Code Style Guidelines

### JavaScript Version

- **Source code (src/)**: ECMAScript 5 - no ES6+ features (no let/const, arrow functions, template literals, etc.)
- **Test code (test/)**: ECMAScript 6 allowed

### Formatting

- **Indentation**: Tabs (not spaces)
- **Line endings**: LF (Unix-style)
- **Semicolons**: Always required
- **Quotes**: Double quotes preferred
- **Trailing commas**: Never allowed (enforced by ESLint)
- **Final newline**: Always insert

### Imports and Module Pattern

This project uses a hybrid CommonJS/browser globals pattern, NOT ES6 modules:

```javascript
// Variety file pattern (src/variety/*.js)
(function(pidlist, classbase) {
  if (typeof module === "object" && module.exports) {
    module.exports = [pidlist, classbase];
  } else {
    pzpr.classmgr.makeCustom(pidlist, classbase);
  }
})(["puzzleid"], {
  /* class definitions */
});

// Core module pattern (src/puzzle/*.js)
(function() {
  // ... module code ...
})();
```

### Naming Conventions

| Type                 | Convention | Examples                                             |
| -------------------- | ---------- | ---------------------------------------------------- |
| Classes/Constructors | PascalCase | `Puzzle`, `Board`, `Cell`, `MouseEvent`              |
| Methods/Functions    | camelCase  | `initBoardSize()`, `setCanvas()`, `getMouseButton()` |
| Properties/Variables | camelCase  | `mouseCell`, `inputData`, `bordermode`               |
| Constants            | UPPER_CASE | `URL_PZPRV3`, `FILE_PZPR`, `MODE_EDITOR`             |
| Puzzle IDs (pid)     | lowercase  | `"slither"`, `"sudoku"`, `"yajilin"`                 |

### ESLint Rules (Enforced)

```javascript
// Required patterns
if (condition) { }      // Always use curly braces
x === y                 // Always use strict equality (===, !==)
new ClassName()         // Constructors must be PascalCase
var x = 1;              // No trailing commas after last item

// Allowed patterns
obj["property"]         // Bracket notation allowed
function() {}           // Functions in loops allowed
!!value                 // Double negation for boolean cast allowed
```

### Global Variables

- `pzpr` - Main library namespace (readonly)
- `ui` - UI namespace (readonly)

### Error Handling

- Use `throw Error("message")` for critical errors
- Use event emission for recoverable errors: `puzzle.emit("fail-open")`
- Minimal try-catch - only use for optional features

```javascript
// Error throwing pattern
if (!pzpr.variety(newpid).valid) {
  puzzle.emit("fail-open");
  throw Error("Invalid Puzzle Variety Selected");
}

// Silent catch for optional features
try {
  Object.freeze(this.nullobj);
} catch (e) {}
```

### Comments

- Japanese comments are common and acceptable
- Use dashed lines for section dividers
- Version numbers in file headers

```javascript
// File.js v3.6.0

//---------------------------------------------------------------------------
// ★ClassName  Brief description in Japanese
//---------------------------------------------------------------------------
```

## Project Architecture

### Directory Structure

```
src/                    # Main library source (ES5)
├── puzzle/             # Core classes (Puzzle, Board, Cell, etc.)
├── pzpr/               # Library core (classmgr, variety, parser)
├── variety/            # ~170 puzzle type implementations
└── variety-common/     # Shared variety code
src-ui/                 # Web UI (buttons, options, etc.)
test/                   # Test files (ES6 allowed)
├── puzzle/             # Core functionality tests
├── script/             # Puzzle type test scripts (one per puzzle)
└── variety/            # Variety-specific tests
dist/                   # Build output
```

### Class System

The project uses a custom prototype-based class system via `pzpr.classmgr`:

```javascript
// Use "ClassName:BaseClass" for inheritance
// Use "ClassName@puzzleid" for puzzle-specific overrides
pzpr.classmgr.makeCommon({
  ClassName: {
    property: value,
    method: function() {}
  }
});
```

### Adding a New Puzzle Type

1. Add entry in `src/pzpr/variety.js`
2. Implement in `src/variety/puzzlename.js`
3. Add test file `test/script/puzzlename.js`
4. Add to UI in `src-ui/js/v3index.js` and `src-ui/list.html`
5. Add icon `src-ui/img/puzzlename.png`

## Common Patterns

- Use `void 0` instead of `undefined`
- Method chaining: `return this;`
- Bitwise ops for integer math: `((id / qc) << 1) + 1`
- For loops over forEach for performance

## Testing Patterns

Tests use Mocha with Node's assert module:

```javascript
var assert = require("assert");
var pzpr = require("../../");

describe("Feature", function() {
  it("should do something", function() {
    var puzzle = new pzpr.Puzzle();
    puzzle.open("puzzleid/params");
    assert.equal(actual, expected);
  });
});
```

## Glossary (Japanese Terms in Code)

- **irowake**: Per-component coloring
- **peke**: Cross mark (X)
- **qnum**: Question number
- **anum**: Answer number
