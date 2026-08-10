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

Feel free to force me to write in answers to the questions if needed. But, if asking me a question, then giving me 3, 4, 5 however many options to answer is applicable, do that. I would like to keep the grill-me process fast wherever possible. When giving me options, always be willing to still take a handwritten answer from me.

If a question can be answered by exploring the codebase, explore the codebase instead.

When giving me options on how something should be implemented, describe any major discrepancies between the options. If one option is a much larger lift to implement, a much better option for the nature of the solution being worked on, and so on, make that obvious to me.

---

## Session hygiene

These rules do not replace the interview style above. They keep long grills coherent and closable.

### Settled so far

After each resolved answer, carry a short **Settled so far** list into the next turn (one line per closed branch). Put it above the next question. If the list grows long (roughly more than 8 items), keep the recent items and summarize earlier ones as "+ N earlier" rather than dropping them silently.

### Short accepts

Treat A, yup a, yes, and similar as a full accept of the labeled option — or of the Recommended option if the user only affirms recommendation and one was marked as such. Note what settled in one line; do not re-dump the full options table. Use best judgment when analyzing the user response to make sure you understand the answer to each question. For example, if three options were asked and the response is "A B A", then the answer to question one is A, the answer to question two is B, and the answer to question three is A. The user may answer in full sentences for one question, then respond with a short accept for the next question. If there is not clear understanding of the answer to each question, ask the user for clarification.

### Freeform answers

If the user answers with a multi-part paragraph instead of picking an option: restate the decision in one tight paragraph, confirm only if something is ambiguous, then ask the **single** highest-dependency residual question. Do not fire multiple new questions in one turn.

### Pushback

If an answer contradicts a settled decision, a stated constraint, or a codebase fact, flag the conflict **before** accepting it. Restate the conflict, re-offer options (or ask them to consciously override). If they override, settle the new choice and move on.

### No implement during grill

Grill is decision work. Do not start implementing, refactoring, or writing plan/recap/handoff files mid-grill unless the user explicitly asks for that artifact.

---

## Closing the grill

Stop when load-bearing branches are resolved **or** the user says the grill is done.

Then:

1. Emit a numbered **Settled decisions** log in chat (Decision / Choice / one-line rationale). Chat only — do not write a file unless they ask.
2. Offer **once** a next step:
   - `plan-as-artifact` — if an implementable contract is the natural next step
   - `wayfinder` — if multi-session fog / many open decisions remain
   - stop — if they only needed alignment
3. Do not implement after close unless they explicitly say to.

### Routing (sibling skills)

| Situation | Skill |
|-----------|--------|
| Stress-test decisions (this skill) | `grill-me` |
| Settled → durable execution plan | `plan-as-artifact` → `docs/artifacts/` |
| Multi-session decision map | `wayfinder` → `docs/plans/` |
| Continue work in a new session | `thread-handoff` → `docs/handoffs/` |
| Work finished; durable record | `recap-thread` → `docs/recaps/` |
| Verify implement slice | `check-work` → chat; optional `docs/checks/` |