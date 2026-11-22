# mkdocs-run-code

Make Python code blocks in MkDocs (Material) sites editable and runnable in the browser.

Blocks marked `{ .interactive }` get a run button. Clicking it turns the block into a
CodeMirror editor and runs the code with [Pyodide](https://pyodide.org) (Python 3.13,
Pyodide 0.29). Output, including tracebacks, is shown below the block. Nothing runs on a
server: everything happens in the reader's browser.

Fork of [samuelcolvin/mkdocs-run-code](https://github.com/samuelcolvin/mkdocs-run-code),
originally written for [docs.pydantic.dev](https://docs.pydantic.dev).

## Usage

Add the script and the required Markdown extensions to `mkdocs.yml`:

```yaml
extra_javascript:
  - https://phil65.github.io/mkdocs-run-code/run_code.js

markdown_extensions:
  - pymdownx.highlight:
      pygments_lang_class: true
  - pymdownx.superfences
```

Mark Python blocks as interactive:

````markdown
```python { .interactive }
print("Hello from Pyodide")
```
````

Attributes can be combined, e.g. `{ .interactive title="Example" }`. Blocks without
`.interactive` are left alone.

### Installing packages

Set `window.mkdocs_run_deps` to a list of requirements before the code runs, either in a
script loaded via `extra_javascript` or inline in a page:

```html
<script>
  window.mkdocs_run_deps = ["pydantic"]
</script>
```

Packages are installed with `micropip` the first time any block runs. Packages that Pyodide
ships prebuilt (such as `pydantic`) and pure-Python wheels from PyPI work. A pinned
`pydantic-core==X.Y.Z` is fetched from the pydantic-core GitHub release instead, which only
works if that release has a wheel for the current Pyodide platform.

## Development

```bash
npm ci
npm run lint    # tsc + eslint + prettier
npm run build   # writes dist/run_code.js and dist/run_code.css
uvx --with mkdocs-material mkdocs serve   # example site in docs/, uses the local build
```

`npm run update-pages` publishes `dist/` to the `gh-pages` branch, which is what sites load
from `https://phil65.github.io/mkdocs-run-code/run_code.js`.

## License

MIT
