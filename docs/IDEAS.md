# Design notes

Carried over from the project brainstorm (November 2026). These are goals
and open questions, not a specification.

A Rust rewrite of pybinds that shows in a floating window instead of an X11
override-redirect bar, so it survives a move to Wayland or a Mac. The window
manager binds the global key, as sxhkd does now.

**Nodes are more than commands:**

- **run**: what pybinds does today.
- **input**: a prompt (replaces `readinput`).
- **pick**: a fuzzy list fed by a command or a built-in source (replaces every
  `rofi -dmenu`, `rofi -show window/drun/run`, and the `kb-custom` exit-code
  trick in `rofi-ff`).
- **view**: a live panel closed with `q` (weather, wifi, volume, calculator).

**Lessons from pybinds2** (what to avoid, not a spec to copy):

- Children pass typed values to the parent instead of `___` string substitution
  into shell, which is why the config is full of nested escaped quotes.
- Real modifiers and Vim notation (`<C-x>`) from fff's keymap; pybinds only
  knows Shift.
- A grid layout when there are many children; the one-row bar runs off-screen.
- Show live state in the window (e.g. "Volume: 45%") instead of `notify-send`
  with a 500 ms timeout.
- Dynamic children from a command (pass entries, wifi networks, browser tabs)
  instead of hand-written YAML subtrees.
- An exception in a command must not kill the daemon.

**Everything is a screen on a stack.** A static submenu, a picker, a prompt
and a live view are all the same thing: a screen with items, keys, and a
refresh policy (never, on open, every N seconds, on an event). Any action can
push another screen; `h`/Esc pops back. So views are interactive like
submenus:

- **Wifi:** live list of networks. Enter connects, and if the network is
  secured, pushes a password prompt. `f` forgets, `d` disconnects.
- **pass:** pick an entry, which pushes a submenu of its fields: `p` password,
  `u` username, `o` OTP, `e` edit.
- **Volume:** a live bar; `j`/`k` adjust and stay open, `m` mutes.

The bottom row always shows the keys available on the current screen, in the
same style as the tree, so a view never feels like a different program.

**Where views come from:**

- **Built in (Rust)** for the core ones that need an API: wifi (iwd's D-Bus
  API), pass, volume.
- **A small script protocol** for personal one-offs: the script prints items
  and their actions as JSON lines, and the daemon tells it which action was
  chosen. A new view is a short script plus one config entry.

**Dynamic subtrees.** A node can generate its children when opened, from a
built-in source or a script, and generated children can generate their own,
to any depth. Good fits are things that are already trees:

- the password store (`mail/`, `code/`, `personal_data/` become submenus)
- Firefox bookmark folders
- projects in `~/phd` (open in Neovim, fff, zathura, or as a session)
- i3 workspaces and their windows, audio outputs

The hard design question is **which key each generated item gets**, because
muscle memory is the point of the interface:

- **Stable keys:** once an item gets a key, remember it (persisted), so
  `w p m g` means the same thing next month even when new entries appear.
  New items take free keys; removed items free theirs.
- **Pinning:** the config can fix a key for a generated item.
- **Too many children** (say over ~30): the screen becomes a fuzzy picker you
  type into instead of keys.

Generating runs in the background with a timeout and a spinner; errors are
shown on the screen instead of killing anything. Results can be cached
until the source changes (inotify on the pass store, for example).

Making it a full file explorer is possible but fff already is one; a dynamic
"places" subtree that opens directories in fff gets the useful part.

**Config: still to design.** Don't copy pybinds' YAML. Goals: declarative, no
shell-quoting gymnastics, reusable subtrees (what the YAML anchor
`&writables` does today), hot reload. One option is fff's hybrid approach:
a text file that's editable by hand, plus editing inside the app. That's
natural here, because the UI already is the tree: "add a child here", "rebind
this key", "rename" right where you are, written back with `toml_edit` so
comments and layout survive (fff already does this).

**Rendering:** ratatui. Start with an st window kept in the i3 scratchpad, so it
appears instantly. A native window drawing the same ratatui buffer is possible
later.

**Views to build on it:**

- **pass:** fuzzy search over `~/.password-store`, copy with a timed clipboard
  clear (like `clipcopy`), other fields with another key, OTP with a countdown,
  password generation, git sync status. Replaces the hand-written pass subtree
  in `01clipboard.yaml`.
- **Wifi:** iwd's D-Bus API (`net.connman.iwd`) instead of parsing `iwctl`'s
  table with awk. Replaces `rofi-wifi`.
- **Weather:** Open-Meteo (no API key) for a real hourly forecast instead of
  `curl wttr.in`. (`clima` still defaults to Barcelona.)
- **Calculator:** live `qalc`.
- **Firefox omnibox:** bookmarks, history and open tabs (brotab), keyword
  searches from `moz_keywords`, live DuckDuckGo suggestions. Replaces
  `query_bookmarks`, `select_browser_tab` and the search nodes in
  `00browser.yaml`.
- **Offline dictionary:** Wiktionary data from kaikki.org (English, Spanish,
  Italian: definitions, translations, conjugations), searched as you type, with
  a key to open WordReference. Replaces `wrwrapper`.

**Bug to fix in passing:** `query_bookmarks` copies `places.sqlite` without
`places.sqlite-wal`, so recent bookmarks can be missing while Firefox runs.
Open it with `?immutable=1` or copy the WAL file too.

## Later: context-aware key tree

When the tree opens, it checks which window is focused and shows actions for
it first:

- **zathura:** open the matching `.tex` in Neovim (synctex), copy the current
  page as an image.
- **Firefox on an arXiv or journal page:** download the PDF into my library.
- **A terminal:** open fff here, or a new terminal in the same directory.

It's not tied to i3. It only needs the focused window and its working
directory, behind one small interface per platform:

| Platform | Focused window | Working directory |
|---|---|---|
| i3 / sway (same IPC) | `get_tree` | `/proc/<pid>/cwd` |
| any X11 window manager | `_NET_ACTIVE_WINDOW` + `_NET_WM_PID` | same |
| Hyprland | `hyprctl activewindow -j` | same |
| macOS | `NSWorkspace.frontmostApplication` + Accessibility API | `proc_pidinfo` |

On a Mac: skhd for the hotkey, AeroSpace as an i3-like window manager.

