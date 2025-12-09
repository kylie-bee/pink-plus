# Pink+ Theme: VS Code → Zed Conversion Notes

These notes document how the Pink+ theme was ported from VS Code’s TextMate/semantic system to Zed’s Tree‑sitter–based themes, with a focus on Python. They also capture the debugging process and trade‑offs we made along the way.

---

## 1. Mental Model: VS Code vs Zed

### VS Code

- Uses **TextMate scopes** and optional **semantic tokens**.
- Theme rules look like:
  - `entity.name.function`, `support.function`
  - `keyword.control`, `keyword.operator.logical.python`
  - `constant.language`, `constant.numeric`
  - `variable.parameter.function-call.python`
- Scopes are very granular and often language‑specific:
  - `variable.parameter.function-call.python`
  - `keyword.operator.logical.python`
  - `meta.function-call.arguments.python`
  - etc.

### Zed

- Uses **Tree‑sitter captures**, exposed to themes as simple token names:
  - `keyword`, `keyword.control`, `keyword.operator`
  - `function`, `function.call`, `function.method.call`
  - `type`, `type.builtin`, `namespace`
  - `variable`, `variable.parameter`, `property`, `constant`, `boolean`
- Themes map **capture → style** in the `syntax` section:

```json
"syntax": {
  "function": { "color": "#99de5e" },
  "type":     { "color": "#4ec9b4" },
  "keyword":  { "color": "#915aeb" },
  "variable": { "color": "#f28ce8" },
  ...
}
```

- There is no direct support in the theme file for:
  - Contextual rules (“only in assignments”, “only in call arguments”)
  - TextMate‑style language‑specific scopes

The porting job is therefore:

> Map TextMate scopes and semantic intent → Zed captures, using **colors and categories** rather than exact scope names.

---

## 2. Core Pink+ Color Intent

From the original VS Code theme (`themes/pink_plus.json`), the core palette is:

| Category              | Color     | Notes                                           |
|-----------------------|-----------|-------------------------------------------------|
| Functions             | `#99de5e` | Function names, methods, calls                  |
| Types / Classes       | `#4ec9b4` | Types, classes, interfaces, namespaces          |
| Control Keywords      | `#915aeb` | `if`, `while`, `for`, `return`, `try`, etc.     |
| Keyword Operators     | `#c65584` | Word‑like ops: `in`, `and`, `or`, `not`, etc.   |
| Variables             | `#ff92ef` | VS Code variable pink (base reference color)    |
| Zed Variable (tuned)  | `#f28ce8` | Slightly cooler pink for better balance         |
| Constants (non‑builtin)| `#ff3fe2`| Enum members, miscellaneous constants           |
| Built‑in constants    | `#915aeb` | Python `None`, etc. (via `constant.builtin`)   |
| Strings               | `#d8b15c` | String literals                                 |
| Numbers               | `#cea8c1` | Numeric literals                                |
| Comments              | `#687469` | All comments, italic                            |
| Operators (symbols)   | `#D4D4D4` | `+`, `-`, `==`, etc.                            |
| General keywords      | `#c65584` | VS Code `keyword` (non‑control)                 |
| Foreground text       | `#D4D4D4` | Default editor text                             |
| Editor background     | `#201e23` | Main background                                 |

The goal of the Zed port is to **keep this semantic coloring**, even when the exact scopes differ.

---

## 3. Zed Theme Structure (Recap)

Zed theme file (`pink_plus.zed.json`) is a theme family:

- Top level:
  - `"$schema": "https://zed.dev/schema/themes/v0.2.0.json"`
  - `"name"`, `"author"`
  - `"themes": [ { "name", "appearance", "style": { ... } } ]`

Inside `style`:

- UI colors: `background`, `text`, `panel.background`, `tab.*`, `status_bar.background`, etc.
- Editor colors: `editor.background`, `editor.foreground`, `editor.active_line.background`, etc.
- Terminal colors: `terminal.*`, `terminal.ansi.*`
- **Syntax**: a single `syntax` object mapping captures to `HighlightStyleContent`:
  - `color`, `font_style` (`"normal"|"italic"|"oblique"`), `font_weight` (100–900)

Example:

```json
"syntax": {
  "function": {
    "color": "#99de5e"
  },
  "keyword": {
    "color": "#915aeb"
  },
  "string": {
    "color": "#d8b15c"
  }
}
```

---

## 4. Final Syntax Mapping (What We Ended Up With)

This is the important part: how we mapped Pink+ semantics onto Zed captures.

### 4.1 Functions & Methods

VS Code scopes: `entity.name.function`, `support.function`, `meta.function-call.*`, etc.

Zed captures → Pink+:

- `function`              → `#99de5e`
- `function.definition`   → `#99de5e`
- `function.call`         → `#99de5e`
- `function.method`       → `#99de5e`
- `function.method.call`  → `#99de5e`
- `function.builtin`      → `#99de5e`

Rationale:

