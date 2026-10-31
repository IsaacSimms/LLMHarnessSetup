---
name: grill-me
description: >
  Interview the user relentlessly about a plan or design until reaching shared understanding,
  resolving each branch of the decision tree. Use when the user wants to stress-test a plan,
  get grilled on their design, or mentions "grill me" or "/grill-me". Do not use in normal
  circumstances when the user has not made it clear they want to stress-test a plan.
metadata:
  short-description: "Stress-test a plan via structured Q&A"
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions. For each question, provide your recommended answer.

Split the questions into batches, each batch containing two or three questions. Ask one batch at a time. If the answer to one question is a dependency which affects how you ask a different question, do not put them in the same batch. Ask all questions that act as dependencies first. After each batch is answered, review the question and answer set to determine if any question needs to be discussed again. Do not hesitate to bring up an already answered question again as we work through the batches.

If a question can be answered by exploring the codebase, explore the codebase instead.

## How to ask

Every question is asked by calling `AskQuestion` in that same turn. That call is the multiple-choice picker: the user moves between answers, picks one, and can type their own answer in the box at the bottom. A turn that only prints the questions in the terminal is incomplete. Do not list options as letters, numbers, or bullets in the message. Printing the menu is not a fallback when `AskQuestion` is available. If the host has no `AskQuestion` tool but has another structured multiple-choice question tool that includes a free-text box, call that tool the same way. Prose menus are only for a host that has neither.

On a question turn the message is only the **Settled so far** list, plus the conflict sentence when pushing back on an answer. When nothing is settled yet, the message is one line naming the decision under discussion, with no options. The questions themselves live only in the tool call.

One batch is one tool call. Put two or three questions in that call, or one when a dependency forbids combining them. Keep the grill fast: concrete options, one recommendation, no second copy of the menu in chat.

For each question:

- Give three to five concrete options when the branch has them, and never fewer than two.
- Put the recommended option first. End that label with ` (Recommended)`.
- Make the major discrepancy visible in the picker. Put relative effort, fit to the problem, and what picking it commits to in the option label, or in the question prompt when it does not fit the label. One option being a much larger lift, or a much better fit, must be obvious before they pick.
- Do not add an "Other" or "I'll type it" option. The picker already has the free-text box. A handwritten answer there is a full answer.
- The question prompt is the question plus any discrepancy that did not fit a label. Do not repeat the option list inside the prompt.
- Set `allow_multiple` only when more than one option can be chosen together. Otherwise leave it unset so they pick exactly one.

---

## Session hygiene

These rules do not replace the interview style above. They keep long grills coherent and closable.

### Settled so far

After each resolved answer, carry a short **Settled so far** list into the next turn (one line per closed branch). That list is the message above the next `AskQuestion` call. If the list grows long (roughly more than 8 items), keep the recent items and summarize earlier ones as "+ N earlier" rather than dropping them silently.

### Short accepts

The tool result is the answer: the option they picked, or the text they typed in the free-text box. Do not ask them to reply with letters.

If they type in the composer anyway, treat "yes", "the recommended one", and similar as a full accept of the recommended option when one was marked, or of the option they named. Note what settled in one line; do not re-dump the options. If several questions were open and they answer in order, map each answer to the matching question. A full sentence on one question and a short accept on the next is fine. If it is unclear which question an answer belongs to, ask again with `AskQuestion`.

### Freeform answers

If the user answers with a multi-part paragraph instead of picking an option (including in the free-text box): restate the decision in one tight paragraph, confirm only if something is ambiguous, then ask the **single** highest-dependency residual question with `AskQuestion`. Do not fire multiple new questions in one turn.

### Pushback

If an answer contradicts a settled decision, a stated constraint, or a codebase fact, flag the conflict **before** accepting it. Restate the conflict in the message, then ask again with `AskQuestion` (keep the previous options, and include a conscious-override option when override is legitimate). If they override, settle the new choice and move on.

### No implement during grill

Grill is decision work. Do not start implementing, refactoring, or writing plan/recap/handoff files mid-grill unless the user explicitly asks for that artifact.

---

## Closing the grill

Stop when load-bearing branches are resolved **or** the user says the grill is done.

Then:

1. Emit a numbered **Settled decisions** log in chat (Decision / Choice / one-line rationale). Keep that log in the chat.
2. Offer **once** a next step as one `AskQuestion` question. Do not print this menu in the message. List these six options in this order. The three-to-five cap, the "recommended option first" rule, and the ban on an `other` option do not apply to this question. Mark the fitting option `(Recommended)` on its label. Do not move it.
   - `implement directly` — this thread implements from the settled log
   - `plan as artifact` — write the execution contract (`plan-as-artifact`)
   - `recap thread` — save the backward-looking record (`recap-thread`)
   - `thread handoff` — continue the work in a fresh session (`thread-handoff`)
   - `wayfinder` — multi-session fog / many open decisions remain
   - `other` — a next step that is none of the above
3. The selection is the request. Run it on the turn they answer. Do not wait for a second ask.
   - `implement directly` — implement in this thread. The settled log is the spec. That pick is the implement instruction. Do not write a plan, recap, or handoff first. Non-trivial work still uses the `tdd` skill.
   - `plan as artifact` — run `plan-as-artifact` in that turn. Its Key Decisions are this settled log, every branch. Writing the plan is not permission to implement.
   - `recap thread` — run `recap-thread` in that turn. The settled log is what was decided. Do not implement.
   - `thread handoff` — run `thread-handoff` in that turn. Carry this settled log in. Do not implement in this thread.
   - `wayfinder` — run `wayfinder` in that turn. Carry this settled log in so those branches stay closed. Do not implement.
   - `other` — text they typed is the request; run that. If they picked `other` and typed nothing, ask once what they want. Do not invent a step. Do not implement unless that request says to.
4. `implement directly` is the only closing pick that starts implementation. The others leave the code alone unless the request they typed says to implement.

### Routing (sibling skills)

| Situation | Skill |
|-----------|--------|
| Stress-test decisions (this skill) | `grill-me` |
| Settled; build it in this thread | `implement directly` (no plan file first) |
| Settled → durable execution plan | `plan-as-artifact` → `docs/artifacts/` |
| Multi-session decision map | `wayfinder` → `docs/plans/` |
| Continue work in a new session | `thread-handoff` → `docs/handoffs/` |
| Work finished; durable record | `recap-thread` → `docs/recaps/` |
| Verify implement slice | `check-work` → chat; optional `docs/checks/` |
| Stand up the project docs tree | `create-docs-folder` → `docs/` (`artifacts`, `recaps`, `handoffs`, `checks`, `plans`, `research`, `misc`) |

Standing up the whole tree is `create-docs-folder`. Docs-writing skills still resolve their own output directory and create only their own leaf when they write.
