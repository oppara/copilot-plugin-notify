# copilot-plugin-notify

Copilot CLI hook events are shown as macOS notifications via [`terminal-notifier`](https://github.com/julienXX/terminal-notifier) by default. Set `COPILOT_NOTIFY_FORCE_STDOUT=1` to emit OSC 777 notification escape sequences instead, for listeners such as cmux.

Tested on macOS only.

## Install

On macOS, install [`terminal-notifier`](https://github.com/julienXX/terminal-notifier).

```bash
brew install terminal-notifier

git clone https://github.com/oppara/copilot-plugin-notify.git
cd <parent directory of copilot-plugin-notify>
copilot plugin install ./copilot-plugin-notify
```

## Test

```bash
bash tests/notify_test.sh
```

## Environment variables


| Name                                         | Description                                                                                     | Default                     |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------- |
| `COPILOT_NOTIFY_DEBUG`                       | set to `1` to enable debug logging (input/debug log + extra `agentStop` dump)                 |                             |
| `COPILOT_NOTIFY_DEBUG_PATH`                  | debug log path used when `COPILOT_NOTIFY_DEBUG=1`                                              | `/tmp/copilot-notify.jsonl` |
| `COPILOT_NOTIFY_FORCE_STDOUT`                | set to `1` to emit OSC 777 to stdout instead of using `terminal-notifier`                      | `0`                         |
| `COPILOT_NOTIFY_AGENTSTOP_POLL_ATTEMPTS`     | number of retries when waiting for the latest `assistant.message` after `agentStop`            | `10`                        |
| `COPILOT_NOTIFY_AGENTSTOP_POLL_INTERVAL_SEC` | interval seconds between retries                                                                | `0.05`                      |
| `COPILOT_NOTIFY_AGENTSTOP_ACCEPTABLE_AGE_MS` | maximum age from hook input timestamp for candidate messages to avoid stale notifications       | `3000`                      |

## Developer notes

### Files

- `plugin.json`: Plugin metadata
- `hooks.json`: Registers `notification`, `preToolUse` and `agentStop` hooks
- `scripts/notify.sh`: Reads hook payload from stdin and shows macOS notifications via `terminal-notifier` by default

### Notes

- Hook payload is read from stdin (JSON).
- Set `COPILOT_NOTIFY_FORCE_STDOUT=1` to emit OSC 777 notification escape sequences to stdout instead of using `terminal-notifier`.
- `notification` hook:
  - fires when Copilot CLI pauses for human confirmation (e.g. tool execution permission).
  - sends the `message` field from the payload as a notification.
- `ask_user` / `exit_plan_mode` / `task_complete` (via `preToolUse`):
  - notify with normalized `question` / `summary`.
- `agentStop`: notify when Copilot finishes an agent turn and waits for user input.
