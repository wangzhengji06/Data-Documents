# 

# Tmux

## The basic

### Create, Check and Exist

- Create tmux session: `tmux new -s <name>`
- Exit tmux: `exit`
- Check the current tmux session: `tmux ls`

### Prefix

- By default, the tmux use prefix `ctrl + B`

### Attach and Detach

-  To detach, use `<prefix> + d`
-  To reattach, use `tmux attach -t <name>`

### Kill a tmux session

- To kill a created session, `tmux kill-session -t <name>`

## Windows, Panes and Commands

### Windows

- To create window:  `<prefix> + c` 
- To view different windows: `<prefix> + p` `<prefix> + n` for previous and next 
- To go to specific window: `<prefix> <window>` 
- To close window: `exit`  Note: if all windows are closed, the tmux would just close the whole session. 
- To list windows, use `<prefix>  w`
- To rename window, use `<prefix> ,`

### Panes

- To split, `<prefix> "`
- To vsplit, `<prefix> %`
- To move around, `<prefix> <arrow-keys>`
- Or just use `<prefix> o`
- To close pane, `<prefix> x` or `exit`
- To rearrange, `<prefix> {` and `prefix }`. Or `<prefix> space` to change the layout. 

### Command

- To start a command mode, `<prefix> : `
- `list-windows`
- `split-windows` -v \ -h
- `list-commands` to just see all the commands

## Tmuxinator

- alias mux as the tmuxinator to make your life easier
- To configure the tmux, `mux new <name>`
- To start, `mux start <name>`

