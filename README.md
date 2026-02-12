# MoonBit Practice Plugin

A Claude Code plugin providing best practices for MoonBit code generation.

## Installation

```bash
claude plugin marketplace add mizchi/moonbit-practice
claude plugin install moonbit-practice@moonbit-practice
```

## Usage

```bash
/moonbit-practice
```

## Included References

- **ide.md**: Code navigation with `moon ide`
- **ffi.md**: MoonBit FFI reference
- **ffi-native.md**: Native C FFI patterns and ownership annotations
- **refactor.md**: Refactoring patterns
- **configuration.md**: moon.mod.json / moon.pkg configuration
- **language.md**: Language features (types, traits, pattern matching, access control)
- **testing.md**: Doc tests, snapshot tests, benchmarks, and QuickCheck
- **performance.md**: View types (StringView, ArrayView, BytesView) for zero-copy operations
- **stdlib.md**: MoonBit standard library (moonbitlang/core) usage

## Directory Structure

```
moonbit-practice/
├── .claude-plugin/
│   └── marketplace.json
├── skills/
│   └── moonbit-practice/
│       ├── SKILL.md
│       ├── reference/
│       │   ├── configuration.md
│       │   ├── ffi.md
│       │   ├── ffi-native.md
│       │   ├── language.md
│       │   ├── ide.md
│       │   ├── performance.md
│       │   ├── refactor.md
│       │   ├── stdlib.md
│       │   └── testing.md
│       └── assets/
│           └── ci.yaml
└── README.md
```

## License

MIT
