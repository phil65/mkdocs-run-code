# AGENTS.md

## Overview

`mkdocs-run-code` is a small browser-side TypeScript bundle for MkDocs (Material) sites. It
turns fenced Python code blocks marked `{ .interactive }` into editable, runnable blocks:
clicking "run" swaps the highlighted code for a CodeMirror editor and executes it in the
browser with [Pyodide](https://pyodide.org). There is no Python package and no MkDocs
plugin; sites include the built `run_code.js` via `extra_javascript`.
Fork of `samuelcolvin/mkdocs-run-code` (remote `upstream`).

## Architecture

- `src/main.ts`: entry point. Loads `run_code.css` from next to the script tag whose `src`
  contains `run_code.js` (`getUrl`), finds `.interactive.highlight` blocks that look like
  Python and wraps each in a `CodeBlock` (play/reset buttons, CodeMirror editor, output panel,
  ANSI output via `ansi-to-html`, Dracula theme on dark backgrounds). Only one block is active
  at a time. Reads `window.mkdocs_run_deps` for packages to install.
- `src/run_python.ts`: lazy, shared Pyodide instance (`load`), installs dependencies via
  micropip (special case: a pinned `pydantic-core==X` is fetched from the pydantic-core GitHub
  release through a proxy), runs code with `runPythonAsync`, buffers stdout/stderr and
  reformats tracebacks.
- `src/pyodide.ts`: Pyodide CDN URL/version (`downloadPyodide`), minimal type declarations,
  and TTY redirection of stdin/stdout/stderr (`preparePyodide`).
- `src/run_code.css`: button/editor/output styles; esbuild emits it as `dist/run_code.css`.
- `docs/index.md` + `mkdocs.yml`: local example site; `docs/dist` is a symlink to `../dist`.

Flow: page load -> `main()` -> click play -> `CodeBlock.run()` -> `runCode()` ->
`load()` (first time only) -> output streamed to `CodeBlock.onMessageMethod`.

## Where to look

| Task | File |
| --- | --- |
| Which blocks become runnable | `src/main.ts` (`main`) |
| Editor / buttons / output UI | `src/main.ts` (`CodeBlock`), `src/run_code.css` |
| Pyodide version | `src/pyodide.ts` (`downloadPyodide`) |
| Dependency installation | `src/run_python.ts` (`load`) |
| Traceback formatting | `src/run_python.ts` (`reformat_exception`) |
| Build / lint config | `package.json`, `tsconfig.json` |
| CI | `.github/workflows/build.yml` |

## Commands

```bash
npm ci                 # install
npm run lint           # tsc (type-check only) + eslint + prettier --check
npm run format         # prettier --write
npm run build          # esbuild -> dist/run_code.js + dist/run_code.css (+ maps)
npm run update-pages   # push dist/ to gh-pages (published at phil65.github.io/mkdocs-run-code/)
uvx --with mkdocs-material mkdocs serve   # example site, after npm run build
```

There are no automated tests (`npm test` is a placeholder). To verify a change, build, serve
the example site and click a run button.

## Conventions

- Prettier: single quotes, no semicolons, trailing commas, 80 columns. ESLint config lives in
  `package.json` (`eslintConfig`, eslint 8 / typescript-eslint 6, legacy config format).
- All packages are listed under `dependencies` (no devDependencies split); keep it that way
  unless restructuring deliberately.
- `tsconfig.json` has `noEmit`; esbuild does the bundling.

## Pitfalls

- The output file name matters: `getUrl` locates the script by `run_code.js` and loads
  `run_code.css` next to it. Renaming the bundle requires updating `src/main.ts`.
- `dist/` is gitignored; the published bundle lives on the `gh-pages` branch (deployed with
  `npm run update-pages`, not by CI). Consumers (e.g. agentpool's docs) load
  `https://phil65.github.io/mkdocs-run-code/run_code.js`, so pushing to `gh-pages` is a release.
- Pyodide is loaded from the jsDelivr CDN; the Pyodide version determines the Python version
  and which packages (e.g. pydantic) are available prebuilt.
- Blocks need `pymdownx.superfences` + `pymdownx.highlight` so the `.interactive` class lands
  on the `div.highlight`.
