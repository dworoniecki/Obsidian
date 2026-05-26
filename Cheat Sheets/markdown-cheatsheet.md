# Markdown Cheatsheet

A practical reference for Markdown syntax, including GitHub Flavored Markdown (GFM) extensions.

---

## Common Workflows

### Documentation Header
```markdown
# Project Name

> Brief one-line description of the project.

## Overview

Short paragraph explaining what this is and why it exists.

## Quick Start

```bash
pip install mypackage
```
```

### Collapsible Section
```markdown
<details>
<summary>Click to expand</summary>

Hidden content goes here. Can include **any** markdown.

- Lists work
- Code works

</details>
```

### Badges (README Style)
```markdown
![Python](https://img.shields.io/badge/python-3.11-blue)
![License](https://img.shields.io/badge/license-MIT-green)
[![Tests](https://github.com/user/repo/actions/workflows/test.yml/badge.svg)](link)
```

### Nested List with Code
```markdown
1. Install dependencies
   ```bash
   pip install -r requirements.txt
   ```
2. Configure environment
   - Copy `.env.example` to `.env`
   - Fill in required values
3. Run the application
```

### Definition List (Not standard, but common)
```markdown
**Term**
: Definition of the term

**Another Term**
: Another definition
```

### Table of Contents
```markdown
## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Usage](#usage)
  - [Basic Usage](#basic-usage)
  - [Advanced Usage](#advanced-usage)
- [Contributing](#contributing)
```

### Linking to Headings
```markdown
<!-- Heading rules: lowercase, spaces → hyphens, remove punctuation -->
## My Section Title  →  [link](#my-section-title)
## Setup & Config   →  [link](#setup--config)
## Step 1: Install  →  [link](#step-1-install)
```

### Math (GitHub and some renderers)
```markdown
Inline math: $E = mc^2$

Block math:
$$
\sum_{i=1}^{n} x_i = \bar{x} \cdot n
$$
```

### Diff Highlighting
```markdown
```diff
- old line (shown in red)
+ new line (shown in green)
  unchanged line
```
```

---

## Headers

| Syntax | Description | Example | Rendered As |
|--------|-------------|---------|-------------|
| `# H1` | Heading level 1 | `# Project Title` | Largest heading |
| `## H2` | Heading level 2 | `## Overview` | Section heading |
| `### H3` | Heading level 3 | `### Setup Steps` | Subsection heading |
| `#### H4` | Heading level 4 | `#### Details` | Smaller subsection |
| `##### H5` | Heading level 5 | `##### Notes` | Minor heading |
| `###### H6` | Heading level 6 | `###### Fine print` | Smallest heading |
| `===` underline | Alternate H1 (Setext) | `Title\n===` | Same as `# H1` |
| `---` underline | Alternate H2 (Setext) | `Section\n---` | Same as `## H2` |

---

## Text Formatting

| Syntax | Description | Example | Output |
|--------|-------------|---------|--------|
| `**text**` | Bold | `**important**` | **important** |
| `__text__` | Bold (alternate) | `__important__` | **important** |
| `*text*` | Italic | `*emphasis*` | *emphasis* |
| `_text_` | Italic (alternate) | `_emphasis_` | *emphasis* |
| `***text***` | Bold and italic | `***critical***` | ***critical*** |
| `~~text~~` | Strikethrough (GFM) | `~~deprecated~~` | ~~deprecated~~ |
| `` `code` `` | Inline code | `` `SELECT *` `` | `SELECT *` |
| `<sub>text</sub>` | Subscript (HTML) | `H<sub>2</sub>O` | H₂O |
| `<sup>text</sup>` | Superscript (HTML) | `x<sup>2</sup>` | x² |
| `<mark>text</mark>` | Highlight (HTML) | `<mark>note</mark>` | Highlighted text |

---

## Lists

### Unordered Lists

| Syntax | Description | Example |
|--------|-------------|---------|
| `- item` | Dash bullet | `- First item` |
| `* item` | Asterisk bullet | `* First item` |
| `+ item` | Plus bullet | `+ First item` |
| `  - item` | Nested item (2 spaces) | `  - Sub-item` |
| `    - item` | Double-nested (4 spaces) | `    - Sub-sub-item` |

### Ordered Lists

| Syntax | Description | Example |
|--------|-------------|---------|
| `1. item` | Numbered list | `1. First step` |
| `1. item` (repeated) | Auto-increment (any number works) | `1. Step one\n1. Step two` |
| `   1. item` | Nested ordered (3 spaces) | `   1. Sub-step` |

### Task Lists (GFM)

| Syntax | Description | Example |
|--------|-------------|---------|
| `- [ ] task` | Unchecked checkbox | `- [ ] Write tests` |
| `- [x] task` | Checked checkbox | `- [x] Write code` |
| `- [X] task` | Checked (uppercase X) | `- [X] Deploy` |

---

## Links

| Syntax | Description | Example | Use Case |
|--------|-------------|---------|----------|
| `[text](url)` | Inline link | `[Google](https://google.com)` | Standard hyperlink |
| `[text](url "title")` | Link with tooltip | `[Docs](url "Open docs")` | Add hover text |
| `[text][ref]` | Reference-style link | `[Google][goog]` | Reuse URLs, cleaner text |
| `[ref]: url` | Reference definition | `[goog]: https://google.com` | Define at bottom of file |
| `[ref]: url "title"` | Reference with title | `[goog]: url "Google"` | Define with tooltip |
| `<url>` | Auto-link | `<https://google.com>` | Display URL as clickable link |
| `[text](#heading)` | Anchor link | `[Go to top](#overview)` | Link to section in same file |
| `[text](./file.md)` | Relative link | `[README](./README.md)` | Link to another file |