- We tried briefly to color `function.call`/`function.method.call` like variables to distinguish assignment attributes, but it made code harder to read and wasn’t consistent with how Zed tags Python.
- Final decision: **all functions & calls are green**, matching the VS Code Pink+ function accent.

### 4.2 Types, Classes, Namespaces

VS Code scopes: `support.class`, `support.type`, `entity.name.type`, `entity.name.class`, `entity.name.namespace`, etc.

Zed captures → Pink+:

- `type`          → `#4ec9b4`
- `type.builtin`  → `#4ec9b4`
- `namespace`     → `#4ec9b4` (for languages that use it)
- `constructor`   → `#4ec9b4`
- `enum` / `variant` → `#4ec9b4`

This keeps `typing.TypedDict`, `APIRouter`, `SemanticDataModel`, etc. in the teal “type” family.

### 4.3 Keywords & Control Flow

VS Code Pink+:

- `keyword.control` → `#915aeb` (purple)
- `keyword` (generic) → `#c65584` (magenta‑pink)
- `keyword.operator.logical.python` / `.wordlike` → `#c65584`

Zed captures → Pink+:

- Base control keywords:
  - `keyword.control`  
  - `keyword.control.conditional`  
  - `keyword.control.repeat`  
  - `keyword.control.import`  
  - `keyword.control.return`  
  - `keyword.control.exception`  
  → **`#915aeb`** (purple)

- Generic `keyword`:
  - `keyword` → `#915aeb`  
  (aligned to your preference for a deeper purple for Python keywords.)

- Operators:
  - `keyword.operator`        → **`#c65584`** (magenta‑pink)
  - `keyword.operator.logical`→ `#c65584`
  - `keyword.operator.word`   → `#c65584`
  - `operator` (non‑keyword)  → `#D4D4D4` (white)

Effect in Python:

- `if`, `for`, `while`, `return`, `try`, `except`, etc. → purple.
- `in`, `not`, `and`, `or`, `is`, `not in`, `is not` (all keyword operators) → **pink** `#c65584`.
- `==`, `+`, `-`, `*`, etc. → white (`operator`).

### 4.4 Variables, Parameters, Properties

VS Code Pink+:

- `variable`, `support.variable`, `meta.definition.variable.name`, etc. → pink `#ff92ef`
- `variable.parameter.*` → variable pink
- `variable.language` (`self`, `this`) → magenta `#c65584`

Zed captures → Pink+:

- `variable`                 → `#f28ce8` (slightly cooler pink we tuned)
- `variable.parameter`       → `#ff92ef` (more vivid)
- `parameter`                → `#ff92ef`
- `variable.special`         → `#c65584` (for `self`/`this`/language‑special vars)
- `property`                 → `#ff92ef`
- `attribute`                → `#ff92ef`

Notes:

- Zed tags Python’s `typing` (module name) simply as `variable`, not `namespace` or `type`. That means:
  - `typing` in `typing.cast(...)` is pink like other variables.
  - The method part `.cast` is `function.method.call` (green).
- We accepted this as a Tree‑sitter/Zed semantic choice; theming alone can’t distinguish “module variable” from “local variable”.

### 4.5 Constants & Booleans (Including `None`)

The subtle tuning:

- `boolean` (e.g. `True`, `False`) → `#915aeb` (purple)
- `constant` (non‑builtin constants, enum members) → `#ff3fe2` (bright constant pink)
- `constant.builtin` (language builtins like `None`) → `#915aeb` (purple)

This yields:

- `True` / `False` → purple.
- `None` / other language builtins → **purple**, consistent with VS Code’s `constant.language` style.
- Enum‑like and non‑builtin constants can stay in the brighter constant accent color.

### 4.6 Strings, Escapes, Regex

VS Code Pink+ separates:

- Strings: `#d8b15c`
- Regex literals: `#d8ac4d`
- Escapes / regex operators: `#e77ecd`

Zed captures → Pink+:

- `string`               → `#d8b15c`
- `string.tag` / `text.literal` → `#d8b15c`
- `string.regex`         → `#d8ac4d`
- `string.escape`        → `#e77ecd`
- `string.special`       → `#e77ecd`
- `string.special.symbol`→ `#e77ecd`

### 4.7 Comments & Markup

- `comment`, `comment.doc` → `#687469`, italic.
- `emphasis` (Markdown) → italic.
- `emphasis.strong` → oblique/bold flavor.
- `title` (headings) → `#c65584`, bold.

### 4.8 Miscellaneous

- `embedded` → editor foreground `#D4D4D4` (for embedded languages / code).
- `link_text` → magenta `#db18ce`, italic.
- `link_uri`  → `#db18ce`.
- `primary` → `#D4D4D4` (fallback text).
- `preproc` → `#c65584` (preprocessor directives).
- `predictive` → teal `#4ec9b4`, italic (for inline completions / predicted code).

---

## 5. Editor & UI Tweaks That Mattered

A few non‑syntax details that significantly affected perceived correctness:

