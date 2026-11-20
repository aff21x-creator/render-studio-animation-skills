# Render Studio Animation Generator — Agent Skill

A detailed, reusable Agent Skill for generating deterministic Canvas 2D animation code projects that import into Personal Animation Render Studio V1.

## Contents

- `SKILL.md` — activation metadata and required workflow.
- `references/` — renderer contract, project ZIP format, Canvas 2D patterns, determinism/performance and validation checklists.
- `examples/single-file/` — a complete standalone JavaScript animation example.
- `examples/multi-file/` — a complete multi-module project with relative ES module imports.
- `scripts/validate_project.py` — standard-library-only static project structure/import validator.

## Compatibility

This follows the open Agent Skills folder format. It is intended for AI clients that can discover and load Agent Skills. A model or application that does not support skills cannot be made to automatically install or follow a skill just by receiving this ZIP; on those platforms, provide the `SKILL.md` and relevant references as files/context manually.

The generated code targets the current Personal Animation Render Studio V1 contract. In particular, V1 supports project-local `.js`/`.mjs` relative ES module imports, not arbitrary npm packages, TypeScript/JSX, external imports or circular dependencies.

## Install

Copy the entire `render-studio-animation` directory into the skills directory configured by your AI client. The exact installation folder is client-specific; follow that client's skill installation instructions. Keep the directory name identical to the `name` field in `SKILL.md`.

## Validate an example

From this directory:

```bash
python scripts/validate_project.py examples/single-file
python scripts/validate_project.py examples/multi-file
```

For JavaScript syntax validation when Node.js is available:

```bash
find examples -name '*.js' -print0 | xargs -0 -n1 node --check
```

Static validation does not replace importing and rendering the project inside the Studio.
