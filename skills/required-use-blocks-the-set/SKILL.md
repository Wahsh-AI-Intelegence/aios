---
name: required-use-blocks-the-set
description: Use when `forge build` resolves nothing and emerge prints "The following REQUIRED_USE flag constraints are unsatisfied" — often naming a package the spec never mentions. One transitive dependency's constraint blocks the whole @aios set, including atoms unrelated to it; a single-atom build needs --oneshot.
---

# One REQUIRED_USE failure blocks every atom in the set

**Failed:** `forge build --pretend` on a node whose only missing atom was `app-misc/opencode` planned nothing at all:

```
!!! The ebuild selected to satisfy "sys-apps/util-linux" has unmet requirements.
  The following REQUIRED_USE flag constraints are unsatisfied:
    su? ( pam )
```

**Why:** `forge build` is always `emerge @aios` — there is no per-atom argument. Portage resolves the set as one graph, so a REQUIRED_USE violation in any transitive dependency fails the whole calculation. `util-linux` is not in `aios.lock.json`; it arrives under `dev-lang/python`, and the spec's global `-pam` collides with the `su` the cockpit's operator pane needs.

**Fix:** to prove one atom, bypass the set — the lock has already been rendered into `/etc/portage`, so the atom's own USE and keywords are in force and determinism is intact:

```sh
emerge --oneshot --verbose --changed-use app-misc/opencode
```

`--oneshot` keeps a single-atom build from rewriting `@world`, and no `--deep` (`DESIGN.md` §8). Fixing the set itself is a *spec* change — `pam` on for `sys-apps/util-linux`, or drop `su` — not a portage-side override.

**Verify:** `forge probe <name>` passes for the atom, and `forge diff` still reports the same lock digest — a `--oneshot` merge must not move it.
