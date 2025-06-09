# Ansiblx (Ansible + Mitogen) Multi‐Version Manager

This repository provides two scripts to manage multiple Ansible/Mitogen virtual environments and switch between them seamlessly:

1. **`ansiblx-installer`**
   
   Creates isolated Python virtual environments (venvs) for specific Ansible + Mitogen versions.

3. **`ansiblx`**
   
   Switches your active Ansible installation (and configuration) to one of the created venvs by updating `~/.ansible.cfg` and recreating `ansible*` symlinks in `~/.local/bin`.

---

## Table of Contents

- [Ansiblx (Ansible + Mitogen) Multi‐Version Manager](#ansiblx-ansible--mitogen-multiversion-manager)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
    - [1. ansiblx-installer](#1-ansiblx-installer)
    - [2. ansiblx](#2-ansiblx)
  - [Usage](#usage)
    - [Creating venvs with ansiblx-installer](#creating-venvs-with-ansiblx-installer)
    - [Switching versions with ansiblx](#switching-versions-with-ansiblx)
  - [How It Works](#how-it-works)
  - [Directory Layout](#directory-layout)
  - [Adding a Custom Config](#adding-a-custom-config)
  - [Cleanup](#cleanup)
  - [Troubleshooting](#troubleshooting)

---

## Prerequisites

* **macOS or Linux** with Bash.
* **Python 3** and `python3 -m venv`.
* **`pip`** (bundled with Python).
* **`fzf`** (optional, for interactive menus). If not installed, menus fall back to plain‐text prompts.
* `~/.local/bin` must be on your `PATH`. (The switcher recreates `ansible*` symlinks there.)

---

## Installation

### 1. ansiblx-installer

Simply download the latest `ansiblx-installer` script into any directory, make it executable, and run it:

```bash
# Download:
curl -Lo ansiblx-installer https://raw.githubusercontent.com/ishad0w/ansiblx/refs/heads/main/ansiblx-installer
chmod +x ansiblx-installer
```

No further setup is needed. You can now invoke `./ansiblx-installer` from current directory.

### 2. ansiblx

Download `ansiblx` into `~/.local/bin` (or another folder already in your `PATH`), then make it executable:

```bash
# Download:
curl -Lo ~/.local/bin/ansiblx https://raw.githubusercontent.com/ishad0w/ansiblx/refs/heads/main/ansiblx
chmod +x ~/.local/bin/ansiblx

# Ensure ~/.local/bin is in your PATH (if not already):
export PATH="$HOME/.local/bin:$PATH"
```

---

## Usage

### Creating venvs with ansiblx-installer

```bash
# Create only Ansible 10.6.0 (Mitogen 0.3.18, ansible-core 2.17.6, ansible-lint 24.9.2)
./ansiblx-installer 10

# Create only Ansible 11.6.0 (Mitogen 0.3.24, ansible-core 2.18.6, ansible-lint 25.4.0)
./ansiblx-installer 11

# Create both version-10 and version-11 venvs
./ansiblx-installer --all

# You can also specify both explicitly:
./ansiblx-installer 10 11
```

After running, you’ll have one or two new directories under `~/.venvs/`:

```
~/.venvs/
├── ansible-mitogen-10.6.0/
│     ├── ansible.cfg
│     ├── ansible-non-mitogen.cfg
│     ├── bin/
│     │    ├── ansible
│     │    ├── ansible-playbook
│     │    ├── ansible-lint
│     │    └── …other ansible-* scripts
│     └── lib/…
└── ansible-mitogen-11.6.0/
      ├── ansible.cfg
      ├── ansible-non-mitogen.cfg
      ├── bin/
      │    ├── ansible
      │    ├── ansible-playbook
      │    ├── ansible-lint
      │    └── …other ansible-* scripts
      └── lib/…
```

* Each venv contains:

  * `ansible.cfg` (Mitogen‐enabled)
  * `ansible-non-mitogen.cfg` (without `strategy_plugins` lines)

### Switching versions with ansiblx

```bash
ansiblx
```

1. If `fzf` is installed, an interactive menu appears:

   ```
   ansiblx> 
   ┌───────────────────────────────────────┐
   │ ansible-mitogen-10.6.0                │
   │ ansible-mitogen-10.6.0 (non-mitogen)  │
   │ ansible-mitogen-10.6.0 (user-customized-cfg)   ← if ansible-customized.cfg exists
   │ ansible-mitogen-11.6.0                │
   │ ansible-mitogen-11.6.0 (non-mitogen)  │
   │ ansible-mitogen-11.6.0 (user-customized-cfg)   ← if ansible-customized.cfg exists
   └───────────────────────────────────────┘
   ```

2. If `fzf` is not installed, it falls back to a plain-text list and prompts for exact input.

3. Once you choose one, the script automatically:

   * Updates `~/.ansible.cfg` to symlink to the selected config inside that venv.
   * Removes any old `ansible*` symlinks in `~/.local/bin`.
   * Creates new symlinks in `~/.local/bin` pointing to every `ansible*` binary inside the chosen venv’s `bin/` directory.
   * Prints a confirmation with:

     * Which venv was chosen (and whether it’s “non-mitogen” or “user-customized-cfg”).
     * The actual path of `~/.ansible.cfg`.
     * The versions of `ansible` and `ansible-playbook` now active.

After switching, simply running `ansible --version` or `ansible-playbook --version` will use the binaries from your chosen venv.

---

## How It Works

1. **`ansiblx-installer`**

   * Parses one or more arguments: `10`, `11`, or `--all`.
   * For each requested version, creates a venv under `~/.venvs/ansible-mitogen-<ANSIBLE_VER>`.
   * Installs fixed versions of `mitogen`, `ansible-core`, `ansible`, and `ansible-lint`.
   * Writes two config files inside that venv:

     * `ansible.cfg` (Mitogen strategy enabled)
     * `ansible-non-mitogen.cfg` (no Mitogen lines)
   * Deactivates and confirms.

2. **`ansiblx`**

   * Ensures `~/.local/bin` exists; warns if it’s not in `PATH`.
   * Scans `~/.venvs` for `ansible-mitogen-*` directories.
   * Builds a menu offering “with Mitogen,” “non-mitogen,” or “user-customized-cfg” (if present).
   * Updates `~/.ansible.cfg` to point at the chosen config inside that venv.
   * Removes any previous `ansible*` symlinks in `~/.local/bin` and recreates them so that typing `ansible` or `ansible-playbook` invokes the binaries from the chosen venv.

---

## Directory Layout

```
~/
├── .ansible.cfg               # Symlink → chosen venv’s config
├── .local/
│   └── bin/
│        ├── ansiblx           # Switcher script
│        ├── ansible           -> /home/you/.venvs/ansible-mitogen-11.6.0/bin/ansible
│        ├── ansible-playbook  -> /home/you/.venvs/ansible-mitogen-11.6.0/bin/ansible-playbook
│        └── …other ansible* symlinks
└── .venvs/
    ├── ansible-mitogen-10.6.0/
    │     ├── ansible.cfg
    │     ├── ansible-non-mitogen.cfg
    │     ├── bin/
    │     │    ├── ansible
    │     │    ├── ansible-playbook
    │     │    ├── ansible-lint
    │     │    └── …other ansible-* scripts
    │     └── lib/…
    └── ansible-mitogen-11.6.0/
          ├── ansible.cfg
          ├── ansible-non-mitogen.cfg
          ├── bin/
          │    ├── ansible
          │    ├── ansible-playbook
          │    ├── ansible-lint
          │    └── …other ansible-* scripts
          └── lib/…
```

---

## Adding a Custom Config

To maintain a hand‐tuned configuration inside a venv:

1. Create `ansible-customized.cfg` inside the desired venv, e.g.:

   ```
   ~/.venvs/ansible-mitogen-11.6.0/ansible-customized.cfg
   ```

   Customize any `[defaults]` settings you like (copy from `ansible.cfg` and tweak).

2. Next time you run `ansiblx`, you will see:

   ```
   ansible-mitogen-11.6.0 (user-customized-cfg)
   ```

   Choose it to symlink `~/.ansible.cfg` to your custom file.

---

## Cleanup

* **Remove a venv**

  ```bash
  rm -rf ~/.venvs/ansible-mitogen-10.6.0
  ```

  That version won’t appear in `ansiblx` anymore. Remove leftover symlinks in `~/.local/bin`:

  ```bash
  rm ~/.local/bin/ansible*
  ```

* **Remove all venvs**

  ```bash
  rm -rf ~/.venvs/ansible-mitogen-*
  rm ~/.ansible.cfg
  rm ~/.local/bin/ansible*
  ```

---

## Troubleshooting

* **`ansiblx: command not found`**
  Ensure `~/.local/bin/ansiblx` exists and is executable, and that `~/.local/bin` is in your `PATH`:

  ```bash
  export PATH="$HOME/.local/bin:$PATH"
  ```

* **`No “ansible-mitogen-*” venvs under ~/.venvs`**
  Run `ansiblx-installer 10` or `11` (or `--all`) first to create at least one venv.

* **`fzf not found`**
  You can install `fzf` with your OS package manager (e.g. `brew install fzf` on macOS, `sudo apt install fzf` on Ubuntu). Otherwise, you’ll be prompted to type the exact venv name.
