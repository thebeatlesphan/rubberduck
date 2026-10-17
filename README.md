Suggested file structure

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
