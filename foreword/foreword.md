# Summerwave Handbook

### Welcome to Wave World!

Welcome to Summerwave! Before you start, we suggest reading the manual below.

### Basic commands

You are probably already familiar with the basic `wave` commands. To get you up to speed
faster, here are the flag-and-command combinations you are most likely to use:

| Combination | What it does | When to use it | Notes |
| --- | --- | --- | --- |
| `wave -h` | Print help | You are not sure how a command works | `wave -h --zh` prints the Chinese help |
| `wave manual --color` | Print the full manual in colour | Same, but more detail | `wave manual --zh` prints the Chinese manual |
| `wave -V` | Print the Summerwave version | Checking which version you are on | |
| `wave install <pkg_name>` | Install a package | Installing software | `wave install <pkg_name>@<pkg_version>` installs one specific version |
| `wave upgrade -X` | Upgrade a package | Everyday upgrades | Aimed at regular users. `-X, --remove-old-versions` deletes the old copies — make sure you do not need them before using it |
| `wave latest <pkg_name>` | Check whether a newer version exists than the one you have | Wondering if a package can be upgraded | `wave latest -a` shows the status of every package |
| `wave selfupdate` | Update Summerwave itself | When you want a newer Summerwave | |
| `wave list` | List the installed packages and their versions | Seeing what you have installed | `wave list -a` does the same |
| `wave info <pkg_name>` | Show which version of a package you have installed (prints `None` when it is not installed) | Not knowing whether — or which version — you installed | |
| `wave search <pkg_name>` | Search for a package (matches part of the name) | Checking whether `wave` has that package | |
| `wave uninstall <pkg_name>` | Remove a package | Uninstalling software | `wave uninstall <pkg_name>@<pkg_version>` removes one specific version |

For more, run `wave -h`, `wave -h --zh` (Chinese), `wave manual`, or `wave manual --zh` (Chinese).

Note: when you actually run these commands, replace the `<placeholder>` parts with real values.

### Install location

**There is exactly one install location per platform, and it is not configurable:**

| Platform | Location |
| --- | --- |
| macOS | `/opt/macwave` |
| Linux | `/home/linuxwave/.linuxwave` |

There is **no** `--prefix`, no custom directory, and no fallback to `~/.local`. This is a
deliberate product decision rather than an omission:

- **One tree per machine.** Two installs of the same package manager on one box is how
  you end up with a `wave` that reads one index and installs into another tree.
- **Dependencies are shared.** A single tree means each dependency exists once no matter
  how many users install packages (`Shore/` commonly holds a few hundred MB).
- **The path never depends on `$HOME`.** Renaming the account, migrating the machine, or
  running under `sudo` cannot make `wave` point somewhere else.
- **Support gets one shape to reason about.** When something is wrong, the location is
  known before the conversation starts.

`wave basedir` prints where the current install lives.

#### Permissions

**macOS** — the tree is `root:<admin group>` 775 with setgid directories. Members of the
admin group **install packages without sudo**; everything else is read-only for local
users, and nobody but the owner can replace the links in `bin/`. Installing needs **one**
administrator password.

**Linux** — the tree belongs to a dedicated `linuxwave` account and its same-named group,
so group members install packages **without sudo**, and a daemon can run under that
low-privilege account instead of root. Adding someone:
`sudo usermod -aG linuxwave <user>`, then have them log out and back in (a group only
applies to a new session) or run `newgrp linuxwave`.

#### No configuration files

There is no `config.toml`, no `~/.config/…`, and no `/opt/macwave_config`. Everything that
used to live there is either **fixed** (the install root above) or **environment-only**
(the mirror addresses below). This removes a whole class of "it works on my machine"
problems: there is nothing on disk that can disagree with the product's defaults.

### Mirrors

If you are in mainland China you most likely need a mirror: `github.com` is frequently
unreachable from mainland networks (measured 2026-11: two out of three attempts could not
even finish a TCP handshake within 20 seconds). For a full mirror, add the following
environment variables to `~/.zshrc` (or `~/.bashrc`). A good mirror will usually tell you
which commands to run:

