# Pink+ Theme

Pink+ is a dark, high‑contrast theme with a cyberpunk‑meets‑nature palette: bright pinks and magentas, cool teals, and vivid greens on a deep, muted background.

This repo contains:

- The **VS Code** Pink+ theme (`themes/pink_plus.json`)
- The **Zed** Pink+ port (`themes/pink_plus.zed.json`)
- Notes on how the VS Code scopes were translated into Zed captures (`THEME_CONVERSION_NOTES.md`)

The intent is for the **VS Code theme to be the canonical design**, with the Zed theme kept in sync as closely as Zed’s highlighting model allows.

---

## Theme Intent & Design

High‑level goals:

- **Readable on long sessions**: deep, low‑glare background (`#201e23`) with soft contrast for structural UI, but crisp contrast for code.
- **Distinct semantic categories**:
  - Functions/calls/methods → green (`#99de5e`)
  - Types/classes/namespaces → teal (`#4ec9b4`)
  - Control keywords (`if`, `for`, `while`, `return`, `try`, `except`) → deep purple (`#915aeb`)
  - Logical/word operators (`in`, `and`, `or`, `not`, `is`) → magenta‑pink (`#c65584`)
  - Variables → slightly cool pink (`#f28ce8`)
  - Enum‑like constants → brighter magenta (`#ff3fe2`)
  - Built‑in constants (`None` etc.) and booleans (`True`/`False`) → purple (`#915aeb`)
  - Strings → warm gold (`#d8b15c`)
  - Numbers → soft lilac (`#cea8c1`)
  - Comments → muted green‑gray (`#687469`, italic)

The overall effect is: **functions and control flow pop in green/purple**, types and structural things are teal, and “user‑data” like variables and strings are warm pink/gold.

---

## Structure of This Repo

- `themes/pink_plus.json`  
  The original **VS Code** theme file. This is the ground truth for colors and usage.

- `themes/pink_plus.zed.json`  
  The **Zed** theme port, mapping TextMate scopes / semantic intent into Zed’s Tree‑sitter captures.

- `THEME_CONVERSION_NOTES.md`  
  A detailed design document on how the VS Code scopes map to Zed’s captures (especially for Python), including:
  - How we matched keywords, operators, variables, types, and constants.
  - How built‑in constants vs normal constants differ.
  - How we tuned variable colors and active line backgrounds.
  - How to debug what Zed is actually highlighting via “Copy Highlight JSON”.

- `README.md` (this file)  
  High‑level orientation and quick reference.

---

## VS Code Usage

1. Copy or symlink `themes/pink_plus.json` into your VS Code extensions folder, or package it as part of a VS Code extension.
2. In your `settings.json`:

   ```jsonc
   "workbench.colorTheme": "Pink+"
   ```

3. If you tweak colors, treat `themes/pink_plus.json` as your primary source of truth and regenerate or sync any published VS Code extension from it.

---

## Zed Usage

1. Copy `themes/pink_plus.zed.json` to your local Zed themes directory:
   - macOS/Linux: `~/.config/zed/themes/`
   - Windows: `%USERPROFILE%\AppData\Roaming\Zed\themes\`
2. Restart Zed.
3. Open the theme selector and choose **Pink+ Dark**.

If you modify the Zed theme locally, keep in mind that:

- **Zed uses captures like** `function`, `function.call`, `type`, `variable.parameter`, `keyword.operator` instead of VS Code’s scopes.
- Some differences (e.g. `typing` being a `variable` instead of `namespace`) are a function of Zed’s grammars and can’t be changed by the theme alone.

For deeper details, see `THEME_CONVERSION_NOTES.md`.

---

## Debugging Highlighting in Zed

To understand how Zed is actually classifying tokens (e.g. why `in` or `typing` is a certain color):

1. Open a file (e.g. Python).
2. Use Zed’s command palette to run **Copy Highlight JSON**.
3. Paste the result into a scratch buffer and look at entries like:

   ```json
   { "text": "typing", "highlight": "variable" }
   { "text": "cast",   "highlight": "function.method.call" }
   { "text": "None",   "highlight": "constant.builtin" }
   { "text": "in",     "highlight": "keyword.operator" }
   ```

4. Match those `highlight` strings against the `syntax` object in `themes/pink_plus.zed.json`:
   - If there is no entry (e.g. no `constant.builtin`), add one with an appropriate Pink+ color.
   - If the entry exists but feels wrong, adjust it there rather than guessing new names.

This is how we iterated on:

- Fixing `in`/`not`/`and`/`or` to be pink instead of white.
- Making `None` and other built‑ins align with booleans and control colors.
- Getting function definitions/usages, parameters, and variables into the right color families.

---

## Updating Tagged Colors in VS Code Theme

In the VS Code theme (`themes/pink_plus.json`), some colors are “tagged” via inline comments (e.g. `// #function`) so they can be updated consistently.

To bulk‑update a tagged color in VS Code:

1. Use search with this regexp:

   ```regexp
   ("#)([0-9a-fA-F]{3}|[0-9a-fA-F]{6})(",? // #tag-name$)
   ```

2. Replace with:

   ```regexp
   $1new-color$3
   ```

This lets you quickly change all places using a tagged semantic color (`#function`, `#type`, etc.) without hunting each instance.

---

## Contributing / Tweaking

If you want to experiment:

- Make small, capture‑based changes in `themes/pink_plus.zed.json`:
  - e.g., add `constant.builtin`, adjust `namespace`, or tune `editor.active_line.background`.
- Use **Copy Highlight JSON** to verify what Zed is doing before you change theme mappings.
- Keep `themes/pink_plus.json` as your canonical reference for what colors are supposed to mean.

PRs or issues that improve language coverage (e.g., JS/TS, Rust, CSS) while respecting the Pink+ palette are very welcome.