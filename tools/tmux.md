# tmux

### Commands:
- `tmux source-file ~/.tmux.conf`- restart configuration
- `tmux new -s mysession` - start new session with the name
- `tmux kill-session -t mysession` - kill session with the name
- `tmux kill-session -a` - kill/delete all sessions but the current
- `tmux a -t mysession` - attach the session
- `tmux kill-server` - kill all sessions
### Hotkeys:
- `C-a d` (detach from session)
- `C-a $` (rename session)
- `C-a w` (windows management)
- `C-a c` (create new window)
- `C-a ,` (rename current window)
- `C-a |` or `C-a -` (split screen)
- `C-a x` (kill window)
- `C-a &` (kill whole window)
- `C-a -> <-` (switch pane, arrow-right | arrow-left | ...)

### Links:
- https://www.youtube.com/watch?v=nTqu6w2wc68
- https://hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/