| Variable | What it points at | Default |
| --- | --- | --- |
| `SUMMERWAVE_API_DOMAIN` | Package and dependency index | `SeaLane` on raw.githubusercontent (`…/SeaLane/HEAD`) |
| `SUMMERWAVE_BOTTLE_DOMAIN` | Prebuilt packages | GitHub Releases |
| `SUMMERWAVE_DEPS_DOMAIN` | Dependency libraries | GitHub Releases |
| `SUMMERWAVE_CORE_DOMAIN` | Summerwave's own files | `Summerwave` on raw.githubusercontent |
| `SUMMERWAVE_PYTHON_DOMAIN` | The bundled CPython | GitHub Release assets (empty = default) |
| `SUMMERWAVE_RELEASE_DOMAIN` | Which version is current | `api.github.com/repos/SummerwaveOrg/Summerwave` |
| `SUMMERWAVE_PIP_INDEX` | The `pip` index (affects only the three Python dependencies) | `https://pypi.org/simple` |

Every value is a **domain root**, and the client appends fixed paths to it — that is why a
mirror only needs to lay out the same tree:

| What is fetched | Path appended |
| --- | --- |
| Index record | `/⟨name⟩/_⟨name⟩@⟨version⟩.⟨os⟩-⟨arch⟩` |
| Prebuilt package | `/⟨name⟩/⟨name⟩@⟨version⟩.⟨os⟩-⟨arch⟩.tar.bz2` |
| Summerwave's own files | `/⟨ref⟩/⟨relative path⟩` |

**The `HEAD` segment is not optional:** the index lives at `…/SeaLane/HEAD/…` (HEAD is the
default branch). Leave it out and `wave search` / `wave latest` get nothing but 404s.

Run the following to add them (replace the host with your mirror):

```zsh
export SUMMERWAVE_API_DOMAIN=https://mirror.example.cn/Summerwave/sealane/HEAD
export SUMMERWAVE_BOTTLE_DOMAIN=https://mirror.example.cn/Summerwave/bottle
export SUMMERWAVE_DEPS_DOMAIN=https://mirror.example.cn/Summerwave/deps
export SUMMERWAVE_CORE_DOMAIN=https://mirror.example.cn/Summerwave/core
export SUMMERWAVE_PYTHON_DOMAIN=https://mirror.example.cn/Summerwave/python
export SUMMERWAVE_RELEASE_DOMAIN=https://mirror.example.cn/Summerwave/release
export SUMMERWAVE_PIP_INDEX=https://pypi.mirror.example.cn/simple
```

`export` only affects the current terminal; it is gone once you close it. Append the block
to `~/.zshrc` and every terminal inherits it from then on:

```zsh
cat >> ~/.zshrc <<'MWMIRROR'

# Summerwave mirror
export SUMMERWAVE_API_DOMAIN=https://mirror.example.cn/Summerwave/sealane/HEAD
export SUMMERWAVE_BOTTLE_DOMAIN=https://mirror.example.cn/Summerwave/bottle
export SUMMERWAVE_DEPS_DOMAIN=https://mirror.example.cn/Summerwave/deps
export SUMMERWAVE_CORE_DOMAIN=https://mirror.example.cn/Summerwave/core
export SUMMERWAVE_PYTHON_DOMAIN=https://mirror.example.cn/Summerwave/python
export SUMMERWAVE_RELEASE_DOMAIN=https://mirror.example.cn/Summerwave/release
export SUMMERWAVE_PIP_INDEX=https://pypi.mirror.example.cn/simple
MWMIRROR
source ~/.zshrc
```

`SUMMERWAVE_PIP_INDEX` only affects the three Python dependencies `pip` installs
(`requests`, `toml`, `colorama`), so it can point at any PyPI mirror (popular ones in
China include USTC and TUNA).

You only need to run this once. New terminals pick it up by themselves; the trailing
`source` merely makes the current one take effect immediately. Delete any lines you do
not need — fewer lines still work, and an unset variable falls back to the official
address.

If you later want to drop the mirror, delete or comment those lines out.

The same variables apply on Linux — `MACWAVE_*` paths are unchanged, only the names above
matter. Use `~/.bashrc` if your shell is bash.

### Contact

If something goes wrong, open an issue on GitHub or write to `macwaveorg@outlook.com`.

**Have a good surf, waver!**
