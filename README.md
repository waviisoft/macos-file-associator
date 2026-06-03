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
associator get  <type> [<type> ...]                          Show the app(s) handling these types
associator set  <type> [<type> ...] --to <app> [--role <role>]   Associate one or more types with an app
associator info <type> [<type> ...]                          Resolve type(s); show details
```

- **`<type>`** — a UTI (`net.daringfireball.markdown`, `public.plain-text`) or a
  file extension (`.md`, `md`, `markdown`). Extensions are resolved to their UTI
  automatically. **One or more** may be given to any command.
- **`--to <app>`** — the target app: a bundle id (`com.microsoft.VSCode`), an app
  name (`"Visual Studio Code"`), or a path to a `.app` bundle. (alias: `--app`)
- **`--role <role>`** — `all` (default), `viewer`, or `editor`.

Unresolvable types are skipped with a warning rather than aborting the batch, so
you can pass a large list and let the tool sort out which ones map to a UTI.

### Examples

See what opens several Markdown variants today:

```sh
$ associator get .md .markdown .mdown
.md         ->  Antigravity (com.google.antigravity)
.markdown   ->  Antigravity (com.google.antigravity)
.mdown      ->  Antigravity (com.google.antigravity)
```

Point all of them at VS Code in one call:

```sh
$ associator set .md .markdown .mdown --to "Visual Studio Code"
.md         ->  Visual Studio Code (com.microsoft.VSCode)
.markdown   ->  Visual Studio Code (com.microsoft.VSCode)
.mdown      ->  Visual Studio Code (com.microsoft.VSCode)
```

Reserve Xcode-specific types for Xcode:

```sh
$ associator set .swift .xcodeproj .xcworkspace --to Xcode --role editor
```

Inspect how an extension maps to a UTI:

```sh
$ associator info .mdx
input:      .mdx
uti:        dyn.ah62d4rv4ge8043d2
extensions: mdx
dynamic:    yes (no app/system declares this type)
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
- **Per-process cache.** LaunchServices caches handler lookups per process, so a
  `get` issued in the *same* process right after a `set` can return the old
  value. `associator set` therefore reports the app it set rather than
  re-querying; verify a change with a fresh `associator get`.
- **Competing declarations.** When several installed apps declare the same
  extension (common with VS Code "clones" like Cursor, or IDEs like Xcode), your
  explicit choice is recorded as a user default and wins, but it can take a
  LaunchServices rebuild to settle:

  ```sh
  /System/Library/Frameworks/CoreServices.framework/Frameworks/\
  LaunchServices.framework/Support/lsregister -kill -r \
  -domain local -domain system -domain user
  ```

- **Scope.** Associations are per-user.

## License

[MIT](LICENSE)
