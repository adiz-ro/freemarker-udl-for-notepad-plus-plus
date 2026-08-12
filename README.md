# FreeMarker for Notepad++

`freemarker.udl.xml` is a **User Defined Language** that brings to Notepad++ the same
syntax rules as the VS Code extension in this repo: FreeMarker on top of HTML, with CSS
and JavaScript in the same file. After installation, `FreeMarker` appears in the
**Language** menu, and `.ftl` and `.ftx` files are highlighted automatically when opened.

Requirements: Notepad++ 8.x (UDL 2.1 format). Light theme, white background.

## Installation

The quick way, without the UI:

1. Close Notepad++.
2. Copy `freemarker.udl.xml` to `%APPDATA%\Notepad++\userDefineLangs\`
   (normally `C:\Users\<user>\AppData\Roaming\Notepad++\userDefineLangs\`).
3. Start Notepad++.

You can do step 2 from a terminal:

```bash
copy notepadpp\freemarker.udl.xml "%APPDATA%\Notepad++\userDefineLangs\"
```

The UI way, if you prefer:

**Language** -> **User Defined Language** -> **Define your language...** -> **Import...**
-> select `freemarker.udl.xml` -> restart Notepad++.

Verification: open `sample.ftl` from the project root. The status bar at the bottom should
read **FreeMarker**, HTML tags are blue, attributes red, the `<#-- ... -->` comment green,
and `+`/`-` markers appear in the left margin on `<#if>`, `<#list>`, `<#macro>`, `<div>`,
`<table>`, `<script>`.

If the file still opens with the HTML lexer, it means `ftl` and `ftx` are listed as user
extensions for the HTML language (**Settings -> Preferences -> Language**, list on the
right, bottom field). Remove them from there and the UDL takes over the files.

Uninstall: delete the file from `userDefineLangs` and restart Notepad++.

## What it highlights

The palette is copied from the Notepad++ HTML lexer (`stylers.model.xml`, `LexerType
name="html"`), so there is no visible difference when you switch from **Language -> H ->
HTML** to **FreeMarker**. The vocabulary also comes from the Notepad++ sources:
`langs.model.xml`, `instre1` for `html` (split into tags and attributes), `javascript`
and `css`.

| Element | Example | Notepad++ style | Color |
| --- | --- | --- | --- |
| HTML tags | `<div`, `</table>`, `>`, `/>` | TAG | blue `0000FF` |
| HTML attributes | `class`, `href`, `src`, `type` | ATTRIBUTE | red `FF0000` |
| Strings, attribute values | `"text"` / `'text'` | DOUBLE / SINGLE STRING | purple `8000FF` |
| Numbers | `42`, `3.14` | NUMBER | red `FF0000` |
| Plain text | text between tags | DEFAULT | black, bold |
| JavaScript | `function`, `const`, `return` | JS KEYWORD | navy `000080` |
| CSS | `@media`, `display`, `background-color` | CSS IDENTIFIER | `8080C0` |
| FreeMarker directives | `<#if>`, `<#assign>`, `</#list>` | USER TAGS1 | purple `6424DB`, bold italic |
| Macro calls | `<@ui.button ...>`, `</@card>` | TAG UNKNOWN | green `008040` |
| Built-ins | `?upper_case`, `?size` | - | `6424DB`, italic |
| Word operators | `as`, `in`, `gt`, `and`, `not`, `true` | - | `000080`, bold |
| Interpolations | `${nume}`, `#{pret}` | - | `6424DB` on cream `FEFDE0` |
| Comments | `<#-- ... -->` | COMMENT | green `008000` |

The first six rows are exactly the HTML lexer's styles. The last five are things the HTML
lexer does not know about; the chosen colors stay within its palette though - `6424DB` is
the very color you already have on `USER TAGS1`, where you put your FreeMarker directives.

Directives and interpolations are also highlighted **inside** HTML attributes, so
`class="<#if activ>on</#if>"` and `href="${url}"` look right - the equivalent of the
injection grammar in the VS Code extension.

The `>` inside expressions breaks nothing: `<#if (produse?size > 1)>` is highlighted all
the way through, because tags are treated as tokens, not as `<#` ... `>` delimiters.

## Folding

- **FreeMarker** - exactly the block directives from `src/freemarker.ts` fold: `#if`,
  `#list`, `#macro`, `#function`, `#attempt`, `#compress`, `#escape`, `#noescape`,
  `#noparse`, `#switch`, `#outputformat`, `#autoesc`, `#noautoesc`, `#transform`.
  `<#else>`, `<#elseif>`, `<#recover>`, `<#case>`, `<#items>`, `<#sep>` are intermediate
  markers, they do not open a new level.
- **HTML** - the containers that in practice are always closed: `html`, `head`, `body`,
  `div`, `table`, `tr`, `ul`, `form`, `script`, `style`, `section`, `header`, `footer`,
  `nav`, `main`, `textarea`, `button`, `label` and others.
- **Comments** `<#-- -->` are foldable too.

`<#assign>` is not part of folding: the block form `<#assign body>` and the assignment
form `<#assign x = 1>` cannot be told apart without a parser, and including them would
break the folding levels. `<p>`, `<li>`, `<td>`, `<option>`, `<a>`, `<span>` are left out
of folding precisely because they are often left unclosed in HTML.

## Limitations, so you know upfront

UDL is a single-level lexer, not a grammar system like TextMate. This is where the
differences from the native HTML lexer come from:

1. **There is no context.** The HTML lexer knows whether a word is inside a tag; UDL does
   not. In practice: `class` or `width` are colored red even when they appear in plain
   text or in JavaScript, and `display` stays blue even outside a `<style>`. Tags don't
   have this problem, because `<` is part of the keyword (`<div`, `</div`).
2. **There are no real nested languages.** Inside `<script>` and `<style>` the JavaScript
   or CSS lexer does not run; there are only the keyword lists from `langs.model.xml`. So
   the light-blue background of the JavaScript block, the `//` and `/* */` comments inside
   scripts, and CSS selector highlighting are all missing.
3. **Unquoted values and entities** (`&nbsp;`) remain plain text. In the HTML lexer they
   are orange on cream and black on cream, respectively.
4. **Built-ins work on the `?` prefix**, so any `?name` gets colored - including made-up
   ones. Side effect: the `??` operator also gets the built-in color.
5. **Single quotes are global.** An apostrophe in plain text starts a string that runs
   until the next apostrophe. If this bothers you, delete `09' 10\ 11'` from the
   `<Keywords name="Delimiters">` line and save the file.

In exchange, compared to the HTML lexer you gain exactly the things this file exists for:
`<#-- -->` comments are real comments (green, foldable), `${...}` interpolations are
visible, `<@...>` macros are no longer an "unknown tag", and folding works on FreeMarker
directives.

There is no automatic tag closing, as in the VS Code extension - Notepad++ cannot do that
from a UDL. For that you need the **XML Tools** plugin or
`Settings -> Preferences -> Auto-Completion`, and even those don't know the FreeMarker
directives.

## Customization

Colors are most conveniently changed from **Language** -> **User Defined Language** ->
**Define your language...**, the `Folder & Default`, `Keywords Lists`, `Comment & Number`,
`Operators & Delimiters` tabs. Notepad++ saves back to
`userDefineLangs\freemarker.udl.xml` on its own.

To add other file extensions, edit the `ext` attribute on the second meaningful line of
the file:

```xml
<UserLang name="FreeMarker" ext="ftl ftx ftlh ftlx" udlVersion="2.1">
```
