# AIos

A self-building Gentoo: you state what you use, it compiles only that. The product is the lowering
pass (intent → build config) and the minimization loop — not a new distro. Loop and standing rules:
`~/.Codex/AGENTS.md` and `~/.Codex/rules/`.

## Two rules everything else follows from

1. **The AI is never in the build path.** `forge lower` writes a canonical, sha256-digested
   `aios.lock.json`; every downstream stage reads only that. Never add a code path where a model
   decides something at build or boot time, and never let `forge lower` output reach portage without
   going through the lockfile.
2. **Every USE flag carries a `why`** naming the intent that justified it. That provenance is what
   makes the system auditable.

Without (1) it becomes an unreproducible pile that can never be debugged; without (2) the audit
trail is gone.

## Constraints that shape the code

- **aarch64 / musl first.** Native VM speed on Apple Silicon beats emulated x86_64.
- **`forge` is stdlib-only Python** — there is no pip on a bare musl target. Do not add
  dependencies; the provider layer is pluggable instead.
- **The target self-modifies** in place via A/B roots. This is a requirement, not a stretch goal.
- **The in-box agent escalates**: haiku-4-5 plans and dispatches, spawning sonnet-5/opus-5 only when
  a task needs it, and may not declare success until `forge probe` has actually passed.

Phase status and open questions live in `DESIGN.md`; the running cluster and image are described in
the ConductorAI `aios` repo focus — `recall` it before touching live state.

## Commands

```bash
python3 -m unittest discover -s tests -t .   # the test suite
forge diff                                   # is the lock stale against the spec?
forge lower                                  # the one step that calls a model; --dry-run writes nothing
forge render --root ./out                    # inspect before rendering to /
forge build --execute                        # emerge --verbose --deep --newuse --changed-use @aios
forge probe tmux                             # after a build: "tmux: ok 5/5", then one line per check
python3 -m aios.build status|tail|list|stop  # long-running builds, detached
```

`forge probe` passing is the only evidence a build worked. Nothing is done because `emerge` exited 0.

## Skills are the compounding surface

`skills/` is a **failure→fix log**, not general advice: one skill per mistake that cost real time,
written the moment it was fixed. 45 of them, reachable as `.agents/skills` (symlink) so they load
automatically. Read the matching one before repeating a mistake — it already covers the portage
keyword trap, the subprocess-stdin hang, colima/kind restart side effects, and the zsh
word-splitting trap.

When something fails and then gets fixed here, add a skill in the same house style: verbatim error →
one sentence of cause → the fix → one verify command. Keep `skills/README.md`'s table in sync.

## Care

- The local `aios` kind cluster is shared infrastructure — `claim` it in ConductorAI before a
  recreate, and remember that starting colima resurrects every kind cluster and steals their host
  ports.
- Pushes are Evgeny's to run (`~/.Codex/rules/never-push.md`), including to `evgeny-pai/aios`.
