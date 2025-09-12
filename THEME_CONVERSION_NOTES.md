# Pink+ Theme: VS Code to Zed Conversion Notes

## Overview

This document explains the conversion of the Pink+ theme from VS Code format to Zed format, including the issues found in the original conversion and how they were resolved.

## Issues with Original Zed Theme

The initial Zed theme conversion had several critical problems:

1. **Invalid JSON Format**: Used C-style comments (`/* */`) which are not valid in JSON
2. **Minimal Syntax Highlighting**: Only 7 basic syntax types vs. 40+ specific token types in VS Code
3. **Incorrect Property Names**: Used VS Code property names instead of Zed equivalents
4. **Missing UI Elements**: Many VS Code UI colors weren't mapped to Zed equivalents
5. **Poor Color Accuracy**: Syntax colors didn't match the rich Pink+ color palette

## Key Color Palette

The Pink+ theme uses this core color palette:

| Purpose | Color | Usage |
|---------|-------|-------|
| Functions | `#99de5e` | Function names, methods |
| Types | `#4ec9b4` | Classes, types, interfaces |
| Control Keywords | `#915aeb` | if, while, return, etc. |
| Variables | `#ff92ef` | Variable names, properties |
| Constants | `#ff3fe2` | Constants, enum members |
| Strings | `#d8b15c` | String literals |
| Numbers | `#cea8c1` | Numeric literals |
| Comments | `#687469` | All comments |
| Operators | `#D4D4D4` | +, -, =, etc. |
| Keywords | `#c65584` | General keywords |

## VS Code to Zed Property Mapping

### UI Elements

| VS Code Property | Zed Property | Notes |
|------------------|--------------|-------|
| `editor.background` | `editor.background` | Direct mapping |
| `sideBar.background` | `panel.background` | Zed uses "panel" for sidebars |
| `activityBar.background` | `element.background` | Activity bar is an element |
| `statusBar.background` | `status_bar.background` | Direct mapping |
| `tab.selectedBackground` | `tab.active_background` | "selected" → "active" |
| `tab.inactiveBackground` | `tab.inactive_background` | Direct mapping |
| `editorGroupHeader.tabsBackground` | `tab_bar.background` | Tab container |

### Editor Features

| VS Code Property | Zed Property | Notes |
|------------------|--------------|-------|
| `editorIndentGuide.background1` | `editor.indent_guide` | Simplified naming |
| `editorIndentGuide.activeBackground1` | `editor.indent_guide_active` | Active guides |
| `editor.selectionHighlightBackground` | `editor.document_highlight.read_background` | Document highlights |
| `peekView.border` | `border.focused` | Focus indication |

## Syntax Highlighting Conversion

### Token Type Mapping

The VS Code theme uses TextMate scopes, while Zed uses semantic token types:

| VS Code Scope | Zed Token | Color |
|---------------|-----------|-------|
| `entity.name.function` | `function` | `#99de5e` |
| `entity.name.type` | `type` | `#4ec9b4` |
| `keyword.control` | `keyword` | `#c65584` |
| `variable` | `variable` | `#ff92ef` |
| `constant.numeric` | `number` | `#cea8c1` |
| `string` | `string` | `#d8b15c` |
| `comment` | `comment` | `#687469` |

### Enhanced Syntax Support

The new Zed theme includes support for:

- **Regex strings**: `string.regex` with `#d8ac4d`
- **String escapes**: `string.escape` with `#e77ecd`  
- **Built-in functions**: `function.builtin` with `#99de5e`
- **Special variables**: `variable.special` with `#c65584`
- **Markdown emphasis**: `emphasis` and `emphasis.strong`
- **Link text**: `link_text` and `link_uri` with `#db18ce`

## Terminal Colors

Improved terminal colors for better readability:

| Color | Value | Purpose |
|-------|-------|---------|
| Red | `#ff6b6b` | Errors, deletions |
| Green | `#51cf66` | Success, additions |
| Yellow | `#ffd43b` | Warnings, modifications |
| Blue | `#339af0` | Information |
| Magenta | `#f06292` | Special highlights |
| Cyan | `#22d3ee` | Utilities |

## UI State Colors

Added comprehensive support for UI states:

- **Error states**: Red (`#f44747`) with 20% opacity backgrounds
- **Warning states**: Yellow (`#ffd43b`) with 20% opacity backgrounds  
- **Success states**: Green (`#51cf66`) with 20% opacity backgrounds
- **Info states**: Blue (`#339af0`) with 20% opacity backgrounds

## Bracket Highlighting

Zed doesn't support the same bracket highlighting as VS Code's rainbow brackets, but uses:
- `editor.document_highlight.bracket_background` for matching brackets
- Standard syntax highlighting for bracket punctuation

## Limitations & Differences

1. **Scope Granularity**: Zed's syntax highlighting is less granular than VS Code's TextMate scopes
2. **Semantic Tokens**: Zed relies more on language servers for semantic highlighting
3. **Theme Structure**: Zed themes are more structured and consistent than VS Code themes
4. **Bracket Colors**: No direct equivalent to VS Code's rainbow bracket colors
5. **Peek View**: Zed doesn't have peek views, so those colors are repurposed for other UI elements

## Testing Recommendations

To verify the theme works correctly:

1. **Syntax Highlighting**: Test with various file types (JS, TS, Rust, Python, etc.)
2. **UI Elements**: Check panel backgrounds, tab colors, status bar
3. **Terminal**: Verify ANSI colors work properly
4. **Search**: Test search highlighting and matches
5. **Diagnostics**: Check error, warning, and info highlighting

## Future Improvements

Potential enhancements:
- Add light theme variant
- Adjust colors based on user feedback
- Add theme-specific icon colors if Zed supports them
- Fine-tune contrast ratios for accessibility