# Long-running processes

Anything that binds a port — a dev server, a preview server, a watcher — outlives the session that started it unless the harness owns it. A leftover listener makes the next run fail to bind, and a process outside the sandbox cannot be signalled afterwards.

## Ownership

- **A server you did not start is not yours.** The owner's, another agent session's, another checkout's — never stop, restart or reconfigure one, even when the task is blocked on its port.
- **Start your own** when you need one, as a harness service with a name and a readiness check (a port, or a log line). The harness then owns the process and its kill releases the port.
- **Stop only what this session started**, then confirm the port is free. A listener still up after your stop is a leak; a listener you did not start is reported, never cleared.
- **Before finishing**, stop every service you started and confirm each port is free.

## The auto-background trap

Some CLIs detect an agentic environment and detach themselves: the command exits 0 and a child keeps the port. A task runner returning is not evidence that anything owns the server.

Find the switch that keeps it in the foreground — an environment variable, or a flag such as `--ignore-lock` — and pass it. A detached child is invisible to the tool's own stop and status commands.

| What | Value in this repo |
| --- | --- |
| Start command | `<...>` |
| Foreground switch | `<env var or flag>` |
| Granted port range | `<...>` |
| Lock or pid file | `<...>` |
| Readiness check | `<port \| log line>` |

## When the port is already taken

Take another port inside the granted range and continue. Do not clear the listener and do not delete the lock file: deleting it only blinds the owner's own stop command while the process keeps the port.

## Bounded runs

A command that never returns is a stall, not a server. When a one-shot needs the app booted — a migration, a seed, a smoke request — bound it: a timeout, a free port, and a check afterwards that nothing remains (`ss -ltn`, `pgrep`).

Prefer the harness service for anything you will query more than once; a bounded run is for a single observation.
