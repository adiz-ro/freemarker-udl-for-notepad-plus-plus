# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single deliverable: `freemarker.udl.xml`, a Notepad++ 8.x **User Defined Language** (UDL 2.1)
that highlights FreeMarker on top of HTML, with CSS and JavaScript in the same file
(`.ftl` / `.ftx`).

There is no build, no package manager, no test suite, no source tree. The XML *is* the
product; the two READMEs are the documentation. Everything else in the working directory
(`git_*.bat` helper scripts, Romanian prompts) is untracked — `*.bat` is in `.gitignore`.

## Two upstream sources of truth

Nothing in this repo is invented; every list and color is copied from somewhere else, and
that is the constraint to respect when editing:

1. **The VS Code extension "FreeMarker Tag Auto-Close & Syntax"** — the FreeMarker rules
   (directive names, block vs. inline, built-ins, macro syntax) come from its
   `syntaxes/freemarker.tmLanguage.json`, `freemarker.injection.json` and `src/freemarker.ts`.
   Those files are **not present in this repo**; the READMEs reference them as the origin of
   the fold list. Treat the extension's directive set as authoritative when adding directives.
2. **The Notepad++ sources** — colors from `stylers.model.xml` (`LexerType name="html"`),
   vocabulary from `langs.model.xml`: `instre1` for `html` (manually split into tags and
   attributes), plus `javascript` and `css`. The goal is that the HTML part looks identical
   whether the file is opened as **Language -> H -> HTML** or as **FreeMarker**, so do not
   "improve" the palette.

## How the file is structured

Keyword-list slots (`<Keywords name="KeywordsN">`), each bound to the matching
`<WordsStyle name="KEYWORDSN">`:

| Slot | Contents | Style |
| --- | --- | --- |
| Keywords1 | FreeMarker directives, `#if` and `/#if` forms | `6424DB` bold italic |
| Keywords2 | just `?` — prefix match, colors all built-ins | `6424DB` italic |
| Keywords3 | word operators: `as in using to step gt lt and or not true false` | `000080` bold |
| Keywords4 | just `@` — prefix match, colors all macro calls | `008040` |
| Keywords5 | HTML tags, both `div` and `/div`, plus `<` `>` `/` | `0000FF` |
| Keywords6 | HTML attributes | `FF0000` |
| Keywords7 | JavaScript keywords | `000080` bold italic |
| Keywords8 | CSS at-rules and properties | `8080C0` italic |

Prefix matching for Keywords2 and Keywords4 is switched on in `<Settings><Prefix .../>`;
that is what makes `?upper_case` and `@ui.button` work without listing every name.

`<Keywords name="Delimiters">` carries `${...}`, `#{...}`, both quote styles, and the
`<#-- -->` comment. Folding lives in `Folders in code1, open/middle/close`.

## Editing rules that are easy to break

These are lexer facts, not style preferences — violating them silently kills highlighting:

- **UDL matches whole tokens.** Text is split at every operator and space, so a keyword
  written `<div` or `<#if` can never match. Tag names are bare words (`div`, `/div`),
  directives are `#if` / `/#if`, and `<`, `>`, `/` are themselves entries in Keywords5.
- **`/` was deliberately removed from Operators1** so `/div` and `/#if` stay single tokens.
  Putting it back breaks every closing tag and closing directive.
- **A word can live in only one list.** `style`, `title`, `data`, `dir`, `list`, `cite`,
  `summary` are filed as attributes (red) rather than tags, on purpose.
- **`#assign` is intentionally absent from the fold lists.** `<#assign body>` (block) and
  `<#assign x = 1>` (assignment) are indistinguishable without a parser, so folding it would
  corrupt fold levels. Same reasoning keeps HTML tags out of folding entirely.
- **The comment is a delimiter, not a Comment.** With `<#-- -->` in the comment slot, a
  Notepad++ lexer defect pushes fold markers down one line per comment. Hence
  `allowFoldOfComments="no"` and the loss of `Ctrl+Q`. Do not "fix" this by moving it.
- Adding a new FreeMarker block directive means touching **four** places: `Keywords1` (open
  and `/#name` forms) and `Folders in code1, open` + `, close`.

The `## Limitations` section of `README.md` documents which visual differences from the HTML
lexer are known and accepted — check it before treating one as a bug.

## Working on the docs

`README.md` (English) and `Readme_ro.md` (Romanian) are the same document with the same
section order and cross-link each other. Any change to behavior, keyword lists, colors or
limitations must be made in **both**, keeping the sections aligned. Romanian text in this
repo is written without diacritics; match that.

## Verifying a change

Manual only — there is no automated check:

```bash
copy freemarker.udl.xml "%APPDATA%\Notepad++\userDefineLangs\"
```

Notepad++ reads UDL files **only at startup**, so it must be closed before the copy and
restarted after. Open an `.ftl` file and confirm: status bar reads `FreeMarker`, tags blue,
attributes red, `<#-- -->` green, `+`/`-` fold markers next to `<#if>` / `<#list>` /
`<#macro>`, `${...}` on cream background.

Caveat when editing through **Language -> User Defined Language -> Define your language...**:
Notepad++ writes the result back to its own copy in `userDefineLangs\`, not to this repo —
copy it back before committing, and remember the UI cannot express the whole-token rule for
you.
