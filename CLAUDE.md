# World Tree

## Language rule

Every artifact in this project is written in English: specs, plans, tasks, checklists,
constitution, CLAUDE.md, code, comments, identifiers, tests, commit messages, branch names,
PR/issue text and docs. The owner may write in Spanish; chat replies may be in Spanish,
but artifacts stay in English.

## Source of truth

`docs/product-spec.md` is the product source of truth. Do not paste it whole into Spec Kit
commands; feed each step only the sections §19 assigns to it.

## Workflow

Development follows GitHub Spec Kit as defined in §19 "Spec Kit handoff" of the product spec:
constitution → specify → clarify → plan → checklist / tasks / analyze → execute → converge,
one feature at a time in the §19 feature breakdown order.

- `/speckit-implement` is NOT used. Execution is done with superpowers
  `subagent-driven-development`, using `plan.md` + `tasks.md` as the plan, with review
  checkpoints per phase (Setup, Foundational, one per user story, Polish).
- Skip superpowers brainstorming and plan-writing for Spec Kit features; Spec Kit already
  produces the spec, plan and tasks.
- After execution, run `/speckit-converge`; appended tasks go back through subagent-driven
  execution until it reports converged.
- The project constitution lives in `.specify/memory/constitution.md`.
