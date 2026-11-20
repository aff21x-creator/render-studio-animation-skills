---
name: render-studio-animation
description: Generate complete, import-ready JavaScript animation projects for Personal Animation Render Studio V1. Use when a user asks for an animation, animated video source code, Canvas 2D scene, single-file animation, multi-file animation project, or code to import into this render studio. Produces deterministic render(ctx, time, frame) code and correctly structured project files or ZIP packages.
compatibility: Designed for Agent Skills-compatible AI clients. Target runtime is Personal Animation Render Studio V1 with native Canvas 2D and local JavaScript ES modules.
metadata:
  version: "1.0.0"
  target: "personal-animation-render-studio-v1"
  output: "javascript-canvas2d-project"
---

# Render Studio Animation Generator

## Mission

Turn a user's animation idea into a complete, visually intentional, deterministic Canvas 2D animation project that can be imported into **Personal Animation Render Studio V1** and rendered or exported by that application.

This is a code-generation skill, not a prompt-writing skill and not a general animation overview. When activated, do the animation design and engineering work: plan the visual sequence, write all required source files, validate the project structure and deliver the actual files when the environment supports file creation.

The target application owns playback, the timeline, frame scheduling and video encoding. Your code defines the visual result for a requested timestamp.

## Activation

Use this skill when the user asks to:

- create an animation or animated video using the Personal Animation Render Studio;
- generate a single JavaScript animation file;
- create a multi-file animation project;
- produce detailed Canvas 2D animation source code;
- modify, expand, optimize or debug a project intended for this renderer.

Do not use this skill to build or modify the Studio application itself, its UI, its export engine, or unrelated websites/apps.

## Output mode selection

Follow the user's explicit choice first:

- **Single file**: deliver one self-contained `main.js`-style JavaScript file. No imports, no sibling files and no external dependencies.
- **Multi-file**: deliver a complete project with `project.json`, root `main.js`, all imported local modules, and `README.md`, preferably as a ZIP whose contents start at the project root.
- **No format specified**: choose single-file for a compact scene with a small amount of logic. Choose multi-file when the user asks for a highly detailed animation, multiple distinct subsystems, reusable components, or a larger sequence. State the choice briefly; do not ask a question when the decision is straightforward.

If the user asks for both, provide both as separate deliverables. Do not silently change the requested format.

## Defaults

Use these defaults only when the user has not supplied values:

| Setting | Default |
|---|---:|
| Width | 1280 px |
| Height | 720 px |
| FPS | 30 |
| Duration | 10 seconds |
| Renderer | Canvas 2D |
| Entry file | `main.js` |
| Module extension | `.js` |

Choose another FPS when the requested motion or user preference calls for it. 24, 25, 30 and 60 FPS are valid project choices. Do not claim that the user's computer will sustain the configured preview FPS. The preview's measured performance depends on scene complexity, browser and hardware.

## Required engineering contract

Every valid project must follow the renderer contract exactly:

```js
export const config = {
  width: 1280,
  height: 720,
  fps: 30,
  duration: 10
};

export function render(ctx, time, frame) {
  // Draw the complete visual state for this exact timestamp.
}
```

The entry module must export both:

- `config`: an object with numeric `width`, `height`, `fps`, and `duration`;
- `render(ctx, time, frame)`: a function that draws one complete frame.

The function arguments are:

- `ctx`: native `CanvasRenderingContext2D` provided by the Studio;
- `time`: requested animation time in seconds;
- `frame`: zero-based frame index.

The Studio invokes this function. Never create a playback loop inside animation code.

For detailed engine constraints, read [references/engine-contract.md](references/engine-contract.md). For project ZIP rules, read [references/project-format.md](references/project-format.md).

## Non-negotiable rendering rules

