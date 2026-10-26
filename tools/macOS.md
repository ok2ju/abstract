# Keep Your Mac Awake

The simplest Mac solution is the built-in `caffeinate` command. It prevents the system from sleeping for as long as it runs — no app install required.

To prevent sleep indefinitely until you cancel:
```
caffeinate
```

To keep the system awake only while a specific process is running:
```
caffeinate -w <PID>
```

Replace `<PID>` with the process ID of your Claude Code session. Find it by running `echo $$` in the same terminal before starting.

The most useful flags:

- `-i` — Prevents idle sleep (system stays awake, display can still dim)
- `-s` — Prevents sleep even on battery power
- `-d` — Keeps the display from sleeping
- `-t <seconds>` — Automatically stops after a set duration

For most cases, run this before starting Claude Code:
```
caffeinate -i &
```

The `&` sends it to the background so you can keep using the same terminal window.