---

## Images

| Syntax | Description | Example | Use Case |
|--------|-------------|---------|----------|
| `![alt](url)` | Inline image | `![Logo](logo.png)` | Embed image |
| `![alt](url "title")` | Image with tooltip | `![Logo](logo.png "Our Logo")` | Image with hover text |
| `![alt][ref]` | Reference-style image | `![Logo][img-logo]` | Reusable image reference |
| `[![alt](img-url)](link-url)` | Clickable image | `[![Badge](badge.png)](repo.com)` | Linked image/badge |

---

## Code

| Syntax | Description | Example | Use Case |
|--------|-------------|---------|----------|
| `` `code` `` | Inline code | `` `dbt run` `` | Commands, variable names, short snippets |
| ```` ``` ```` | Fenced code block | ```` ```python ```` | Multi-line code |
| ```` ```lang ```` | Syntax highlighted block | ```` ```sql ```` | Language-specific highlighting |
| 4 spaces indent | Indented code block | `    code here` | Alternative code block |
| ```` ~~~lang ```` | Tilde fenced block | ```` ~~~bash ```` | Alternative fence style |

### Common Language Tags for Syntax Highlighting

| Tag | Language | Tag | Language |
|-----|----------|-----|----------|
| `python` | Python | `sql` | SQL |
| `bash` / `sh` | Shell/Bash | `yaml` / `yml` | YAML |
| `json` | JSON | `markdown` / `md` | Markdown |
| `javascript` / `js` | JavaScript | `typescript` / `ts` | TypeScript |
| `html` | HTML | `css` | CSS |
| `diff` | Diffs | `text` / `plain` | Plain text (no highlight) |

---

## Blockquotes

| Syntax | Description | Example | Use Case |
|--------|-------------|---------|----------|
| `> text` | Blockquote | `> Important note` | Quotes, callouts, notes |
| `> > text` | Nested blockquote | `> > Nested quote` | Quoted replies |
| `> **Note**` | Styled callout | `> **Note** Check this` | Emphasize within quote |
| `> text\n> more` | Multi-line quote | `> Line 1\n> Line 2` | Multiple lines |

### GitHub Alerts (GFM Extension)

| Syntax | Description | Use Case |
|--------|-------------|----------|
| `> [!NOTE]` | Blue info callout | General information |
| `> [!TIP]` | Green tip callout | Helpful suggestions |
| `> [!IMPORTANT]` | Purple important callout | Key information |
| `> [!WARNING]` | Yellow warning callout | Caution needed |
| `> [!CAUTION]` | Red caution callout | Dangerous action |

---

## Tables (GFM)

| Syntax | Description | Example |
|--------|-------------|---------|
| `\| col \| col \|` | Table header row | `\| Name \| Age \|` |
| `\|---\|---\|` | Header separator | `\|----\|----\|` |
| `\| val \| val \|` | Table data row | `\| Alice \| 30 \|` |
| `\|:---\|` | Left-align column | `\|:-----\|` |
| `\|---:\|` | Right-align column | `\|-----:\|` |
| `\|:---:\|` | Center-align column | `\|:----:\|` |

---

## Horizontal Rules

| Syntax | Description | Notes |
|--------|-------------|-------|
| `---` | Dash rule | Must have blank line above |
| `***` | Asterisk rule | Same rendering |
| `___` | Underscore rule | Same rendering |
| `- - -` | Spaced dashes | Also valid |

---

## Escaping Special Characters

| Character | Escaped | Character | Escaped |
|-----------|---------|-----------|---------|
| `\*` | Asterisk | `\#` | Hash |
| `\_` | Underscore | `\|` | Pipe |
| `\`` | Backtick | `\[` | Bracket |
| `\~` | Tilde | `\>` | Angle bracket |
| `\\` | Backslash | `\!` | Exclamation mark |

---

## HTML in Markdown

| Element | Use Case | Example |
|---------|----------|---------|
| `<br>` | Line break | `Line 1<br>Line 2` |
| `<details>` | Collapsible section | `<details><summary>Click</summary>Hidden</details>` |
| `<kbd>text</kbd>` | Keyboard key display | `Press <kbd>Ctrl</kbd>+<kbd>C</kbd>` |
| `<code>text</code>` | Inline code (HTML) | `<code>variable</code>` |
| `<!-- comment -->` | HTML comment (hidden) | `<!-- TODO: update this -->` |
| `&nbsp;` | Non-breaking space | `word&nbsp;word` |

---

## Footnotes (GFM)

| Syntax | Description | Example |
|--------|-------------|---------|
| `[^1]` | Footnote reference | `Text here[^1]` |
| `[^1]: text` | Footnote definition | `[^1]: This is the note` |
| `[^label]` | Named footnote | `See docs[^docs]` |

---

## Key Rules to Remember

1. **Blank lines matter** — separate paragraphs, required before/after code blocks and lists in some renderers
2. **Trailing spaces** — two spaces at end of line = line break (`<br>`); most people use `<br>` instead
3. **Indentation** — use 4 spaces (or 1 tab) for code blocks; 2 spaces for nested list items
4. **Reference links** — define at the bottom of file: `[label]: https://url.com`
5. **GFM vs Standard** — Task lists, tables, strikethrough, and alerts are GitHub-only (GFM), not standard Markdown
6. **Escaping** — prefix any special character with `\` to render it literally
