# Borland

A Borland / Turbo C++ inspired theme for JetBrains IDEs. Blue editor background,
yellow text, white keywords, cyan strings — the classic DOS-era look.

## Project layout

```
resources/                      ← resource root (becomes the root of the plugin jar)
├── META-INF/
│   ├── plugin.xml              ← plugin id, version, vendor, themeProvider
│   └── pluginIcon.svg
└── theme/
    ├── borland.theme.json      ← UI theme: palette + IDE chrome (panels, selection, scrollbar)
    └── Borland.xml             ← editor color scheme (syntax highlighting)
```

`borland.theme.json` references the editor scheme as `/theme/Borland.xml` — that path
is resolved from the resource root, **not** relative to the json file. Keep them in sync
if you move anything.

## Build & install locally

Package (jar of `resources/`, wrapped in the standard `<name>/lib/` zip layout):

```sh
mkdir -p dist/borland/lib
(cd resources && zip -qr ../dist/borland/lib/borland-1.0.0.jar .)
(cd dist && zip -qr borland-1.0.0.zip borland && rm -rf borland)
```

Install: **Settings → Plugins → ⚙ → Install Plugin from Disk… → `dist/borland-1.0.0.zip`**,
restart, then pick **Borland** under **Settings → Appearance & Behavior → Appearance → Theme**.
The editor scheme is applied automatically; it also appears under **Editor → Color Scheme**.

To iterate: edit the files, rebuild the zip, reinstall from disk (the IDE replaces the
old version), restart.

## Debugging

- **Theme shows up but the editor stays Darcula-colored** → the `editorScheme` path in
  `borland.theme.json` doesn't match the file's location inside the jar. Verify with
  `unzip -l dist/borland/lib/borland-1.0.0.jar` — the path in the json must match a jar
  entry exactly (leading `/` = jar root).
- **Theme doesn't appear in the Appearance list at all** → check `resources/META-INF/plugin.xml`
  has the `<themeProvider>` extension and that the jar actually contains `META-INF/plugin.xml`
  (a common failure is zipping the wrong directory level). Also check **Help → Show Log in
  Finder** (`idea.log`) for plugin-load errors right after startup.
- **A color looks wrong in the editor** → put the caret on the offending token and use
  **Jump to Colors and Fonts** (search it in Find Action, ⇧⌘A) — it names the attribute key,
  which you can then override in `Borland.xml`. Unlisted keys inherit from
  `parent_scheme="Darcula"`.
- **A UI element looks wrong** → enable the **UI Inspector** (Help → Diagnostic Tools →
  or `ide.ui.inspector` in Registry), Ctrl-Alt-click the element to see which
  `*.background`-style key it uses, then set that key in the `"ui"` section of
  `borland.theme.json`. Full key list: [Theme metadata](https://plugins.jetbrains.com/docs/intellij/themes-metadata.html).
- **Validate before packaging**: `python3 -m json.tool resources/theme/borland.theme.json`
  and `xmllint --noout resources/theme/Borland.xml`.
- For heavier development, the [DevKit workflow](https://plugins.jetbrains.com/docs/intellij/developing-themes.html)
  (`borland.iml` is already wired to `resources/META-INF/plugin.xml`) lets you run a sandbox
  IDE with the theme loaded instead of reinstalling.

## Publishing to JetBrains Marketplace

1. Bump `<version>` in `resources/META-INF/plugin.xml` (the Marketplace rejects re-uploads
   of an existing version) and update `<change-notes>`.
2. Rebuild the zip with the matching filename.
3. Sign in at [plugins.jetbrains.com](https://plugins.jetbrains.com) → **Upload plugin** →
   choose the zip, category **UI**.
4. First upload goes through manual moderation (typically 1–2 business days); updates are
   usually approved faster. Vendor name/email must be real (already set) and the plugin
   `<id>` (`com.tanaymehra.borland`) can never change after the first upload.
5. Optional: add screenshots on the plugin page — themes without them get few installs.

## Alternate variant (dark background)

The classic scheme uses Borland blue. For a black/gray variant, background options:

| name         | hex       |
|--------------|-----------|
| borland_blue | `#0000AA` |
| dark_blue    | `#00003c` |
| dark_gray    | `#1B1C1A` |
| black        | `#000000` |

Steps to add one (a plugin may ship several themes):

1. Copy `theme/borland.theme.json` → `theme/borland-dark.theme.json`; change `"name"` to
   `"Borland Dark"`, set `"darkblue"`-equivalent background entries to the new background,
   and point `"editorScheme"` to `/theme/BorlandDark.xml`.
2. Copy `theme/Borland.xml` → `theme/BorlandDark.xml`; change `<scheme name="Borland Dark">`
   (the name must be unique) and swap the background values: `TEXT`/`GUTTER_BACKGROUND`
   `003078` → new bg, `CARET_ROW_COLOR`/other `004078` accents → a slightly lighter shade
   of the new bg.
3. Register it in `resources/META-INF/plugin.xml` next to the existing provider (ids must
   differ):
   ```xml
   <themeProvider id="com.tanaymehra.borland.dark" path="/theme/borland-dark.theme.json"/>
   ```
4. Rebuild the zip — both themes will show up in the Appearance list.

## Palette reference ("New Scheme")

| dark*       | hex       | light*       | hex       |
|-------------|-----------|--------------|-----------|
| darkblack   | `#000000` | lightblack   | `#575757` |
| darkwhite   | `#a8a8a8` | lightwhite   | `#ffffff` |
| darkred     | `#a80000` | lightred     | `#ff5757` |
| darkorange  | `#a85700` | lightorange  | `#ffa857` |
| darkyellow  | `#a8a800` | lightyellow  | `#ffff57` |
| darkgreen   | `#308800` | lightgreen   | `#57ff57` |
| darkcyan    | `#00a8a8` | lightcyan    | `#57ffff` |
| darkblue    | `#003078` | lightblue    | `#5757ff` |
| darkmagenta | `#a800a8` | lightmagenta | `#ff57ff` |
| darkscroll  | `#004078` | lightscroll  | `#006078` |

Role mapping in the editor scheme: background `darkblue`, plain text `lightyellow`,
keywords `lightwhite` (bold), strings/numbers `lightcyan`, comments `darkwhite` (italic),
preprocessor/fields `lightgreen`, constants/escapes `lightmagenta`, errors `lightred`,
warnings `lightorange`, selection `lightblue`.
