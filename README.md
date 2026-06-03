# macos-file-associator

A tiny, dependency-free CLI for inspecting and changing **default app
associations** on macOS — e.g. making `.md` files open in VS Code instead of
whatever currently claims them.

It talks directly to the macOS **LaunchServices** API. There is nothing to
install: it runs on the `swift` interpreter that ships with the Xcode Command
Line Tools (`xcode-select --install`).

## License

[MIT](LICENSE)
