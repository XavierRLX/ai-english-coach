# Agent Behavior

`AGENTS.md` is the behavioral contract for the AI English Coach.

This document explains the model for developers. It does not replace the contract.

## Runtime model

The project separates three concerns:

```text
ChatGPT conversation = current lesson runtime
AGENTS.md            = behavior contract
learner repository   = persistent memory
```

A new chat should not be treated as a blank learner when the private repository already contains prior state.

The coach should load the repository on `/start` and use that state to shape the next lesson.

## Learning Mode vs Development Mode

The same repository can be discussed for two different reasons.

### Learning Mode

The user is studying English.

The coach should:

- prioritize learner production;
- use conversation-first teaching;
- apply the correction policy;
- track evidence;
- persist only after a valid `/finish`.

### Development Mode

The user is changing the project itself.

The agent should:

- discuss architecture, documentation, schemas, or repository design normally;
- not interpret technical project discussion as learner evidence;
- not update learning progress from development conversation.

This distinction prevents project maintenance from contaminating learner history.

## Evidence policy

Persistent state must be auditable.

The coach must not invent:

- duration;
- learner words;
- mistakes;
- occurrence counts;
- pronunciation quality;
- listening comprehension;
- skill improvement;
- CEFR changes.

Unknown information should remain unknown.

Examples:

```text
cefr: null
pronunciation: null
durationMinutes: null
```

These are valid states, not failures.

## Conservative assessment

The system intentionally avoids forced early scoring.

A first session may provide enough information to identify useful focus areas without providing enough evidence for a CEFR level.

For example:

```text
sessionsCompleted: 1
cefr: null
speaking: null
grammar: null
```

can be correct.

CEFR should reflect consistent evidence across multiple sessions, not a single short interaction.

Internal skill metrics are separate from CEFR and should not be treated as percentages of fluency.

## Correction policy

The coach uses three conceptual levels.

### Critical

Correct immediately when the error blocks understanding or directly affects the current lesson objective.

### Important

Correct at a suitable moment without destroying conversation flow.

### Minor

Usually save for later feedback.

The goal is useful correction, not maximum correction volume.

## Mistake tracking

Recurring issues should be consolidated.

Instead of creating a new record for every equivalent error, update a stable conceptual mistake when the evidence supports it.

The stored learner error must come from observed evidence. A pedagogical example invented by the coach is not learner-history evidence.

## Vocabulary tracking

Do not persist every word used in a lesson.

Track vocabulary only when it has pedagogical value for this learner.

The vocabulary state exists to support future use and review, not to become a transcript-derived word list.

## Voice behavior

Voice is a lesson interface, not a guaranteed repository-access environment.

Recommended sequence:

```text
TEXT
/start
/lesson
  ↓
VOICE
lesson conversation
  ↓
TEXT
/finish
```

The coach should enter Voice with relevant learner context already loaded into the current conversation.

While Voice is active, the coach must not assume GitHub or other connected repository tools are available.

After returning to text, `/finish` can persist the lesson when repository write access exists.

### Transcript limitations

Voice transcripts may be incomplete or normalized.

Therefore:

- do not reconstruct missing learner speech;
- do not store a guessed exact error;
- do not infer occurrence counts from missing turns;
- do not treat transcript spelling as pronunciation evidence.

Pronunciation requires adequate audio evidence available to the evaluating model. Text alone is insufficient.

## Persistence behavior

`/start` reads.

`/finish` writes.

A normal finish conceptually creates or updates:

```text
sessions/<sessionId>.md
mistakes/mistakes.json
vocabulary/vocabulary.json
review/queue.json
progress/current.json
progress/history.json
```

Not every file needs to change after every lesson.

Historical session records are append-only in normal lesson flow.

## Repository tool access

The learner repository must be explicitly identified to ChatGPT. Connecting GitHub does not make the project automatically discover the correct learner repository.

Recommended first-chat instruction:

```text
Use YOUR_USER/english-progress as my AI English Coach learner repository.

Read AGENTS.md and use this repository as my persistent English-learning memory.

Then execute /start.
```

Repository read/write capabilities depend on the ChatGPT environment and connected GitHub integration.

The project should not depend on direct write access.

If writing is unavailable, the coach should prepare clear manual file changes for the learner to apply, commit, and push.

## Template safety

Files matching:

```text
*.example.*
```

are templates.

They are not learner evidence.

This includes:

```text
sessions/session.example.md
```

A template file must not be counted as a completed lesson or used to infer learner ability.

## Core design principle

The project prefers missing data over fabricated data.

A smaller but trustworthy learner memory is more useful than a complete-looking state built from guesses.
