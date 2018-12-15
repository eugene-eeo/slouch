# slouch

Shitty version of rofi/dmenu using your favourite terminal
and fzf. I wanted to call it `slaunch` at first, but slouch
seems more appropriate.

## Install

```sh
$ mkdir ~/.config/slouch/
$ touch ~/.config/slouch/hooks
$ chmod +x ~/.config/slouch/hooks
```

## Run

```sh
$ slouch run     # runs executables in $PATH
$ slouch drun    # runs desktop entries
$ slouch window  # switch to windows
$ slouch filter  # xdg-open files
```

## Config

You can override the behaviour of `slouch` using a bash script in `~/.config/slouch/hooks`.
For instance, by default `slouch` assumes that you are using [herbstluftwm](https://herbstluftwm.org/).
This means that if you have a different window manager it won't work.
To override this you can add this to your slouch config file:

```sh
__slouch_focus() {
    xdotool windowfocus "$1"   # should work but needs xdotool
    i3-msg "[id=\"$1\"] focus" # i3
}
```

Usually you'd want to override `__slouch_run` and `__slouch_drun` as well,
because by default slouch assumes that you're using `st`. If you are using
`urxvt` then you might want to use:

```sh
__slouch_run() {
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

To make `slouch` look better you might have to fiddle with the rules on your
window manager. For instance on herbstluftwm:

```
hc rule title='slouch' focus=on pseudotile=on
```

### Available hooks

| Name                  | Description                                                       | Arguments |
|-----------------------|-------------------------------------------------------------------|-----------|
| `__slouch_focus`      | focus the given window                                            | `$1` = X window id |
| `__slouch_pdetach`    | run and detach the given program from the shell                   | `$@` = command to be ran |
| `__slouch_fzf`        | run fzf                                                           | `$@` = additional arguments |
| `__slouch_run`        | run a terminal which runs the slouch script with a given argument | `$1` = argument given to slouch |
| `__slouch_window_ids` | get a tab separated list of window IDs and window names           | none |
| `__slouch_drun`       | run a freedesktop entry                                           | `$1` = whether command should be ran in a terminal (empty = no) |
|                       |                                                                   | `$2` = command to be executed |
