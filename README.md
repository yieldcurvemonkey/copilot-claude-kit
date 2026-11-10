# Claude Code → VS Code Copilot Chat kit

An ultracode-style agent for VS Code Copilot Chat. All the files live in `.github/`, so VS Code discovers them per repo.

Checked against the VS Code 1.145 docs (2026-11-04). Choosing a Claude model does not change the harness.

```
.github/
  agents/ultracode-mode.agent.md   pick it in the agent picker: every substantive task fans out
  agents/scout.agent.md            read-only investigator subagent (hidden from the picker)
  agents/verifier.agent.md         adversarial refuter subagent (hidden from the picker)
  skills/ultracode/SKILL.md        the protocol, also usable as /ultracode <task> from any agent
  copilot-instructions.md          Claude Code-style working habits (always on)
```

## 0. Check first (2 min)

1. **Session Target picker** in the Chat view. If you see **Claude**, consider using it. It runs the real Claude Agent SDK, billed through Copilot, and your home CLAUDE.md, `.claude/skills`, `.claude/agents` and hooks work as-is. ANY Copilot enterprise-managed setting disables it.
2. **Command Palette → Developer: Policy Diagnostics.** Look for:
   - `ChatAgentMode`
   - `ChatStrictPluginOnlyCustomization` (blocks this whole kit)
   - `Claude3PIntegration`
   - `ChatAllowManagedHooksOnly`
   - `CopilotCliCommand`

## 1. Install into a repo

Run in PowerShell. The repo is private, so cloning it at work needs your GitHub login.

```powershell
git clone https://github.com/yieldcurvemonkey/copilot-claude-kit "$HOME\copilot-claude-kit"
$k = "$HOME\copilot-claude-kit\.github"
$r = "C:\path\to\your\repo\.github"          # <- edit this
New-Item -ItemType Directory -Force "$r\agents", "$r\skills" | Out-Null
Copy-Item "$k\agents\*.agent.md" "$r\agents\"
Copy-Item -Recurse -Force "$k\skills\*" "$r\skills\"
Get-Content "$k\copilot-instructions.md" | Add-Content "$r\copilot-instructions.md"   # appends; never overwrites
```

Or open this repo itself in VS Code to try it out.

Then reload the window. Run **Chat: Open Customizations** and check that the Agents and Skills tabs list the new files.

## 2. Harness

The files use only fields that both VS Code harnesses accept. That is why they have no `skills:`, `agents:`, `handoffs:` or `argument-hint` on the agents, and no tool names specific to one harness.

- **Copilot harness.** The default since 1.143. Works as-is. Delegation uses its native `task` tool.
- **Local harness.** Legacy; this is what `target: vscode` refers to. Delegation uses `runSubagent`. To use it, add these user settings, then pick **Local** in the Session Target picker:

  ```json
  "chat.editor.localAgent.enabled": true,
  "chat.defaultToCopilotHarness": false,
  "chat.editor.preferCopilotHarness": false
  ```

  These settings are experimental, and an admin can forbid Local with `vscode.temporaryAllowLocalHarness: false`.

## 3. Use

- **Standing mode:** pick **ultracode-mode** in the agent picker.
- **One-shot from any agent:** `/ultracode review my uncommitted changes`.
- **Model:** Claude Opus 5.5 or Sonnet 5.5. Click `>` next to the model to raise reasoning effort. Avoid Auto, because subagents inherit whatever Auto picks.

## 4. Smoke tests (things the docs don't promise)

- **Parallelism.** In the Local harness, parallel subagents are announced in the 1.109 release notes. In the Copilot harness, the docs only say "when the harness supports" it. Watch the scout timers in the Agents window: overlapping timers mean parallel. If they run one after another, the kit still works, just slower.
- **Instructions reach subagents?** GitHub's docs say Copilot-harness custom subagents don't get repo instruction files. That is why the skill packs the needed context into every brief.

## 5. Cost

- Billing is per token in AI credits. 1 credit = $0.01.
- Monthly allowance per user, pooled across the org: Business 1,900 credits, Enterprise 3,900.
- Fan-out multiplies tokens, so one big run can eat a large share of the month.
- Cheaper workers: give `scout.agent.md` a `model:` line.
  - Copilot harness: use runtime IDs, e.g. `model: ['claude-sonnet-5.5', 'claude-haiku-5.5']`.
  - Local harness: use display names, e.g. `Claude Sonnet 5.5 (copilot)`.

## Claude Code → Copilot harness map

Gate key: **OOB** = works out of the box. **Admin** = an org policy can block it.

| Claude Code | Copilot equivalent | Gate |
|---|---|---|
| CLAUDE.md | `AGENTS.md` or `.github/copilot-instructions.md`; personal: `~/.copilot/copilot-instructions.md` | OOB |
| `.claude/rules` | `.github/instructions/*.instructions.md` with an `applyTo:` glob | OOB |
| `.claude/agents` | `.github/agents/*.agent.md` or `~/.copilot/agents` | OOB |
| Task tool / subagents | native `task` tool (the `agent` alias in `tools:`); ask "delegate to the X agent" | OOB; parallelism undocumented |
| Skills | `.github/skills/<name>/SKILL.md` or `~/.copilot/skills` | OOB |
| Slash commands | skills (add `disable-model-invocation: true` for manual-only). `.prompt.md` files are NOT loaded by the Copilot harness (Local only) | OOB |
| Plan mode / TodoWrite / `/compact` / rewind | `/plan`, `todo` tool, `/compact`, checkpoints and `/fork` | OOB |
| MCP | `.mcp.json` (`mcpServers` key) | Admin |
| Hooks | `.github/hooks/*.json`; also reads the repo's `.claude/settings.json` hooks | Preview + Admin |
| Plugins (superpowers etc.) | setting `chat.plugins.enabled: true`, then `/plugin marketplace add <owner/repo>`. The `.claude-plugin` format is documented. Superpowers inside VS Code is reported to work but isn't in the docs, so test it (its CLI install is documented) | off by default + Admin |
| ultracode / workflows | this kit; Agents window parallel sessions with New Worktree; Compare Models (judge + synthesizer, Preview); Copilot Fusion model (draft → critique → revise); Copilot CLI `/fleet` in the terminal | mixed |
| `--dangerously-skip-permissions` | Autopilot mode or the "Allow all" approval level | Admin |

Main sources:

- https://code.visualstudio.com/docs/agents/run/agent-harnesses
- https://code.visualstudio.com/docs/agents/run/subagents
- https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference
- https://code.visualstudio.com/docs/agent-customization/agent-skills
- https://code.visualstudio.com/docs/enterprise/manage-ai-settings
