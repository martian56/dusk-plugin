# dusk-plugin

Write plugins for the [dusk](https://github.com/martian56/dusk) editor in
Raven. dusk runs a protocol plugin as its own process and talks to it over
JSON-RPC on stdin and stdout; this package handles that, so a plugin is a
`main` that registers commands, event handlers, and key bindings.

```toml
[dependencies]
"github.com/martian56/dusk-plugin" = "v0.1.0"
```

```rust
from std.json import JsonValue
from "github.com/martian56/dusk-plugin" import Ctx, plugin, param

fun hello(ctx: Ctx, params: JsonValue) {
    ctx.message("hello from ${param(params, "path")}")
}

fun main() {
    plugin("hello")
        .command("hello.say", "Say hello", hello)
        .bind("normal", "space h", "hello.say")
        .serve()
}
```

Build it with `rvpm build` and point dusk at the binary:

```toml
[[plugin]]
name = "hello"
command = "/path/to/hello"
protocol = true
```

## API

`plugin(name)` starts a plugin. On it:

- `command(name, title, run)` adds a command. Name it `<plugin>.<command>`.
  `run` gets the context and the params dusk sent: `path`, `version`, and
  `selection`.
- `on(event, run)` handles a notification: `buffer/opened`,
  `buffer/changed`, `buffer/saved`, `buffer/closed`, or `cursor/moved`.
- `bind(mode, keys, command)` suggests a key binding. dusk keeps any binding
  the user or dusk already has.
- `serve()` runs until dusk shuts the plugin down.

A handler talks back through its `Ctx`:

- `message(text)` and `error(text)` show text in the message line.
- `status(id, text)` sets a status line item; empty text removes it.
- `buffer_text(path)` returns a buffer's text, or a file's when it is not
  open. `buffer(path)` also returns its version.
- `edit(path, version, edits)` replaces ranges as one undo step. Each edit is
  `(start_row, start_col, end_row, end_col, text)`, counted from 0. It fails
  if the buffer changed since `version`.
- `open(path, line)` opens a file at a line counted from 1.
- `register(name, title)` adds a command while running.

`param(params, key)` and `int_param(params, key, default)` read fields of the
params. The protocol itself is described in dusk's
[docs/protocol.md](https://github.com/martian56/dusk/blob/main/docs/protocol.md).

## Example

`examples/wordcount.rv` keeps a word count in the status line and adds a
`wordcount.show` command:

```sh
raven build examples/wordcount.rv -o wordcount
```

## License

MIT
