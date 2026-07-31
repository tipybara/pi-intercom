# pi-intercom (fork)

Local fork of [pi-intercom](https://github.com/nicobailon/pi-intercom) **0.10.1** with broker-enforced **session group isolation**.

Upstream docs (unchanged feature set, install, tools, skill patterns): see [`original_readme.md`](./original_readme.md).

## Customization: `PI_INTERCOM_GROUP`

Sessions only list, see presence/join/leave, and message peers in the **same group**. Cross-group targets behave as if absent (`Session not found`).

| Env | Default | Effect |
| --- | --- | --- |
| `PI_INTERCOM_GROUP` | `default` | Isolation partition for this process. Unset or blank → `default`. |

### Example

```bash
# Team A sessions only see each other
PI_INTERCOM_GROUP=teamA pi

# Team B sessions only see each other
PI_INTERCOM_GROUP=teamB pi

# Unset / blank → default group
pi
```

### Behavior

- Registration carries `SessionInfo.group` (broker defaults missing/blank to `default`).
- `list`, `session_joined`, `session_left`, and `presence_update` stay inside requester group.
- `send` / `ask` / delivery cannot cross groups; other groups’ names and IDs resolve as absent.
- Non-default groups appear in session-list UI and tool rows as `[group:<name>]`.
- Same-group behavior retains upstream 0.10.1 endpoint epochs, durable pending asks, cwd-scoped mailboxes, and tmux roster metadata.

### Notes

- One local broker still serves all groups; isolation is logical, not separate process.
- Pick stable group name per team/project. Typos create empty partitions.
- Default group stays compatible with clients that omit `group`.