### 5.1 Active Line Background

Original:

```json
"editor.active_line.background": "#ffffff08",
"editor.highlighted_line.background": "#ffffff05",
```

This produced a faint white bar behind the active line, sometimes visually odd on top imports like `import pytest`.

We softened it:

```json
"editor.active_line.background": "#ffffff05",
"editor.highlighted_line.background": "#ffffff03",
```

If needed, these can be set to `null` to disable the overlay entirely.

### 5.2 Variable Color Tuning

Original variable pink was `#ff92ef` everywhere. With `typing`, `agent_platform`, etc. also being `variable`, this was visually heavy next to teal types and green functions.

We tuned **Zed’s** `variable` to:

```json
"variable": {
  "color": "#f28ce8"
}
```

This is a slightly cooler pink that still fits the cyberpunk palette but plays nicer with teals and greens.

---

## 6. How We Debugged Captures in Zed

Since Zed doesn’t have a built‑in “scope inspector” like VS Code’s, we used:

### 6.1 “Copy Highlight JSON”

In Zed, you can:

1. Open a file (e.g. Python).
2. Use the **Copy Highlight JSON** command (from command palette).
3. Paste the resulting JSON somewhere.

The JSON is a list of line fragments:

```json
{
  "text": "typing",
  "highlight": "variable"
}
{
  "text": "cast",
  "highlight": "function.method.call"
}
{
  "text": "None",
  "highlight": "constant.builtin"
}
{
  "text": "in",
  "highlight": "keyword.operator"
}
{
  "text": "True",
  "highlight": "boolean"
}
```

This told us exactly:

- `typing` is `variable`, not `namespace` or `type`.
- `cast` is `function.method.call`.
- `None` is `constant.builtin`.
- `TYPE_CHECKING` is `constant`.
- `in` / `not` / `and` / `or` are `keyword.operator`, not `keyword.operator.logical`.
- `True` / `False` are `boolean`.

From there we adjusted the `syntax` entries instead of guessing.

### 6.2 Using the Token List from Another Theme Repo

We also referenced a community token list (`tokens.md` in a public theme repo) that enumerated Zed captures per language, e.g. for Python:

- `@variable`
- `@property`
- `@type.class`
- `@constant`
- `@type`
- `@comment`
- `@string`
- `@keyword`
- `@function.method.call`
- `@function.call`
- `@function.decorator`
- `@function.definition`
- `@variable.parameter`
- `@function.kwargs`
- `@function.builtin`
- `@boolean`
- `@constant.builtin`
- `@number`
- `@variable.special`
- `@punctuation.*`
- `@operator`
- `@keyword.operator`
- `@keyword.definition`
- `@attribute.builtin`
- `@type.builtin`

This confirmed which capture names were valid and worth styling, even though it didn’t provide an in‑editor inspector.

---

## 7. Practical Limits & Known Differences

Even with all the tuning, there are inherent limits:

1. **Capture granularity is different**  
   - VS Code has per‑language scopes like `keyword.operator.logical.python` and `variable.parameter.function-call.python`.
   - Zed collapses these into broader categories like `keyword.operator` and `variable.parameter`.

2. **No contextual theming in theme JSON**  
   - You can’t say “color this `variable` differently when it’s part of an attribute assignment” or “only when inside `typing.cast`”.
   - You can only color based on the final capture name Zed assigns.

3. **Some Python semantics differ from VS Code**  
   - `typing` is a `variable`, not `namespace`.
   - Assignment attributes and function definitions sometimes share captures, so you can’t differentiate them by theme alone.

4. **Semantic tokens vs captures**  
   - VS Code semantic tokens can differentiate `parameter` vs `local` vs `namespace` at a higher level; Zed’s theme JSON only sees Tree‑sitter captures (and possibly LSP‑driven ones), not VS Code’s semantic token categories.

Overall, we pushed the theme as far as we can **within Zed’s capture system**, with a strong focus on Python and a good approximation of the Pink+ VS Code look and feel.

---

## 8. How to Iterate Further

If you want to tweak more in the future:

1. **Use Copy Highlight JSON**  
   - Identify the `highlight` value (`function.*`, `type.*`, `variable`, etc.) for any token that looks wrong.

2. **Add or adjust `syntax` entries**  
   - Use the capture name as a key:
     - `function.decorator`
     - `string.escape.regex`
     - `variable.other.member` (if present in newer grammars)
   - Map them to one of your existing color families.

3. **Keep changes small and reversible**  
   - Start with one capture at a time.
   - Reload theme, check visually, then commit.

4. **If something still feels impossible**  
   - It may require a grammar change (e.g., tagging `typing` as `namespace` instead of `variable`) or new Zed features, not just theme tweaks.

For now, the Pink+ Zed theme is in a good place: Python code reads with the intended Pink+ cyberpunk+nature palette, and the key pain points (`in`/`not`, `None`, function defs/usages, parameters, and variables) have all been tuned explicitly.