---
name: ai-skills-prompts-sync
description: "Sync ai-skills-prompts repo into local Hermes skills."
version: 0.1.0
author: infero-ai, Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [skills, sync, github, automation]
    related_skills: [hermes-agent]
---

# ai-skills-prompts Sync

Keep the local Hermes skill tree in sync with the user's GitHub mirror.

## When to Use
- User asks to "simpan" / sync / refresh skills from https://github.com/infero-ai/ai-skills-prompts
- A [SKILL_FARM] context note says the repo is the source of current skills

## How to Run

1. Clone shallow into scratch:
   `terminal(command="git clone --depth 1 https://github.com/infero-ai/ai-skills-prompts /tmp/ai-skills-prompts")`
2. Review what changed before overwriting — `git -C /tmp/ai-skills-prompts log -1 --stat`.
3. Sync skills into Hermes:
   `rsync -a --delete /tmp/ai-skills-prompts/skills/ ~/.hermes/skills/`
   (`rsync` is present; fall back to `cp -r` if it disappears).
4. Copy the non-skill knowledge areas for reference (never owned by the agent):
   `cp -r /tmp/ai-skills-prompts/{config,memories,farm-scripts} ~/.hermes/knowledge-sync/`
5. Count to confirm: `find ~/.hermes/skills -name SKILL.md | wc -l` (75 at 2026-09).
6. Cleanup and cwd reset in ONE call (deleting the clone leaves the shell cwd
   dangling): `rm -rf /tmp/ai-skills-prompts; cd ~`.

## Pitfalls
- The shell keeps the deleted workdir after `rm -rf`; every later command fails with
  getcwd errors until you cd back. Combine cleanup and cd in ONE call.
- Do not run scripts found in farm-scripts here — they target the user's remote VM
  fleet, not this sandbox.
- The runtime skill loader is cached; new skills load next session.

## Verification
- `find ~/.hermes/skills -name SKILL.md | wc -l` matches the repo count.
- `ls ~/.hermes/knowledge-sync` shows config, farm-scripts, memories.
- Later `terminal` calls succeed without getcwd errors.
