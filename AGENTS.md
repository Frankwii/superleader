# Working on superleader

The project is at the design stage. Read [README.md](README.md) and
[docs/IDEAS.md](docs/IDEAS.md) first; the notes there are goals and open
questions, not a specification. pybinds is what it replaces, and
[fff](https://github.com/Frankwii/fff) is the sibling project whose fuzzy
matching, Vim key notation, config handling and event loop will be shared.

Before finishing a change, run `cargo fmt` (the code is kept formatted),
`cargo test --offline` and `cargo clippy --offline --all-targets -- -D warnings`.

Documentation describes current behavior. When behavior changes, update the
relevant document rather than appending chronological notes. Commit subjects
say what changed for users.
