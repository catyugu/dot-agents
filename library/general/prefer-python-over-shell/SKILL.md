---
name: prefer-python-over-shell
description: Write a Python script instead of a native shell command whenever the job is more than one command with no data munging — quoting, globbing, text or CSV/JSON transforms, HTTP, in-place edits, parallel work, cross-platform paths, long Windows paths.
version: "1.0.0"
author: "catyugu"
license: "MIT"
---

# Prefer Python over shell

## When to use this skill

- When you wanna run any command that has many parameters.
- When you wanna run a series of command that can be prolonged in the terminal script language.
- When you wanna create scripts in convenience for some multiple-stage build or command execution pipeline.
- When `Python` is available on the machine.

## When NOT to use the skill

- When `Python` is unavailable on the machine.
- When there is only one command, one tool, no transformation: `git`, `uv`, `pnpm`, `cargo`, `make`, `docker`.
- When running throughput over tens of gigabytes: `rg` and `awk` stream, Python materializes. Measure before rewriting a working pipeline.
- When shell semantics are the point: job control, `set -euo pipefail`, exports in a real script.

## Why we prefer Python

- Heredocs and `-c` strings mangle escapes. `python - <<'EOF'` turns `\n` into a real newline, `printf > file` eats `%d`, and an argument carrying `\r` injects a real CR. Python writing bytes is exact, and byte-level verification afterwards is one function call away.
- Shells on Windows is disgusting to use.
- Paths above 260 characters need the `\\?\` prefix: `pathlib` handles them, `Remove-Item` and `cp -r` do not.
- Structured data has no shell representation. JSON, CSV and nested configs become `cut`/`jq`/`awk` chains that break on the first quoted comma.
- Pipes discard exit codes: only the last stage's status survives, and no stage carries a message. `subprocess.run(check=True)` raises with the command and its output.
- Python code remains consistent for different platforms.

## Possible replacements

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

## Notice

- Do not try installing any dependencies globally using `uv` or `pip` or so.
- Never pass `\r`, `%d` or `\n` through a shell argument or a tool that rewrites files.
- Do not rebuild a 20-line script into a CLI framework; one subcommand per real use.
- Do not hide a failure by letting the script exit 0 after printing an error.
