# mdview

A small, transparent Bash script that renders Markdown files with consistent
GitHub-style formatting - no server, no Electron, no JavaScript framework required.

```
mdview README.md
```

That's it. Under the hood it's just `pandoc` + your browser, glued
together with about 80 lines of Bash you can read in two minutes.

| Default GitHub formatting | Wide - Document formating |
|---|---|
| ![default](screenshots/default%20github.png) | ![wide](screenshots/wide_formatting.png) |

## Why

Most Markdown viewers for Linux are either:

- a full editor (VS Code, Obsidian, Typora) when you just want to *read*
  a file, or
- a heavyweight native app (Electron, Tauri, Rust/egui binaries) with its
  own rendering engine, theme system, and update cycle.

`mdview` takes a different approach: reuse `pandoc` (which already
renders Markdown better than most bespoke renderers) and your existing
browser (which already renders HTML/CSS better than any custom viewer
ever will).  
The script is glue, not a rendering engine - which also
means it inherits every improvement pandoc and your browser make, for
free, forever.

## Features

- **Self-contained output** - images and resources are embedded
  (base64) into a single HTML file via `pandoc --embed-resources`, so
  the result is portable and doesn't break if the source folder moves.
- **Works over network mounts** - resolves and `cd`s into the actual
  directory of the Markdown file before invoking pandoc, so relative
  image paths work correctly even over GVFS/SFTP/NFS mounts, not just
  the local filesystem.
- **App-style browser window** - opens in `chromium --app` or
  `brave-browser --app` if available (clean window, no tabs or address
  bar), falling back to `xdg-open` and your system's default browser
  otherwise.
- **Optional full-width layout** - `--wide` / `-w` overrides pandoc's
  default readability-focused max-width, letting the text reflow to
  the full browser width.
- **Zero footprint** - temp files live in `/tmp` (RAM-backed on most
  modern distros) and are lazily cleaned up on the next run. Nothing is
  written next to your source files.

## Requirements

- `pandoc` (3.x recommended; tested on 3.6.4)
- `xdg-utils` (for `xdg-open`)
- Optional: `chromium` or `brave-browser`, for the clean app-style
  window. Without either, `mdview` falls back to your system's default
  browser via `xdg-open`.

## Installation

```bash
sudo cp mdview /usr/local/bin/mdview
sudo chmod +x /usr/local/bin/mdview
```

### Associate `.md` files with mdview (optional)

```bash
cp mdview.desktop ~/.local/share/applications/
xdg-mime default mdview.desktop text/markdown
```

Now double-clicking a `.md` file in your file manager (Nemo, Nautilus,
Dolphin, etc.) opens it through `mdview`.

## Usage

```bash
mdview file.md              # standard GitHub-like width
mdview --wide file.md        # full browser width
mdview file.md -w            # flag order doesn't matter
```

## How it works, briefly

1. Resolves the Markdown file's real path and directory (handles
   symlinks and GVFS/FUSE mount paths via `realpath`).
2. `pandoc` renders a single HTML file in `/tmp`.
   Running pandoc inside the source directory makes relative image
   embedding reliable across regular filesystems and network mounts alike.
3. Optionally injects a small CSS override (`--wide`) that removes
   pandoc's default max-width constraint.
4. Opens the resulting HTML file in an app-style browser window if
   `chromium` or `brave-browser` is available, otherwise hands it to
   `xdg-open`.
5. On the *next* run, deletes leftover temp files from the previous
   run (lazy cleanup - there's no reliable way to know when you've
   closed the browser tab, so cleanup happens up front instead).

## What this isn't

`mdview` isn't a Markdown editor, doesn't watch files for live-reload,
and doesn't try to be a general-purpose document viewer. If you want
any of that, tools like `glow`, `frogmouth`, or dedicated GUI viewers
are a better fit.  
`mdview` is for the specific moment when you want to see a rendered
`.md` file, right now, without opening an IDE.

## License

MIT - see [LICENSE](LICENSE).
