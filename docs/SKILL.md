# SKILL.md — Antigravity Installed Skill Discovery & Application Policy

## Purpose

The Antigravity agent must use relevant skills already installed in the user's environment. Merely naming a skill, claiming familiarity, or listing a skill without reading and applying its instructions does not count as skill usage.

## Mandatory discovery procedure

1. Inspect Antigravity's installed skills list and the configured skills directories using the supported Antigravity workflow/tools.
2. Identify skills relevant to the current task. Examples of relevant domains may include UI/UX design, accessibility, React, TypeScript, Rust, Tauri, Windows development, performance, testing, security, documentation, 3D modeling, Blender automation, rigging, and animation. These are examples only; do not claim they are installed unless confirmed.
3. Open and read the complete `SKILL.md` for each selected skill. Follow linked reference files, templates, scripts, and constraints when relevant.
4. Read any supporting files the skill explicitly requires before using it.
5. Apply the skill's actual methodology to the task. Do not simply mention the skill in a plan.
6. If skills overlap or conflict, prefer the most task-specific and current instruction, then document the resolution.
7. If a relevant skill is missing, unreadable, or incompatible, state that honestly and use a documented alternative. Never invent installed skills or claim a skill was used when it was not.

## Required skill-use log

Maintain `docs/SKILL_UTILIZATION.md` (create it if absent) with:

| Field | Required content |
|---|---|
| Task | What work was performed |
| Skill name | Exact installed skill name |
| Skill location | Actual path/identifier |
| Read status | Full instructions and supporting references read? |
| Applied method | Concrete work influenced by the skill |
| Evidence | Files/components/tests/design decisions affected |
| Limitation | Missing or unusable skill, if applicable |

Do not count a skill as applied until the agent has read it and used its guidance in a deliverable.

## Task-specific expectations

- UI work: inspect installed UI/UX and accessibility skills before layout/design implementation.
- React work: inspect React/component/testing skills before implementing components.
- Rust/Tauri/Windows work: inspect relevant native systems and security skills before implementing tracking.
- Mascot work: inspect image-generation, 3D, Blender, rigging, or animation skills if installed. The user does not operate Blender; automate the pipeline where possible.
- QA/performance: inspect testing, profiling, and performance skills before claiming results.

## Honesty requirement

If Antigravity cannot expose its skill inventory or read skill files in the current session, say so in the task report. Do not pretend that this document itself proves installed skills were discovered or applied.
