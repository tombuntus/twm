# twm
Tmux Workspace Manager

![twm](https://s2.gifyu.com/images/twm2.gif)


## Another one?
Yes - I was originally inspired to start using something to manage my tmux sessions by [ThePrimeagen's](https://github.com/ThePrimeagen/.dotfiles/blob/602019e902634188ab06ea31251c01c1a43d1621/bin/.local/scripts/tmux-sessionizer) tmux-sessionizer script. I used it for a bit, but wanted something more, that did a bit more.

That led me to [tmuxinator](https://github.com/tmuxinator/tmuxinator). I thought the concept of layouts was really cool, and it felt nice to use, but there are several drawbacks that are addressed by [dmux](https://github.com/zdcthomas/dmux).

Honestly I like dmux quite a bit but for some reason was annoyed by a dependency on `fzf`. After that I found the dependencyless [tmux-sessionizer](https://github.com/jrmoulton/tmux-sessionizer) that uses the Skim rust crate instead.

I felt like something for *my* workflow was missing from each of these, hence this thing.

## What features does it have?

- [x] fuzzy find user-defined workspaces to open in tmux
- [x] no external dependencies (unless you count tmux?)
- [x] define separate types of workspaces (e.g. python, node, rust, etc) to handle them differently
- [x] define global layouts to initialize your newly-opened workspaces with
- [x] default layouts optional for each workspace type
- [x] option to override default layouts with any global layout
- [x] local layouts within workspace directories to take precedence over defaults
- [x] option to open an arbitrary directory as a new workspace
- [x] blazingly fast (in spite of my code not because of it - rust!)
- [x] will open tmux properly whether it is already running or not
- [ ] tests (coming soon i swear)
- [ ] search up directories to see if you can inherit local layout from a parent (useful for worktrees)
- [ ] support for more complex matching (e.g. regex, glob, combined include/exclude lists)
- [ ] just general code improvements i hope


I've tried to make sure to not add anything I can easily accomplish with other tools or simple scripts, e.g. `tmux-sessionizer` has builtin support for Git Worktrees, but I didn't like how it was implemented.

The example config will have each worktree show up as a separate workspace

## Usage
```
A utility for managing workspaces in tmux

Usage: twm [OPTIONS]

Options:
  -l, --layout       Prompt user to select a layout to open the workspace with
  -p, --path <PATH>  Open the given path as a workspace
  -h, --help         Print help
  -V, --version      Print version
```

## Installation
The easiest way to install is to use cargo
```bash
cargo install twm
```

It will be available as `twm` in your path.

Coming to NixOS soon!

## Configuration

Your config file should be located at $XDG_CONFIG_HOME/twm/twm.yaml (default: ~/.config/twm/twm.yaml).

`twm` should run with some sensible defaults if you don't have a config file, but it certainly won't work for everyone.


### Example config

Here is the example config I used to while developing
```yaml
# ~/.config/twm/twm.yaml

search_paths:  # directories we should begin searching for workspaces in. i just use home. shell expansion is supported
    - "~"      # default: ["~"]

exclude_path_components:  # search branches will be pruned if they contain any of these substrings
  - /.git
  - /.direnv
  - /.nix-
  - node_modules
  - /venv
  - /target

max_search_depth: 5  # how deep we should search for workspaces (default: 3)

workspace_definitions:             # our list of workspaces, each with different properties
    - name: python                 # they all have to be named
      has_any_file:                # if any file matches this list, we consider it a match, since its "has_any_file"
        - requirements.txt         # more complex matching isn't implemented currently
        - setup.py
        - pyproject.toml
        - Pipfile
      default_layout: python-dev   # the hierarchy for how a layout gets chosen is user opts to select manually > local layout > default for workspace type

    - name: node                   # the order of these definitions matters - if a directory matches multiple, the first one wins
      has_any_file:
        - package.json
        - yarn.lock
        - .nvmrc
      default_layout: node-dev

    - name: rust
      has_any_file:
        - Cargo.toml
        - Cargo.lock
      default_layout: rust-dev

    - name: other
      has_any_file:
        - .git
        - flake.nix
        - .twm.yaml

layouts:                           # our list of layouts just have names and a list of commands. the command get sent directly with tmux send-keys
    - name: python-dev             # i chose not to use any custom configuration becuase that would be a lot of work to basically maintain a subset of possible functionality
      commands:
        - tmux split-window -h
        - tmux resize-pane -x 80
        - tmux split-window -v
        - tmux send-keys -t 0 'nvim .' C-m

    - name: rust-dev
      commands:
        - tmux split-window -h
        - tmux resize-pane -x 80
        - tmux select-pane -t 0
        - tmux send-keys -t 1 'cargo watch -x test -x run' C-m
        - nvim .

    - name: python-debugger
      commands:
        - tmux split-window -h
        - tmux resize-pane -x 80
        - tmux split-window -v
        - tmux send-keys -t 0 'nvim .' C-m
        - tmux send-keys -t 1 'python -m pdb' C-m
```

### Example local config

```yaml
# ~/dev/random/project/dir/.twm.yaml

layout:
  name: layout-for-this-project
  commands:
    - tmux split-window -h
    - tmux split-window -h
    - tmux split-window -h
    - tmux split-window -h
```

## Contributing

If for some reason you want to contribute to this, just be warned, this is the first thing I've written in rust.

As long as your changes don't go against anything I've said in this README, I'd imagine I'll be happy to merge them.

Just use the standard cargo fmt and clippy. CI should tell you if you've messed it up.

The clippy lint I've been using is `cargo clippy -- -D clippy::pedantic -A clippy::missing_errors_doc`

## License

GPL v2.0



