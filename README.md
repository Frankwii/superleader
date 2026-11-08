# superleader

A leader key for the whole desktop. One global key opens a floating window
with a tree of keys; each key runs a command, opens a submenu, a fuzzy picker,
a prompt or a live view (wifi, volume, passwords, weather). Everything is a
screen on a stack, so `h` or Esc always goes back.

It is the successor of pybinds: the window manager binds the global key, and
superleader draws in a terminal window (ratatui) instead of an X11 bar, so it
works on X11, Wayland and macOS alike.

**Status:** early design; nothing usable yet. The goals and open questions
are in [docs/IDEAS.md](docs/IDEAS.md).

## License

GPL-3.0-only. See [LICENSE](LICENSE).
