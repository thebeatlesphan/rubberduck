## First-version design

`rubberduck` is a small command-line pair-programming buddy. Its first goal is to send a
user's question-and only the context they choose-to a language model, then print the mode's reply.

The first veresion is deliberately not an autonomous agent. It will not edit files, run shell
commands, or decide on its own what the user should do next. The user stays in control of
what gets sent.

### Request flow

1. `main.rs` starts the program and passes the command-line arguments to `cli.rs`.
2. `cli.rs` turns those arguments into a command, such as `ask` or `context`.
3. The selected command gathers the question and any requested files.
4. `context/` prepares the material to include and estimates its size.
5. `prompt.rs` combines the user's question, instructions, and prepared context.
6. `model/` sends the request to the configured model server and returns its response.
7. The command prints the repsonse to the terminal.

## File responsibilities

### `src/main.rs`

The program entry point. It should parse the command-line arguments, call the appropriate command, and display any error.

Keep this file small. The work of handling each command belongs elsewhere.

### `src/cli.rs`

Defines the command-line interface: command names, options, and arguments.

For example, an `ask` command might accept a question and optional `--file` arguments. This file describes what the user can type; it should not read files or contact the model server.

### `src/config.rs`

Stores settings needed by the application, such as the model server address, model name, and context-size limit.

For the first version, defaults can be enough. Configuration files and environment-variable support can be added later.

### `src/session.rs`

Holds temporary conversation state, if the program supports more than one turn. It should contain only the current task and a short amount of useful conversation history.

The first version can omit persistent sessions. Do not save an unlimited transcript by default.

### `src/context/mod.rs`

Connects the context-related functions and types so other parts of the program can use them.

### `src/context/files.rs`

Reads files the user explicitly requested for a prompt. It should include the file path with its contents so the model can tell which code it is seeing.

It should report problems such as missing or unreadable files. The first version should not automatically include the entire project.

### `src/context/budget.rs`

Estimates how much material is being included and decides whether it exceeds the configured limit.

The estimate may be approximate; do not present it as an exact token count unless it uses the model’s tokenizer. If the selected context is too large, tell the user which files contributed to it rather than silently dropping important content.

### `src/prompt.rs`

Builds the final messages sent to the model. It should combine the assistant’s instructions, the user’s question, and selected project context in a consistent format.

The instructions should encourage pair-programming behavior: explain reasoning, ask for clarification when needed, and offer hints before jumping to a complete solution.

### `src/model/mod.rs`

Defines the shared types and interface for sending a request to a model and receiving a response.

Keeping this interface separate from the HTTP implementation makes it easier to change model servers later.

### `src/model/openai_compat.rs`

Implements communication with a model server that provides an OpenAI-compatible HTTP API. It should turn the application’s request into an HTTP request, send it, and convert the response into the application’s response type.

It should return a useful error if the server is unavailable or returns an unexpected response.

### `src/commands/mod.rs`

Makes the command handlers available to the rest of the application.

### `src/commands/ask.rs`

Implements the `ask` command. It should collect the question and selected files, prepare the context and prompt, call the model, and print the reply.

This is the main end-to-end feature of the first version.

### `src/commands/context.rs`

Implements a command that previews the context currently selected for a request. The user should be able to see which files and text would be sent before asking the model.

### `src/commands/files.rs`

Implements a command that lists files available in the project, if file discovery is included in the first version. Keep file listing separate from automatically adding files to a prompt.

### `tests/context_budget.rs`

Tests context-size estimation and behavior when the context is too large.

### `tests/prompt.rs`

Tests that prompt construction includes the question and selected context in the expected format.

```text
src/
├── main.rs          # Parse CLI args; call the selected command
├── cli.rs           # Clap command definitions
├── config.rs        # Model endpoint, model name, context limits
├── session.rs       # Current goal and short conversation history
├── context/
│   ├── mod.rs
│   ├── files.rs     # Read explicitly selected files
│   └── budget.rs    # Choose what fits; report included material
├── prompt.rs        # Build the tutor/pair-programmer prompt
├── model/
│   ├── mod.rs       # Provider trait and shared request/response types
│   └── openai_compat.rs # HTTP implementation for compatible local servers
└── commands/
    ├── mod.rs
    ├── ask.rs
    ├── context.rs
    └── files.rs

tests/
├── context_budget.rs
└── prompt.rs
```