1. **Deterministic random access.** The same `(time, frame)` input must produce the same image every time. Seeking directly to 8 seconds must not depend on having rendered seconds 0–7 first.
2. **No accumulated simulation state.** Do not advance positions by mutating values from the previous frame. Derive positions from `time`, `frame`, and stable object parameters.
3. **No playback/timer APIs.** Do not use `requestAnimationFrame`, `setTimeout`, `setInterval`, `performance.now`, or `Date.now` to drive animation.
4. **No external module dependencies.** Do not import CDN URLs, npm packages, node built-ins, remote code or browser modules. The V1 runtime resolves project-local relative JavaScript modules only.
5. **No DOM or application access.** Do not use `document`, `window`, React, UI component state, browser storage, `fetch`, network access, audio, or app-internal objects. Draw through `ctx` only.
6. **Complete frame redraw.** Paint the full canvas background every call. Do not assume the previous frame remains as a valid base image.
7. **State hygiene.** Use `ctx.save()` and `ctx.restore()` around transforms, clipping, compositing and alpha changes. Restore state even when helper code is complex. Reset the transform before full-canvas drawing.
8. **Correct configuration.** Use the exported `config` values throughout the project. Do not hard-code a second conflicting resolution, FPS or duration.
9. **Complete source.** Never leave TODOs, pseudocode, missing functions, unresolved imports or placeholder drawing calls in the delivered project.
10. **Honest validation.** Distinguish syntax/static checks from actual execution inside the user's Studio. Never claim a render or video export was tested unless it actually was.

## Workflow: idea to deliverable

### Step 1 — Understand the requested video

Extract or infer:

- topic and subject;
- visual style (cinematic, educational, infographic, abstract, documentary-like, minimal, etc.);
- intended audience and tone, where relevant;
- output resolution, FPS and duration;
- single-file or multi-file preference;
- any required text, labels, sequence or visual details.

Ask only when an omitted detail materially changes the result. Otherwise use the defaults and state any major assumption briefly.

### Step 2 — Design the scene before coding

Create a compact internal scene plan:

- overall composition and visual hierarchy;
- background and environment;
- main subject(s);
- secondary elements and atmospheric detail;
- motion behaviour;
- lighting, shadows, depth cues and colour palette;
- timeline beats and transitions;
- final hold or ending.

For a multi-scene animation, define timestamp ranges. Avoid arbitrary motion: every movement should support the subject or story.

### Step 3 — Choose project architecture

For a single-file result, put all constants, helpers and rendering logic in `main.js`.

For a multi-file result, use a clear dependency direction. Recommended pattern:

```text
project.json
main.js                 # config + required render() entry
README.md
src/
  scene.js               # scene orchestration
  background.js          # background/environment
  subjects.js            # primary subject(s)
  effects.js             # particles, light, atmosphere
  utils.js               # deterministic math helpers
```

Do not create files merely to inflate the project. A module should have one clear responsibility. Keep the dependency graph acyclic; do not create circular imports.

### Step 4 — Generate all files

Generate the complete source for every file. For multi-file projects:

- put `project.json` and `main.js` at the ZIP root;
- use explicit relative imports such as `./src/scene.js`;
- ensure every imported module exists in the archive;
- ensure the entry file exports `config` and `render`;
- keep `project.json.config` consistent with the entry file's exported `config`;
- include a short `README.md` with project summary, file roles and settings.

Never wrap the files in an additional enclosing folder inside the ZIP. The Studio's ZIP importer expects `project.json` and `main.js` at the archive root.

### Step 5 — Validate before delivery

Read [references/validation-checklist.md](references/validation-checklist.md). If available, run:

```bash
python scripts/validate_project.py path/to/project-or-zip
```

Also use a JavaScript syntax checker (for example `node --check`) when Node.js is available. Static validation is not a substitute for running the project in the Studio.

### Step 6 — Deliver correctly

When file creation is available:

- create the actual project directory and source files;
- for multi-file projects, create a ZIP with root-level `project.json` and `main.js`;
- verify the exact output path exists before linking it;
- optionally provide the folder or individual entry file as an additional convenience.

When file creation is not available:

- provide every complete source file in clearly labelled code blocks;
- preserve exact paths and filenames;
- do not say a ZIP or download exists when it does not.

In the final response, state the project type, resolution, FPS, duration, file count, validation performed and any untested limitations. Keep the explanation short; the code files are the main deliverable.

## Quality bar

The result should feel like a deliberately designed short animation, not a random collection of shapes. Use a coherent visual language, readable composition, controlled motion, meaningful timing and a clear ending. Detailed does not mean needlessly expensive: prioritize the most visually valuable details and keep rendering efficient.

For code patterns, read [references/canvas2d-cookbook.md](references/canvas2d-cookbook.md). For deterministic motion and performance, read [references/determinism-performance.md](references/determinism-performance.md). For complete examples, see `examples/single-file/` and `examples/multi-file/`.
