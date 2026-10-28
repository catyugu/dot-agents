---
name: prefer-python-over-shell
description: Write a Python script instead of a native shell command whenever the job is more than one command with no data munging — quoting, globbing, text or CSV/JSON transforms, HTTP, in-place edits, parallel work, cross-platform paths, long Windows paths. Use when about to write sed/awk/grep/find/xargs/jq/curl pipelines, a multi-line bash script, or a heredoc, when an argument needs escaping, or when the same task must run on both Windows and Unix. Not a licence to reimplement git, compilers, formatters or other tools that are their own interface.
category: general
version: "1.0"
license: "MIT"
---

# Prefer Python over shell

The habit: when a shell task stops being one command with no data munging, write a Python file and run the file. The shell is for invoking a tool. It is the wrong instrument for transforming that tool's output.

## Why this matters here

Shell failure modes are silent corruption, not clean errors:

- Heredocs and `-c` strings mangle escapes. `python - <<'EOF'` turns `\n` into a real newline, `printf > file` eats `%d`, and an argument carrying `\r` injects a real CR. Python writing bytes is exact, and byte-level verification afterwards is one function call away.
- Two quoting systems meet on Windows (PowerShell and MSYS bash). `//PID` is rewritten, `$` expands where it should not, and every nested quote is a bug waiting for a second shell.
- Paths above 260 characters need the `\\?\` prefix: `pathlib` handles them, `Remove-Item` and `cp -r` do not.
- Structured data has no shell representation. JSON, CSV and nested configs become `cut`/`jq`/`awk` chains that break on the first quoted comma.
- Pipes discard exit codes: only the last stage's status survives, and no stage carries a message. `subprocess.run(check=True)` raises with the command and its output.
- A script in the repo is reviewable, testable and reruns identically. A one-liner in a chat transcript is gone.

## Where the shell still wins

- One command, one tool, no transformation: `git`, `uv`, `pnpm`, `cargo`, `make`, `docker`.
- The tool *is* the interface: compilers, formatters, `ffmpeg`, `rg`. Call them from Python; do not replace them.
- Throughput over tens of gigabytes: `rg` and `awk` stream, Python materializes. Measure before rewriting a working pipeline.
- Shell semantics are the point: job control, `set -euo pipefail`, exports in a real script.

The line: the shell **invokes**, Python **transforms**. When both are needed, Python calls the tool through `subprocess`.

## How to run it

Never inline multi-line code (`python -c`, heredoc, `bash -c "..."`): the quoting is the bug. Write the file, then run the file.

One-off, no project around it — `uv run --script` with pinned PEP 723 metadata:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = ["httpx==0.28.1"]
# ///
```

Durable or shared — put it under the repo's `scripts/` and run it in the project environment (`uv run --project python --locked python scripts/task.py`, or `pnpm exec` for JS). Do not `pip install` to make a script work.

## Replacements

| Task | Shell | Python |
| --- | --- | --- |
| list files | `find . -name "*.py"` | `Path().rglob("*.py")` |
| filter by content | `grep -rn TODO src` | `[p for p in Path("src").rglob("*") if "TODO" in p.read_text("utf-8")]` |
| in-place edit | `sed -i s/a/b/g f` | `p.write_text(re.sub("a", "b", p.read_text("utf-8")), encoding="utf-8")` |
| one CSV column | `cut -d, -f2` | `[r["col"] for r in csv.DictReader(f)]` |
| pick from JSON | `jq .a.b` | `json.loads(text)["a"]["b"]` |
| GET a URL | `curl -s URL` | `urllib.request.urlopen(URL).read()` |
| save a download | `curl -o out URL` | `shutil.copyfileobj(resp, out.open("wb"))` |
| parse HTML | `sed` on tags | `html.parser` / `selectolax` |
| parallel jobs | `xargs -P8 -n1 cmd` | `ThreadPoolExecutor(8).map(fn, items)` |
| recursive copy, delete | `cp -r`, `rm -rf` | `shutil.copytree`, `shutil.rmtree` |
| total directory size | `du -sh` | `sum(p.stat().st_size for p in root.rglob("*"))` |
| diff two files | `diff -u a b` | `difflib.unified_diff` |
| unzip, untar | `unzip x.zip` | `zipfile.ZipFile(...).extractall()` |
| env var for a child | `FOO=1 cmd` | `subprocess.run([...], env={**os.environ, "FOO": "1"})` |

## Shape of the script

```python
import argparse
import sys
from pathlib import Path


def main() -> int:
    parser = argparse.ArgumentParser()
    parser.add_argument("root", type=Path)
    args = parser.parse_args()

    for path in args.root.rglob("*.md"):
        text = path.read_text(encoding="utf-8")
        if "\r" in text:
            print(f"CRLF: {path}", file=sys.stderr)
            return 1
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- `argparse` for anything with more than one input; undocumented flags are how scripts rot.
- `pathlib.Path` for every path, `encoding="utf-8"` for every text read and write. The Windows default is cp936 and will corrupt non-ASCII content.
- `subprocess.run(argv_list, check=True, capture_output=True, text=True)`. Never `shell=True`, never a command string.
- Data to `stdout`, diagnostics to `stderr`, an exit code from `main`. No `print` in library code.
- A script that rewrites many files defaults to printing the diff and applies only under `--apply`.

## Traps

- Never pass `\r`, `%d` or `\n` through a shell argument or a tool that rewrites files.
- Do not rebuild a 20-line script into a CLI framework; one subcommand per real use.
- Do not reimplement what one `rg` call does; call `rg`.
- Do not hide a failure by letting the script exit 0 after printing an error.
