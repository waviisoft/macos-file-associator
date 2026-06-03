# macos-file-associator

A tiny, dependency-free CLI for inspecting and changing **default app
associations** on macOS — e.g. making `.md` files open in VS Code instead of
whatever currently claims them.

It talks directly to the macOS **LaunchServices** API. There is nothing to
install: it runs on the `swift` interpreter that ships with the Xcode Command
Line Tools (`xcode-select --install`).

## Why

macOS associates files with apps by **content type** (a Uniform Type
Identifier, or UTI) rather than by file extension. There's no built-in
command-line way to query or change these associations — you normally have to
right-click a file → *Get Info* → *Open with* → *Change All…*. This tool does
the same thing scriptably.

## Install

```sh
git clone https://github.com/waviisoft/macos-file-associator.git
cd macos-file-associator
chmod +x associator            # already executable in the repo
# optional: put it on your PATH
ln -s "$PWD/associator" /usr/local/bin/associator
```

## Usage

```
associator get  <type>                Show the app associated with a content type
associator set  <type> <app> [role]   Associate a content type with an app
associator info <type>                Resolve a type/extension and show details
```

- **`<type>`** — a UTI (`net.daringfireball.markdown`, `public.plain-text`) or a
  file extension (`.md`, `md`, `markdown`). Extensions are resolved to their UTI
  automatically.
- **`<app>`** — a bundle id (`com.microsoft.VSCode`), an app name
  (`"Visual Studio Code"`), or a path to a `.app` bundle.
- **`<role>`** — `all` (default), `viewer`, or `editor`.

### Examples

See what opens Markdown files today:

```sh
$ associator get .md
type:       net.daringfireball.markdown
handler:    Antigravity (com.google.antigravity)
```

Make Markdown open in VS Code:

```sh
$ associator set .md "Visual Studio Code"
set net.daringfireball.markdown -> Visual Studio Code (com.microsoft.VSCode)
```

Inspect how an extension maps to a UTI:

```sh
$ associator info .mdx
input:      .mdx
uti:        dyn.ah62d4rv4ge8043d2
extensions: mdx
dynamic:    yes (no app/system declares this type)
type:       dyn.ah62d4rv4ge8043d2
handler:    (none)
```

## Notes & caveats

- **Dynamic UTIs.** Extensions that no installed app or the system formally
  declares (e.g. `.mdwn`, `.mkd`) get a synthesized `dyn.…` UTI. These are
  derived from the extension and are stable on a given machine, but they are not
  portable identifiers — don't hard-code them across systems.
- **Markdown is split across several UTIs.** `.md` and `.markdown` share
  `net.daringfireball.markdown`, but `.mdown`, `.mkd`, `.mdwn`, and `.mdx` each
  resolve to their own dynamic UTI. To reroute all of them you must `set` each
  type.
- **Propagation.** Changes apply immediately to newly opened files, but Finder
  icons may take a moment (or a relaunch) to update.
- **Scope.** Associations are per-user.

## License

[MIT](LICENSE)
