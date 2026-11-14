# apogee-console

The command line of an [Apogee](https://github.com/venomous-maker) app, as
Laravel's Artisan. It has:

- commands with Laravel's signatures, whose `handle()` the container fills in
- styled output, tables, progress bars and Laravel's components
- questions, which tests can answer in advance
- lookup by name, alias or abbreviation, with "did you mean" suggestions
- `list`, `help`, bash completion, isolated commands and events

```grav
import "std"
import "apogee-console"

class SendMail extends ConsoleCommand {
    constructor() {}
    fn signature() -> string { return "mail:send {user : The user's id} {--Q|queue=default : The queue} {--force}" }
    fn description() -> string { return "Send the welcome mail" }
    fn handle(mailer: Mailer) -> int {
        if (!self.flag("force") && !self.confirm("Send it?", true)) { return 1 }
        mailer.send(self.argumentInt("user"), self.option("queue"))
        self.info("Sent.")
        return 0
    }
}

fn main() {
    let console = new ConsoleKernel(container)
    console.discover()                                  // every ConsoleCommand in the program
    console.command("inspire", fn(c: ClosureCommand) -> int { c.comment("Simplicity is the ultimate sophistication."); return 0 })
        .describe("Display an inspiring quote")
    Process.exit(console.run(Process.args()))
}
```

## Signatures

```
mail:send {user} {user?} {user=ada} {files*} {files?*}            arguments
          {--force} {--queue=} {--queue=default} {--tag=*} {--Q|queue=}   options
          {user : The user's id}                                   descriptions
```

On the command line:

- **Option values:** `--queue=high` or `--queue high`; `-Q high` or `-Qhigh`.
- **Flags:** `-abc` sets several at once, and `--no-force` turns a flag off.
- **End of options:** after `--`, everything is an argument.
- **Mistakes:** each gets an error that says what was meant, such as
  `the option '--forse' does not exist. Did you mean --force?`,
  `not enough arguments (missing: "user")`, or "too many arguments", followed
  by the command's usage line.

In `handle()`:

- **Arguments:** `argument`, `argumentInt`, `argumentList`.
- **Options:** `option`, `optionInt`, `optionFloat`, `optionList`, `flag`,
  `hasOption`.

## Output

```grav
self.info("Done.")  self.comment("…")  self.warn("…")  self.error("…")  self.line("…", "question")
self.output.writeln("<info>green</> <comment>yellow</> <fg=red;bg=white;options=bold>loud</> \\<literal>")
self.table(["Name", "Email"], rows)                    // "default" (+-|), "box", "compact", "markdown"
self.task("Migrating", fn() -> bool { return migrate() })        //   Migrating .......... 12ms DONE
self.twoColumn("Cache", "<info>redis</>")
self.output.infoBlock("Ready.")  warnBlock  errorBlock  successBlock  alert("Careful")
let bar = self.progressBar(rows.size()); bar.start(); … bar.advance() …; bar.finish()
self.withProgressBar(items, fn(x: any) { process(x) })
```

- **Colour:** tags become colours only when stdout is a terminal and
  `NO_COLOR` is unset. Elsewhere they're dropped.
- **Verbosity:** `-q` silences output. `-v`, `-vv` and `-vvv` show
  `self.output.verbose(…)` and `debug(…)` lines.
- **Width:** tables count characters, not bytes, so accented text lines up.
- **Progress bars:** on a terminal a bar redraws in place, at most once per
  percent. Elsewhere it writes a line every 10%.

## Questions

```grav
let name = self.ask("Name?", "ada")
let pw = self.secret("Password?")                     // not shown as it is typed
let sure = self.confirm("Continue?", false)
let role = self.choice("Role?", ["admin", "editor", "viewer"], 2)    // by number or text
let lang = self.anticipate("Language?", ["grav", "go", "rust"])     // "gr" -> "grav"
```

With `-n` (`--no-interaction`), every question takes its default.

## Running and finding commands

- **Running:** `console.run(Process.args())` gives the exit code. `handle()`
  can return an `int`, a `bool` (true becomes 0), or nothing (0). An exception
  becomes an error block and exit code 1.
- **Lookup:** a command is found by its exact name, an alias, or an
  abbreviation of each part (`m:c` finds `make:controller`). An ambiguous or
  unknown name lists what it might mean.
- **Global options:** `-h`, `-q`, `-v`/`-vv`/`-vvv`, `-n`, `--ansi` /
  `--no-ansi`, `-V`, and `--env=NAME`.
- **Built-in commands:**
  - `list [namespace] [--raw]`, grouped by namespace
  - `help <command>`
  - `completion`, which prints a bash script: `eval "$(app completion)"`
- **Isolated commands:** a command whose `isolated()` is true runs once at a
  time, through a cache lock when `console.cache` is set. A second run exits
  at once.
- **Calling other commands:** `self.call("cache:clear --tag=x")` and
  `callSilently(…)`.
- **Events:** `CommandStarting` and `CommandFinished` (with the exit code and
  time) when `console.events` is set.

## Testing

```grav
assertEq(console.call("mail:send 7 --force"), 0)
assertEq(console.output(), "Sent.\n")
console.call("make:wizard", ["Ada", "s3cret", "1"])    // the answers, in order
```

## Tests

```
grav test tests/console.grav
```
