# tmux

#### Commands:
- o - restart configuration
- `tmux new -s mysession` - start new session with the name
- `tmux kill-session -t mysession` - kill session with the name
- `tmux kill-session -a` - kill/delete all sessions but the current
- `tmux a -t mysession` - attach the session
- `tmux kill-server` - kill all sessions
- `tmux list-sessions` - list all tmux sessions
- `tmux swap-window -s 3 -t 1` - let window number 3 and window number 1 swap their positions.
#### Hotkeys:
- `C-a d` (detach from session)
- `C-a $` (rename session)
- `C-a w` (windows management)
- `C-a c` (create new window)
- `C-a ,` (rename current window)
- `C-a |` or `C-a -` (split screen)
- `C-a x` (kill window)
- `C-a &` (kill whole window)
- `C-a -> <-` (switch pane, arrow-right | arrow-left | ...)
- `C-a :` (run tmux-command prompt)

### Links:
- https://www.youtube.com/watch?v=nTqu6w2wc68
- https://hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/
