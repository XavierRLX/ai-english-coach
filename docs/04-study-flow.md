# Study Flow

The core loop is intentionally small:

```text
/start
  ↓
plan
  ↓
/lesson
  ↓
text or voice
  ↓
/finish
  ↓
persist
  ↓
next /start
```

The repository stores long-term state. The current ChatGPT conversation is the active lesson runtime.

## Phase 1 — Read

Run `/start` in text mode.

When real learner files exist, the coach loads context in this order:

```text
AGENTS.md
config/student.json
progress/current.json
review/queue.json
mistakes/mistakes.json
vocabulary/vocabulary.json
up to 3 recent real session files
```

Files matching `*.example.*` are templates and must be ignored as learner evidence.

The read phase should answer:

- What is the learner trying to achieve?
- Is CEFR known or still unassessed?
- What is the current focus?
- Which mistakes are recent or recurring?
- Which review items are due?
- What happened in the most recent sessions?

`/start` does not change learner state.

## Phase 2 — Plan

The coach prepares a short lesson plan using:

```text
student goal
+
current level or unknown assessment state
+
current focus
+
recent mistakes
+
due reviews
+
recent sessions
+
student interests
```

If CEFR is `null`, the coach should adapt through natural interaction instead of assigning a beginner level without evidence.

## Phase 3 — Lesson

Run:

```text
/lesson
```

The lesson may stay in text or continue in Voice.

### Text path

```text
/start
  ↓
/lesson
  ↓
text conversation
  ↓
/finish
```

### Voice path

```text
TEXT: /start
  ↓
TEXT: /lesson
  ↓
VOICE: conversation
  ↓
TEXT: /finish
```

Voice uses context already loaded into the current conversation.

The workflow must not assume repository or connected-app tools are available while Voice is active.

If a Voice transcript is incomplete, missing speech is not evidence and must not be reconstructed.

Transcript text alone is not reliable pronunciation evidence.

## Phase 4 — Finish and write

Run `/finish` in text mode when repository persistence is required.

The coach analyzes only available lesson evidence.

Conceptual persistence order:

```text
1. analyze the lesson
2. prepare changes
3. create the session record
4. update mistakes
5. update vocabulary
6. update review queue
7. update current progress
8. append progress snapshot
9. verify consistency
```

Potentially updated files:

```text
sessions/<sessionId>.md
mistakes/mistakes.json
vocabulary/vocabulary.json
review/queue.json
progress/current.json
progress/history.json
```

`config/student.json` normally changes only when the learner configuration changes.

## What may stay unchanged

A valid `/finish` does not need to change every file.

For a short lesson, the coach may correctly leave:

- CEFR as `null`;
- skill scores as `null`;
- vocabulary unchanged;
- review queue unchanged;
- duration unknown.

Evidence quality is more important than filling every field.

## Phase 5 — Next session

The next `/start` reads the updated learner repository.

Example:

```text
Session 1
/start
/lesson
/finish
   ↓
GitHub learner state updated
   ↓
Session 2
/start
   ↓
coach sees prior focus, mistakes, strengths and session history
```

This is the primary persistence behavior the MVP is designed to prove.

## Read vs write summary

| Phase | Reads repository | Writes repository |
| --- | --- | --- |
| `/start` | Yes | No |
| Lesson in text | Uses loaded context | No normal persistence |
| Lesson in Voice | Uses conversation context | Must not assume repository access |
| `/finish` | May reread current state | Yes, when write access exists |
| `/status` | Yes | No |
| `/review` | Yes | Only through a later valid `/finish` |

## Write-access fallback

If the ChatGPT environment cannot write to the learner repository:

1. the coach still analyzes the lesson;
2. it prepares the exact affected file changes;
3. the learner applies them locally;
4. the learner commits and pushes;
5. the next `/start` reads the new state.

The persistence model therefore does not require direct GitHub write access to remain usable.
