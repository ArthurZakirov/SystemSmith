---
name: agents-freshness
description: On demand only, verify whether this Codex conversation's last recorded global AGENTS.md matches the current global instructions. Return current, outdated, or unverifiable; never guess which session is active.
---

# Verify current Codex session's AGENTS.md

When explicitly invoked as `$agents-freshness`, perform a **read-only**
verification of the *invoking conversation's* last recorded global
`AGENTS.md` snapshot against the current global instructions on disk.

## Check

Run this command with the **shell tool in the invoking Codex session**. That
tool may provide `CODEX_THREAD_ID` in its environment. Do not infer the chat
from the frontmost application, current directory, most recently modified
rollout file, or manually supplied guess.

```sh
python3 - <<'PY'
import json
import os
from pathlib import Path
import subprocess
import sys

session_id = os.environ.get("CODEX_THREAD_ID")
if not session_id:
    print(json.dumps({"status": "unknown", "reason": "CODEX_THREAD_ID unavailable"}))
else:
    codex_home = Path(os.environ.get("CODEX_HOME") or Path.home() / ".codex")
    checker = codex_home / "hooks-json/agents-md-freshness/check.py"
    if not checker.is_file():
        print(json.dumps({"status": "unknown", "reason": "Freshness checker not installed"}))
    else:
        subprocess.run(
            [sys.executable, str(checker), "--check", "--session-id", session_id],
            check=True,
        )
PY
```

If shell execution isn't available or fails, the outcome is **unknown**.
Never infer a green status from a missing notification or unsuccessful check.

## Report the observed JSON

- `current`: **✅ AGENTS.md current** — this conversation's last recorded
  global instructions match the current file. Include the session ID prefix.
- `outdated`: **❌ AGENTS.md outdated** — include session ID prefix and source
  digests (recorded vs. current) when available. Recommend a fresh session if
  reloading the instructions doesn't occur.
- `unknown`: **❔ AGENTS.md not verified** — report the actual reason.
  Never present the state as current without evidence.

Keep the reply concise. The evidence reflects the last persisted
model-visible *global* snapshot, not all project-specific `AGENTS.md` files
or inaccessible internal model context. Do not modify files, restart Codex,
or send unrelated messages.
