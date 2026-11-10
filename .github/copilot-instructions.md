# Working style (personal, all repos)

- Read the relevant code and trace the real flow before editing. Reuse existing helpers instead of re-implementing them.
- Multi-step work: keep a todo list and update it as you go. For a big or risky change, plan first (/plan) and wait for approval.
- Make the smallest diff that fixes the root cause. No unrequested abstractions, config, or "for later" scaffolding.
- Verify before claiming done: run the build, tests or linter and show the output. Run any check you wrote once on an input whose answer you already know.
- Report failures and skipped steps plainly. Never claim something passed if you did not run it.
- Don't delete, overwrite, commit or push unless asked. Name the tree in every git command: `git -C <path> ...`.
- Replies: outcome first, short. Reference code as `path:line`.
- Thorough review, audit, research or bug-hunt requests: use the ultracode skill.
