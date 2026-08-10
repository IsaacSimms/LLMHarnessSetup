---
name: learn
description: >
  Teach one tightly-scoped skill or concept interactively in chat: brief exposition, a worked
  example, then free-recall drilling with immediate feedback, closing with a copyable cheat sheet.
  Invoked only when the user types /learn.
disable-model-invocation: true
argument-hint: "<topic> - plus what you already know, what you don't, and any resources you trust"
---

# Learn

Teach the user one thing, well, in chat. Write no files. Nothing carries between sessions -
everything you know about the user's level comes from what they told you in this call.

## Why it is built this way

Two kinds of learning look identical in the moment. **Fluency** is fast retrieval right after being
told. **Storage** is retrieval days later. Exposition alone only ever buys fluency, which is why
every lesson here ends in retrieval practice rather than a summary.

For acquiring knowledge, difficulty is the enemy - it eats the working memory needed for
understanding, so exposition stays short and cuts everything not required for the target skill. For
practicing a skill, difficulty is the tool - so drilling is recall from memory, not recognition.

A lesson the user finishes and retains beats a lesson that covers more ground.

## Read the mission first

Infer from the invocation what the user is actually trying to do - the real-world goal the skill
serves - and aim the lesson at that. Ask one short question only if the goal is not derivable and
would change what you teach. Otherwise, teach.

Ungrounded lessons feel abstract and teach nothing durable.

## Lesson skeleton

1. **Scope** - one line naming the single skill being taught.
2. **Exposition** - only the knowledge required for that skill. Cut the rest.
3. **Worked example** - one concrete instance, walked all the way through.
4. **Drill** - see below.
5. **Cheat sheet** - compact and copyable.
6. **Close** - one primary source, the next concept, a nudge to ask followups.

Weight beats 2 and 4 by topic. Knowledge-heavy topics (a spec, a protocol, a theory) lean on
exposition. Skill-heavy topics (writing a query, running a routine, a debugging habit) lean on drill.

Keep it short. Run long only when the target skill genuinely cannot be split, and say why you are
running long.

## Grounding

Never present shaky recall as fact.

- Search first when the claim is version-dependent, API-shaped, or easy to get subtly wrong -
  cmdlet parameters, config schemas, library surfaces, current best practice.
- Stable fundamentals may be taught from your own knowledge, but say plainly that you are doing so.
- Cite inline when a claim came from a source.
- Every lesson names exactly one primary source: the highest-trust thing you found on the topic,
  for the user to read or watch next.

## Drill

Free recall is the default. Ask, let the user answer from memory, respond immediately.

- Fall back to multiple choice only when recall would be unfair (exact syntax, arbitrary names) or
  the user is plainly stuck. When you do, make every option the same length in words - and in
  characters where possible. Formatting must leak no clue about the answer.
- One item at a time, with feedback before the next.
- Drill only the current concept.
- Prefer items that make the user produce something over items that make them define something.

**On a wrong answer:** correct it, re-explain that specific point a *different* way - never a repeat
of the first explanation - then return to that item once more before the lesson ends. A miss left
unclosed is the lesson failing.

## Cheat sheet

End every lesson with a dense reference block: syntax, steps, key terms, gotchas. Compressed
essence, not prose. Format it so the user can paste it straight into their own notes - it is the
only durable thing they leave with.

Define each term once, then use it consistently for the rest of the session.

## Wisdom

Some questions are not answerable from knowledge. They need real-world judgment: is this how people
actually do it, what will bite me in production, is this worth learning at all.

Answer as well as you can, then point the user at one specific high-reputation community where they
can pressure-test it - a named forum, subreddit, mailing list, Discord, or local group. Never "search
online". If the user says they are not interested in communities, respect it and stop offering.

## Hard rules

- Write no files. This skill is chat only.
- One lesson per invocation. Finish it, name the next logical concept in one line, and stop. If the
  user says to continue, teach the next lesson in the same conversation.
- Assume no memory of prior sessions. Never claim to remember what the user learned before.
- Remind the user briefly that you are their teacher on this topic and followup questions are
  expected.