# CLI Tools

> Arch / Arch-based. Install everything at once:

```bash
pacman -S ripgrep-all bat eza btop lazygit tldr zoxide
yay -S lazydocker yazi
```

---

## Tools

### `bat` — better `cat`

Syntax-highlighted file viewer with line numbers, git diff markers, and automatic paging.

```bash
bat file.py
bat --plain file.txt   # no decorations
bat -A file            # show non-printable characters
```

> Drop-in replacement for `cat`. Add `alias cat=bat` to your shell config.

---

### `eza` — better `ls`

Modern `ls` replacement with colours, icons, and git status.

```bash
eza -la                # long list, all files
eza --tree             # directory tree
eza -la --git          # show git status per file
eza --icons -la        # file-type icons (requires a Nerd Font)
```

> Suggested aliases:
>
> ```bash
> alias ls='eza --icons'
> alias ll='eza -la --icons'
> alias lt='eza --tree --icons'
> ```

---

### `ripgrep-all` (`rga`) — search inside anything

`rga` extends `ripgrep` to search inside PDFs, archives, Office docs, and more.

```bash
rga "password" .            # search current directory recursively
rga "TODO" src/             # search a specific path
rga -i "error" logs/        # case-insensitive
rga --type pdf "keyword"    # search only PDF files
```

---

### `btop` — system monitor

Interactive resource monitor: CPU, memory, disks, network, and processes in one view.

```bash
btop
```

| Key        | Action                |
| ---------- | --------------------- |
| `q`        | Quit                  |
| `F2` / `o` | Options menu          |
| `k`        | Kill selected process |
| `←` / `→`  | Switch between views  |

> Config file: `~/.config/btop/btop.conf`

---

### `tldr` — practical man pages

Community-maintained command examples — gets straight to usage instead of exhaustive reference.

```bash
tldr tar
tldr git commit
tldr --update           # refresh the local cache
```

---

### `zoxide` — smarter `cd`

Learns your most-visited directories and jumps to them with partial matches.

```bash
# Add to ~/.zshrc (or ~/.bashrc)
eval "$(zoxide init zsh)"
```

```bash
z projects       # jump to the directory matching "projects"
z dot conf       # multi-word match
zi               # interactive fuzzy picker (requires fzf)
```

> After init, `z` replaces `cd`. The more you use it the smarter it gets.

---

### `yazi` — terminal file manager

Fast, async file manager with previews (images, text, archives) powered by `bat`, `ffmpegthumbnailer`, and others.

```bash
yazi
```

| Key       | Action                  |
| --------- | ----------------------- |
| `h` / `l` | Navigate parent / enter |
| `j` / `k` | Move down / up          |
| `Space`   | Select file             |
| `y`       | Yank (copy)             |
| `p`       | Paste                   |
| `d`       | Delete                  |
| `r`       | Rename                  |
| `q`       | Quit                    |

> Config: `~/.config/yazi/`

---

### `lazygit` — terminal Git UI

Full Git workflow — stage, commit, push, diff, branches, stash — without leaving the terminal.

```bash
lazygit
```

| Key     | Action               |
| ------- | -------------------- |
| `1`–`5` | Switch panels        |
| `Space` | Stage / unstage file |
| `c`     | Commit               |
| `P`     | Push                 |
| `p`     | Pull                 |
| `b`     | Branch menu          |
| `?`     | Keybinding help      |
| `q`     | Quit                 |

> Add alias: `alias lg='lazygit'`

---

### `lazydocker` — terminal Docker UI

Manage containers, images, volumes, and logs without typing `docker` commands.

```bash
lazydocker
```

| Key       | Action                    |
| --------- | ------------------------- |
| `[` / `]` | Switch panels             |
| `Enter`   | View logs / inspect       |
| `r`       | Restart container         |
| `d`       | Remove container or image |
| `u`       | Pull latest image         |
| `q`       | Quit                      |

> Requires Docker to be running. See [Docker Setup](../tools/Docker.md).

---

## What to Read Next

| You want to...                | Go here                            |
| ----------------------------- | ---------------------------------- |
| Set up Docker                 | [Docker Setup](../tools/Docker.md) |
| Go back to Linux fundamentals | [Linux](README.md)                 |
