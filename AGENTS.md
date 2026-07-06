# AGENTS.md

## Commands

```bash
pip install -r requirements.txt   # Python 3.13+ required
python main.py [input_path] [output_path] [options]
python modules/mark2epub.py <md_dir> <epub_path>  # standalone EPUB-only mode
```

**No test suite, linter, formatter, or CI exists in this repo.**

## CLI flags

`main.py` accepts: `--max-pages N`, `--start-page N`, `--skip-epub`, `--skip-md`.

The README documents `--batch-multiplier` and `--langs` flags that do **not** exist in the code — do not trust those.

## Architecture

Two-stage pipeline: `modules/pdf2md.py` → `modules/mark2epub.py`.

| Stage | Module | Input | Output |
|---|---|---|---|
| PDF → Markdown | `pdf2md.py` | `.pdf` file | `{name}.md`, `{name}_metadata.json`, `images/` |
| Markdown → EPUB | `mark2epub.py` | dir with `.md` + `images/` | `{name}.epub` (ZIP) |

## Gotchas

- **marker-pdf 1.x requires an explicit page list.** `--start-page` without `--max-pages` will warn and still process all pages (no open-ended range).
- **Default input is `./input/`** (auto-created if missing). Default output goes next to the PDF in a directory named after its stem.
- **EPUB generation is interactive.** `mark2epub.py` prompts for title, author, etc. and offers to open the markdown for review — blocks non-interactive use unless a `description.json` already exists in the markdown dir.
- **Image lookup is case-insensitive** (`build_image_lookup()` lowercases filenames).
- **`pyproject.toml` is a stub** — `dependencies = []` is empty. Real dependencies live only in `requirements.txt`. If you add a dep, update `requirements.txt`.
- **Windows is the dev platform.** File-opening uses `start` on Windows vs `xdg-open` on Linux.
- **The README is stale** — dependency versions and CLI flags listed there are outdated. Trust the source code.
- **`modules/postprocessing/`** contains an LLM-driven framework (`template.py` + `prompt.txt`) for generating regex fix scripts to clean up OCR/conversion artifacts in the markdown output.

## Module notes

`mark2epub.py` supports both import (`convert_to_epub(dir, path)`) and standalone execution (`python modules/mark2epub.py <md_dir> <epub_path>`). Its internal `main()` expects raw sys.argv-like args.
