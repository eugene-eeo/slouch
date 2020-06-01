
      ██████ ██▓    ▒█████  █    ██ ▄████▄  ██░ ██ 
    ▒██    ▒▓██▒   ▒██▒  ██▒██  ▓██▒██▀ ▀█ ▓██░ ██▒
    ░ ▓██▄  ▒██░   ▒██░  ██▓██  ▒██▒▓█    ▄▒██▀▀██░
      ▒   ██▒██░   ▒██   ██▓▓█  ░██▒▓▓▄ ▄██░▓█ ░██ 
    ▒██████▒░██████░ ████▓▒▒▒█████▓▒ ▓███▀ ░▓█▒░██▓
    ▒ ▒▓▒ ▒ ░ ▒░▓  ░ ▒░▒░▒░░▒▓▒ ▒ ▒░ ░▒ ▒  ░▒ ░░▒░▒
    ░ ░▒  ░ ░ ░ ▒  ░ ░ ▒ ▒░░░▒░ ░ ░  ░  ▒   ▒ ░▒░ ░
    ░  ░  ░   ░ ░  ░ ░ ░ ▒  ░░░ ░ ░░        ░  ░░ ░
          ░     ░  ░   ░ ░    ░    ░ ░      ░  ░  ░
                                   ░

Shitty version of rofi using your favourite terminal, bash, and fzf.
The rationale behind this being
I already have a fuzzy finder and a terminal installed,
so we should just glue them together.
This is being dogfooded everyday by me.
**V2-changes:** easier to script with, less reliant on hooks.

## install

Requires [fzf](https://github.com/junegunn/fzf) and bash.

```sh
$ mkdir ~/.config/slouch/
$ touch ~/.config/slouch/hooks
$ chmod +x ~/.config/slouch/hooks
```

For window switching, requires `xwininfo` and `xprop`.

## run

```sh
$ slouch run               # runs executables in $PATH
$ slouch drun              # runs desktop entries
$ slouch window            # switch to windows
$ slouch filter            # xdg-open files
$ cat $(ls | slouch pipe)  # fzf with a gui
```

## config

You can override parts of `slouch`
using a bash script in `~/.config/slouch/hooks`,
which will be sourced by `slouch` at runtime.
For instance, by default `slouch` assumes you
are running [herbstluftwm](https://herbstluftwm.org/);
this means that the `slouch window` command will not work
under a different WM. You can add the following hook to
your hooks file:

```sh
__slouch_focus() {
    bspc node "$1" -f          # bspwm
    i3-msg "[id=\"$1\"] focus" # i3
    xdotool windowfocus "$1"   # works across wms, but needs xdotool
}
```

Usually you'd want to override `__slouch_term` and `__slouch_drun`
as well. By default `slouch` assumes that you're using `st`.
So if you use `urxvt` then you might use:

```sh
__slouch_term() {
    urxvt -title slouch -geometry 80x20+570+300 -e $0 "$@"
}

__slouch_drun() {
    if [ "$1" ]; then
        __slouch_pdetach urxvt -e "$2"
    else
        __slouch_pdetach "$2"
    fi
}
```

You can also add default arguments passed to `fzf`;
personally I use:

```sh
SLOUCH_FZF_ARGS="--color=bw --layout=reverse --margin=1,2"
```

To make slouch look better you might have to fiddle with your wm.
For instance on herbstluftwm you can add this to your autostart:

```sh
hc rule title='slouch' focus=on pseudotile=on
```

### available hooks

| Name                  | Description                                                       | Arguments |
|-----------------------|-------------------------------------------------------------------|-----------|
| `__slouch_focus`      | focus the given window                                            | `$1` = X window id |
| `__slouch_pdetach`    | run and detach the given program from the shell                   | `$@` = command to be ran |
| `__slouch_term`       | run a terminal which runs the slouch script with a given argument | `$@` = command to be ran |
| `__slouch_fzf`        | run fzf                                                           | none (see code for details) |
| `__slouch_window_ids` | get a tab separated list of window IDs and window names           | none |
| `__slouch_drun`       | run a freedesktop entry                                           | `$1` = whether command should be ran in a terminal (empty = no), `$2` = command to be executed |
