<div align="center">

```
‎ ‎ ‎ ██████╗  █████╗ ████████╗ █████╗ ███████╗██╗  ██╗██╗███████╗████████╗
‎ ‎‎  ██╔══██╗██╔══██╗╚══██╔══╝██╔══██╗██╔════╝██║  ██║██║██╔════╝╚══██╔══╝
██║  ██║███████║   ██║   ███████║███████╗███████║██║█████╗     ██║
██║  ██║██╔══██║   ██║   ██╔══██║╚════██║██╔══██║██║██╔══╝     ██║
██████╔╝██║  ██║   ██║   ██║  ██║███████║██║  ██║██║██║        ██║
╚═════╝ ╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚═╝╚═╝        ╚═╝
```

**怠竜 Tearyū · Universal File Converter · Claude Code Skill Edition**

![Version](https://img.shields.io/badge/version-1.0.0-00fff7?style=flat-square)
![Python](https://img.shields.io/badge/python-3.8+-00fff7?style=flat-square&logo=python&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-ff153f?style=flat-square)
![Dependencies](https://img.shields.io/badge/dependencies-zero-00fff7?style=flat-square)
![Formats](https://img.shields.io/badge/formats-CSV_%C2%B7_JSON_%C2%B7_XML-3a486a?style=flat-square)
![Claude Code](https://img.shields.io/badge/Claude_Code-skill-712637?style=flat-square)
![Paths](https://img.shields.io/badge/conversion_paths-6-ff153f?style=flat-square)
[![Based on](https://img.shields.io/badge/based_on-datamorph-3a486a?style=flat-square)](https://github.com/The-Lazy-Dragon/datamorph)

</div>

---

*datamorph, but rebuilt as a single file and taught to Claude.*

---

It's a file converter. But neon. But with a Claude Code skill wired in.

**tearyu-datashift** is the Claude Code skill edition of [datamorph](https://github.com/The-Lazy-Dragon/datamorph) — same core problem (CSV ↔ JSON ↔ XML, zero dependencies), completely different form factor. Where datamorph is a proper Python package with a library API, pip install, and 56 tests, tearyu-datashift is one file and a skill. You drop it in `~/.claude/skills/` and Claude Code knows what to do with your data files automatically.

Same dragon. Different build.

---

## How it relates to datamorph

| | [datamorph](https://github.com/The-Lazy-Dragon/datamorph) | tearyu-datashift |
|---|---|---|
| Form factor | Python package (`src/` structure) | Single file |
| Install | `pip install -e .` | Copy one `.py` file |
| Library API | `convert()`, `convert_file()` | CLI only |
| Claude Code skill | ✗ | ✓ |
| Type inference | ✗ (all strings) | ✓ (int/float/bool/null) |
| Stdin/stdout pipes | ✓ | ✗ |
| Test suite | 56 tests | — |
| Live web editor | ✓ | ✗ |
| Python required | 3.10+ | 3.8+ |

**Use datamorph if** you want a proper library to import in your own code, `pip install`, or the web UI.

**Use tearyu-datashift if** you want Claude Code to handle your file conversions automatically, or you just want to drop one `.py` file anywhere and run it.

---

## Install as a Claude Code Skill

```bash
git clone https://github.com/The-Lazy-Dragon/tearyu-datashift \
  ~/.claude/skills/tearyu-datashift
```

Done. Ask Claude Code anything like:

> "Convert employees.csv to XML with a `<person>` element per row"
> "Turn this nested JSON into a flat CSV"
> "Convert data.xml to JSON and preserve all attributes"

It triggers the skill, runs `datashift.py`, and hands you the output.

---

## Use as a Standalone CLI

No install needed beyond Python 3.8+. Grab `datashift.py` and run it anywhere:

```bash
python datashift.py <input> -t <format> [options]
```

### All 6 conversion paths

```bash
python datashift.py data.csv  -t json
python datashift.py data.csv  -t xml
python datashift.py data.json -t csv
python datashift.py data.json -t xml
python datashift.py data.xml  -t json
python datashift.py data.xml  -t csv
```

### Options

| Flag | Default | Description |
|------|---------|-------------|
| `-f`, `--from` | auto | Force source format (`csv` / `json` / `xml`) |
| `-t`, `--to` | *(required)* | Target format |
| `-o`, `--output` | auto | Output file path |
| `--root TAG` | `root` | XML root element name |
| `--record TAG` | `record` | XML element name per row/item |
| `--indent N` | `2` | Indentation spaces in JSON/XML output |
| `--no-inference` | off | Keep CSV values as plain strings |
| `--flatten-sep SEP` | `.` | Separator for flattened nested keys |
| `-v`, `--verbose` | off | Print detailed stats |

---

## Examples

### CSV → JSON with type inference

```bash
python datashift.py employees.csv -t json -v
```
```
[OK] CSV → JSON  'employees.csv' → 'employees.json'
     rows: 5
     fields: ['id', 'name', 'age', 'active']
```

`"42"` → `42`, `"true"` → `true`, `""` → `null`. Pass `--no-inference` to keep everything as strings (same behavior as datamorph).

---

### CSV → XML with named elements

```bash
python datashift.py employees.csv -t xml --root employees --record employee
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<employees>
  <employee>
    <id>1</id>
    <name>Alice Chen</name>
    <active>true</active>
  </employee>
</employees>
```

---

### Nested JSON → flat CSV

```bash
python datashift.py nested.json -t csv --flatten-sep _
```
```csv
id,role_title,role_level,contact_address_city,skills_0,skills_1
1,Senior Engineer,L5,Taipei,Python,Rust
```

---

### XML → JSON (attributes preserved)

```xml
<person id="1" type="admin"><name>Alice</name></person>
```
```json
[{ "@id": "1", "@type": "admin", "name": "Alice" }]
```

`@attr` keys are round-trippable — pass them back through `json_to_xml` and they become attributes again.

---

## What gets preserved

| Source | Output behavior |
|--------|----------------|
| CSV headers | JSON keys / XML element names |
| CSV numeric values | Type-inferred as int / float (disable with `--no-inference`) |
| CSV `true`/`false` | Inferred as JSON booleans |
| JSON nested objects | Flattened `parent.child` keys in CSV |
| JSON arrays | Repeated XML sibling elements / indexed CSV keys |
| JSON `@attr` keys | XML attributes |
| XML attributes | `@attr` keys in JSON, `@attr` columns in CSV |
| XML same-tag siblings | JSON arrays (auto-detected) |
| UTF-8 characters | Preserved throughout |

---

## Error messages

```
[ERROR] 'data.json' is not valid JSON.
  → Expecting ',' delimiter: line 4, column 3.

[ERROR] 'legacy.csv' contains non-UTF-8 characters.
        Re-save the file as UTF-8 and try again.

[ERROR] XML root element has no children.
        DataShift maps each child element to one CSV row —
        there must be at least one.

[WARN]  Row(s) [3, 7] have a different column count than the header.
        Missing cells will be empty, extra cells will be ignored.
```

Warnings are non-fatal. Errors exit with code 1.

---

## Project structure

```
tearyu-datashift/
├── SKILL.md          Claude Code skill — install to ~/.claude/skills/
├── datashift.py      The converter — one file, no dependencies
├── CHANGELOG.md      Version history and roadmap
├── WARP.md           Quick reference card
├── requirements.txt  Documents stdlib modules used (no pip needed)
└── examples/
    ├── README.md     Walkthrough of all 6 conversion paths
    ├── sample.csv    Flat data, 5 records
    ├── sample.json   Same data as JSON array
    ├── sample.xml    Same data as XML
    ├── nested.json   Nested objects + arrays
    └── nested.xml    XML with attributes and deep nesting
```

---

## Requirements

- Python 3.8+
- No external packages — `csv`, `json`, `xml.etree.ElementTree`, `xml.dom.minidom`, `argparse`, `pathlib`, `re`, `sys`

---

## Also check out

| Project | What it is |
|---------|-----------|
| [**datamorph**](https://github.com/The-Lazy-Dragon/datamorph) | The original — full Python package, library API, pip install, 56 tests, live web editor |
| [**NavalStrike**](https://github.com/The-Lazy-Dragon/NavalStrike) | Battleship · 25 ships · 35 achievements · procedural audio · one `.html` |
| [**lazy-dragons-tictactoe**](https://github.com/The-Lazy-Dragon/lazy-dragons-tictactoe) | TicTacToe · Minimax AI · achievements · dev console · one `.html` |
| [**Tearyu-Deadshot-Crosshair**](https://github.com/The-Lazy-Dragon/Tearyu-Deadshot-Crosshair) | Chrome extension + Android alpha · crosshair for deadshot.io |
| [**tearyu-humanizer**](https://github.com/The-Lazy-Dragon/tearyu-humanizer) | Claude Code skill · strips AI texture from writing · v1.1 |

---

<div align="center">

Built with zero frameworks and questionable life choices — actually the same idea as datamorph, stripped to one file and bolted onto Claude Code.

**怠竜 · TEARYŪ — The Lazy Dragon**

</div>
