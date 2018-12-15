# slouch

Shitty version of rofi/dmenu using your favourite terminal, bash, and fzf.
The rationale behind this is that
I already have a fuzzy finder and a terminal installed so we should just glue them together.
I wanted to call it `slaunch` at first, but slouch seems more appropriate.

## install

Requires [fzf](https://github.com/junegunn/fzf) and bash.

```sh
$ mkdir ~/.config/slouch/
$ touch ~/.config/slouch/hooks
$ chmod +x ~/.config/slouch/hooks
```

## run

```sh
$ slouch run     # runs executables in $PATH
$ slouch drun    # runs desktop entries
$ slouch window  # switch to windows
$ slouch filter  # xdg-open files
```

## config

You can override parts of `slouch` using a bash script in `~/.config/slouch/hooks`.
This 'config file' is just sourced by `slouch`.
For instance, by default `slouch` assumes that you are using [herbstluftwm](https://herbstluftwm.org/).
This means that if you have a different window manager then focusing windows won't work.
To override this you can add this to your slouch config file:

```sh
__slouch_focus() {
    bspc node "$1" -f          # bspwm
    i3-msg "[id=\"$1\"] focus" # i3
    xdotool windowfocus "$1"   # should across multiple wms, but needs xdotool
}
```

Usually you'd want to override `__slouch_term` and `__slouch_drun` as well.
By default slouch assumes that you're using `st`.
So if you use `urxvt` then you might do:

```sh
__slouch_term() {
    urxvt -title slouch -geometry 80x20+570+300 -e "$0" "$1"
}

__slouch_drun() {
    if [ "$1" ]; then
        __slouch_pdetach urxvt -e "$2"
    else
        __slouch_pdetach "$2"
    fi
}
```

To configure fzf-specific stuff you can override `__slouch_fzf`.
E.g. if you want to add previews for files:

```sh
__slouch_fzf() {
    if [ "$1" = 'filter' ]; then
        fzf --preview='cat {} || tree {}' $2
    else
        fzf $2
    fi
}
```

To make `slouch` look better you might have to fiddle with your window manager.
For instance on herbstluftwm you can add this to your autostart:

```sh
hc rule title='slouch' focus=on pseudotile=on
```

### available hooks

| Name                  | Description                                                       | Arguments |
|-----------------------|-------------------------------------------------------------------|-----------|
| `__slouch_focus`      | focus the given window                                            | `$1` = X window id |
| `__slouch_pdetach`    | run and detach the given program from the shell                   | `$@` = command to be ran |
| `__slouch_fzf`        | run fzf                                                           | `$1` = mode (window/drun/run/filter), `$2` = additional arguments |
| `__slouch_term`       | run a terminal which runs the slouch script with a given argument | `$1` = argument to be forwarded |
| `__slouch_window_ids` | get a tab separated list of window IDs and window names           | none |
| `__slouch_drun`       | run a freedesktop entry                                           | `$1` = whether command should be ran in a terminal (empty = no), `$2` = command to be executed |
