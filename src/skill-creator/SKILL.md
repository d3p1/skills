---
name: skill-creator
description: >
  Create new skills, and modify and improve existing skills.
  Use when users want to create a skill from scratch, edit, or 
  optimize an existing skill.
---

# Goal

Help to write new skills and improve existing ones so they trigger reliably
and stay easy to follow.

## Anatomy of a skill

```
skill-name/
├── SKILL.md         (required: YAML frontmatter + instructions)
├── scripts/         (optional: code the model can run)
├── references/      (optional: docs loaded into context only when needed)
└── assets/          (optional: files used in the output, e.g. templates)
```

Start with just `SKILL.md`. Only add a resource folder once the skill
actually needs it — e.g. move a reusable script to `scripts/`, or split
out a long reference doc so `SKILL.md` stays short.

## Writing the description

The `description` is the only part that's always in context — it's what
decides whether the skill gets used at all. Cover two things: what the
skill does, and when to use it. Be specific about triggering contexts,
and lean slightly "pushy" — models tend to under-trigger skills, so spell
out the phrasings and situations that should invoke it, not just the
ideal case.

**Weak:** `Helps format commit messages.`
**Better:** `Format commit messages to match this repo's Angular-style
convention. Use whenever the user is about to commit, asks for a commit
message, or writes one that doesn't follow the convention.`

## Writing the body

- Keep it short — a few hundred lines at most. If a skill is growing past
  that, split detail into `references/` and point to it from `SKILL.md`
  rather than inlining everything.
- Point to reference files conditionally, tied to when they're actually
  needed (e.g. "Read `references/aws.md` only if deploying to AWS"),
  rather than as a plain link. `SKILL.md` is always loaded once the skill
  triggers, but `references/` files are only pulled into context when the
  model follows the pointer — vague or unconditional pointers erase that
  benefit by getting read every time.
- Use imperative instructions ("Do X", not "You should do X").
- Explain *why* a step matters instead of stacking ALWAYS/NEVER rules — a
  clear rationale generalizes better than a rigid instruction.
- Prefer guidance that works across the range of cases the skill will
  see, not just the one example that prompted writing it.

## Template

Use [this template](./references/template.md) as the starting structure
for a new skill.
