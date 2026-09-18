# Data Model v1

This document defines the conceptual and structural data contract for AI English Coach v1. It describes the persistent files used by one learner instance before JSON Schemas are introduced.

This is documentation only. It does not define a backend, database, API, schema files, or migration tooling.

## 1. Data Model Principles

AI English Coach uses three complementary persistence forms:

```text
JSON = structured current state
Markdown = narrative session history
Git = versioned persistence
```

Structured JSON files represent machine-readable state that agents can load, update, and validate. Markdown session files represent human-readable learning history. Git provides the versioned persistence layer for both.

All structured JSON files MUST include a top-level `version` field. The initial data model version is `1`.

Dates and timestamps MUST use ISO 8601:

- timestamps with a time component MUST include timezone information;
- dates without a time component MUST use `YYYY-MM-DD`;
- agents MUST NOT invent timestamps or durations when evidence is missing.

IDs MUST be stable. They should remain valid across edits to descriptive text and MUST NOT depend on array position.

Agents MUST NOT invent unknown fields, fictional values, artificial evidence, or placeholder learning data. Absence of evidence MUST remain explicit as `null`, an omitted optional field, or another contract-defined unknown state.

## 2. Sources of Truth

A real learner instance uses these files as sources of truth:

```text
config/student.json
progress/current.json
progress/history.json
mistakes/mistakes.json
vocabulary/vocabulary.json
review/queue.json
sessions/*.md
```

Files matching:

```text
*.example.*
```

exist only in the public template. They are examples, not real learner state. This includes `sessions/session.example.md`. Agents MUST NOT read example files as evidence about a learner and MUST NOT update them as if they were active data.

## 3. `config/student.json`

`config/student.json` stores learner configuration and preferences.

Example v1 structure:

```json
{
  "version": 1,
  "name": "Example",
  "nativeLanguage": "pt-BR",
  "targetLanguage": "en-US",
  "goal": {
    "primary": "conversation",
    "description": "Speak English comfortably in daily situations"
  },
  "preferences": {
    "method": "conversation-first",
    "correctionStyle": "balanced",
    "explanationLanguage": "pt-BR"
  },
  "interests": [
    "technology",
    "work"
  ]
}
```

Field semantics:

| Field | Meaning |
| --- | --- |
| `version` | Format version for this file. Initial value is `1`. |
| `name` | Friendly learner identifier. It is a string and does not need to be a legal name. |
| `nativeLanguage` | Learner's native language as a BCP-47 language tag. |
| `targetLanguage` | Language being learned as a BCP-47 language tag. |
| `goal.primary` | Main learning objective. Initial concepts include `conversation`, `professional`, `travel`, `academic`, and `general`, but this documentation does not require a rigid enum yet. |
| `goal.description` | Human-readable description of the learner's goal. |
| `preferences.method` | Teaching method preference. v1 initially uses `conversation-first`. |
| `preferences.correctionStyle` | Correction intensity preference. Initial values are `gentle`, `balanced`, and `intensive`. |
| `preferences.explanationLanguage` | Preferred explanation language as a BCP-47 language tag. |
| `interests` | Simple list of interests used to contextualize lessons. |

## 4. `progress/current.json`

`progress/current.json` stores the consolidated current learning state. It is mutable and should represent the latest known view of the learner.

Example v1 structure:

```json
{
  "version": 1,
  "level": {
    "cefr": null,
    "confidence": 0
  },
  "stats": {
    "sessionsCompleted": 0,
    "totalMinutes": 0
  },
  "skills": {
    "speaking": null,
    "listening": null,
    "grammar": null,
    "vocabulary": null,
    "pronunciation": null
  },
  "focus": [],
  "strengths": [],
  "weaknesses": [],
  "lastSessionId": null
}
```

### CEFR

`level.cefr` records the current estimated CEFR level. v1 allows `null` when there is not enough evidence to estimate CEFR yet, or one of:

```text
A1
A2
B1
B2
C1
C2
```

`null` means "not assessed yet." The initial state MUST NOT assume `A1` only because the learner has not been evaluated.

CEFR progression MUST be conservative. A single lesson should not automatically change the learner's CEFR level.

### Confidence

`level.confidence` is a number from `0.0` to `1.0`.

It represents confidence in the CEFR assessment, not language mastery.

When `level.cefr` is `null`, `level.confidence` MUST be `0`.

### Stats

`stats.sessionsCompleted` is the number of completed sessions persisted through the normal finish flow.

`stats.totalMinutes` is the known total lesson time in minutes. It MUST only include duration that is supported by evidence. Unknown session duration MUST NOT be guessed.

### Skills

`skills` stores internal learner metrics on a `0` to `100` scale for:

```text
speaking
listening
grammar
vocabulary
pronunciation
```

These values:

- are not percentages of fluency;
- do not directly equal CEFR levels;
- may remain stable between sessions;
- may decrease when evidence supports recalibration.

For v1, an unassessed skill SHOULD be represented as `null`, not `0`.

This is the safer semantic choice because `0` can be confused with "assessed and extremely weak." `null` means the skill is known to be unavailable or not yet assessed. A numeric `0` should mean a real measured value of zero according to a future assessment method.

Pronunciation in particular SHOULD remain `null` unless the agent has reliable audio evidence. Text transcripts alone are not sufficient pronunciation evidence.

### Focus, Strengths, Weaknesses, and Last Session

`focus` is a list of active learning topics.

`strengths` is a list of currently observed learner strengths.

`weaknesses` is a list of currently observed learner difficulties.

`lastSessionId` is the session ID of the most recent completed session. It may be `null` before the first completed session. When present, it MUST correspond to an existing file at `sessions/<sessionId>.md`.

## 5. `progress/history.json`

`progress/history.json` stores append-only progress snapshots in normal operation. It is used to understand learner evolution over time.

Example v1 structure:

```json
{
  "version": 1,
  "items": [
    {
      "capturedAt": "2026-09-17T20:30:00-03:00",
      "sessionId": "2026-09-17T233000Z",
      "level": {
        "cefr": "A1",
        "confidence": 0.25
      },
      "skills": {
        "speaking": 18,
        "listening": 15,
        "grammar": 20,
        "vocabulary": 22,
        "pronunciation": null
      }
    }
  ]
}
```

History snapshots MUST NOT duplicate the entire `progress/current.json` file. They should store only the minimum useful state needed to track historical development, such as timestamp, related session, CEFR estimate, assessment confidence, and skill metrics.

## 6. `mistakes/mistakes.json`

`mistakes/mistakes.json` stores consolidated recurring mistakes and learning issues. It is mutable because a mistake can accumulate evidence, change priority, or move through learning statuses.

Example v1 structure:

```json
{
  "version": 1,
  "items": [
    {
      "id": "grammar:past_simple:work",
      "category": "grammar",
      "topic": "past_simple",
      "examples": [
        {
          "wrong": "Yesterday I work.",
          "correct": "Yesterday I worked."
        }
      ],
      "occurrences": 3,
      "firstSeen": "2026-09-10T20:00:00-03:00",
      "lastSeen": "2026-09-17T20:30:00-03:00",
      "status": "learning",
      "priority": 80
    }
  ]
}
```

Field semantics:

| Field | Meaning |
| --- | --- |
| `version` | Format version for this file. Initial value is `1`. |
| `items` | List of consolidated mistake records. |
| `id` | Stable mistake ID. Human-readable deterministic IDs are acceptable in the MVP. |
| `category` | Broad type of issue. Initial categories include `grammar`, `vocabulary`, `sentence_structure`, `listening`, `pronunciation`, and `usage`, but this documentation does not require a closed list yet. |
| `topic` | More specific learning topic, such as `past_simple`. |
| `examples` | Evidence examples for the issue. `wrong` MUST represent an error actually observed from the learner. `correct` may contain the corresponding pedagogical correction. |
| `occurrences` | Count of observed occurrences. It can increase only when there is observable evidence. |
| `firstSeen` | Timestamp when the mistake was first observed. |
| `lastSeen` | Timestamp when the mistake was most recently observed. |
| `status` | Learning status for this mistake. |
| `priority` | Pedagogical review priority on a `0` to `100` scale. It is not absolute severity. |

Mistake statuses:

```text
new
learning
improving
mastered
```

Agents SHOULD update an existing semantically equivalent mistake instead of creating duplicates.

Pedagogical examples that were not produced by the learner may be useful in teaching materials, but they MUST NOT be stored as historical mistake evidence. This keeps `occurrences` auditable.

## 7. `vocabulary/vocabulary.json`

`vocabulary/vocabulary.json` stores consolidated vocabulary learning state.

Example v1 structure:

```json
{
  "version": 1,
  "items": [
    {
      "id": "yesterday",
      "term": "yesterday",
      "translation": "ontem",
      "examples": [
        "Yesterday I worked."
      ],
      "firstSeen": "2026-09-10T20:00:00-03:00",
      "lastSeen": "2026-09-17T20:30:00-03:00",
      "successfulReviews": 2,
      "failedReviews": 1,
      "status": "learning"
    }
  ]
}
```

Field semantics:

| Field | Meaning |
| --- | --- |
| `version` | Format version for this file. Initial value is `1`. |
| `items` | List of vocabulary records. |
| `id` | Stable vocabulary ID. In the MVP, a normalized term can be used when it is unambiguous. |
| `term` | Vocabulary item in the target language. |
| `translation` | Translation or learner-language explanation. It may be `null` when unnecessary. |
| `examples` | Contextual examples from real lesson use or clearly valid pedagogical examples. |
| `firstSeen` | Timestamp when the vocabulary item was first introduced or observed as pedagogically relevant. |
| `lastSeen` | Timestamp when the vocabulary item was most recently used, reviewed, or observed. |
| `successfulReviews` | Number of observed successful reviews. |
| `failedReviews` | Number of observed failed reviews. |
| `status` | Learning status for this vocabulary item. |

Vocabulary statuses:

```text
new
learning
familiar
mastered
```

Agents MUST NOT automatically record every word used by the coach. Vocabulary should be persisted only when it has pedagogical value for the learner.

## 8. `review/queue.json`

`review/queue.json` stores review items needed by the future review system. v1 defines the state shape only; it does not define a spaced repetition algorithm.

Example v1 structure:

```json
{
  "version": 1,
  "items": [
    {
      "id": "vocabulary:yesterday",
      "type": "vocabulary",
      "referenceId": "yesterday",
      "stage": 1,
      "dueAt": "2026-09-18",
      "priority": 50,
      "lastResult": "success"
    }
  ]
}
```

Field semantics:

| Field | Meaning |
| --- | --- |
| `version` | Format version for this file. Initial value is `1`. |
| `items` | List of review queue records. |
| `id` | Stable review item ID. |
| `type` | Review item type. v1 values are `vocabulary` and `mistake`. |
| `referenceId` | ID of the referenced vocabulary or mistake item. |
| `stage` | Non-negative integer. Its scheduling meaning will be defined by the Review System later. |
| `dueAt` | Due date for review. v1 uses `YYYY-MM-DD` when only a date is needed. |
| `priority` | Pedagogical priority on a `0` to `100` scale. |
| `lastResult` | Most recent review result. |

`lastResult` may be:

```text
success
failure
partial
null
```

No interval rules are defined in this stage.

Grammar content that needs review MUST be represented through a mistake record instead of a standalone review type, because v1 does not define a separate grammar source of truth with stable IDs. For example, grammar review for past simple can reference a mistake such as `grammar:past_simple:work`:

```json
{
  "type": "mistake",
  "referenceId": "grammar:past_simple:work"
}
```

## 9. Session Identity

Each persisted session MUST have a stable session ID. The `sessionId` MUST correspond to the Markdown filename without the `.md` extension.

v1 uses a compact UTC timestamp with seconds:

```text
YYYY-MM-DDTHHmmssZ
```

Example:

```text
2026-09-18T012900Z
```

The corresponding file path is:

```text
sessions/2026-09-18T012900Z.md
```

The session ID is derived from `recordedAt`, the technical timestamp when `/finish` persists the session. `recordedAt` is system metadata, not a pedagogical inference and not necessarily the start time of the lesson.

The Markdown content uses ISO 8601 UTC for `recordedAt`:

```text
2026-09-18T01:29:00Z
```

The filename and `sessionId` use the compact corresponding form:

```text
2026-09-18T012900Z
```

This convention is:

- readable;
- lexicographically sortable by time;
- compatible with filenames;
- timezone-unambiguous because `Z` indicates UTC.

Including seconds makes collisions less likely. Still, if `sessions/<sessionId>.md` already exists, the agent MUST NOT overwrite it. The agent should treat this as a conflict and generate a non-conflicting identity explicitly. No complex suffix algorithm is defined in this stage.

## 10. `sessions/*.md`

Session files are human-readable Markdown records. They preserve narrative evidence and learning context that should not be compressed into structured JSON alone.

Stable v1 structure:

```md
# English Session

## Metadata

- Session ID:
- Recorded at:
- Started at:
- Duration minutes:
- Modality:
- CEFR before:
- CEFR after:

## Focus

...

## Summary

...

## Vocabulary

...

## Mistakes

...

## Strengths

...

## Difficulties

...

## Evidence Notes

...

## Next Lesson Recommendation

...
```

`Modality` initially supports:

```text
text
voice
mixed
```

`Duration` may be absent or unknown. Agents MUST NOT invent duration.

`Recorded at` must exist when a session is persisted.

`Started at` may be a real timestamp when reliable evidence exists, or `null` when the start time is unavailable. Agents MUST NOT calculate `startedAt` from `recordedAt` minus an estimated duration unless the duration and temporal relationship are both supported by evidence.

`Duration minutes` should be represented conceptually as `durationMinutes`. It may be a number or `null`.

`CEFR before` and `CEFR after` may be `null` or conceptually unknown when CEFR has not been assessed.

If the session did not provide reliable audio evidence to the agent, pronunciation should not be evaluated as if it had been heard. The limitation should be recorded in `Evidence Notes`.

## 11. Nullability and Unknown State

The data model distinguishes unknown state from empty, zero, and absent state.

| Representation | Meaning |
| --- | --- |
| `null` | Known to be unassessed, unavailable, not applicable, or unknown when the contract permits it. |
| `0` | A real numeric value of zero. It is not a generic unknown placeholder. |
| `""` | Empty string. It should not be used as a substitute for unknown, except for configurable template fields that are intentionally blank. |
| `[]` | Known empty collection. |
| Absent field | Allowed only when the contract declares the field optional. |

Agents MUST avoid ambiguity. When evidence is missing, they should use the contract-defined unknown representation instead of fabricating values.

## 12. IDs

IDs should follow these principles:

- stable over time;
- lowercase when applicable;
- independent of array position;
- not changed only because descriptive text changed;
- deterministic and human-readable when practical;
- unique within their file or declared namespace;
- references should point to existing IDs.

The MVP does not require UUIDs. Human-readable deterministic IDs are acceptable when they avoid collisions and remain stable.

## 13. Referential Integrity

References MUST NOT become silent orphans.

For example:

```text
type: vocabulary
referenceId: yesterday
```

requires an item with:

```text
id: yesterday
```

in:

```text
vocabulary/vocabulary.json
```

Similarly, review items of type `mistake` should reference existing mistake IDs when `referenceId` is used for that type.

`progress/current.json` uses `lastSessionId` to reference the latest completed session. When not `null`, it MUST correspond to an existing file:

```text
sessions/<lastSessionId>.md
```

If a referenced item is missing, agents should report the inconsistency instead of silently inventing the missing record or ignoring the broken reference.

## 14. History Rules

Normal persistence follows these mutability rules:

- `sessions/` is append-only learning history;
- `progress/history.json` is append-only in the normal flow;
- `progress/current.json` is mutable consolidated state;
- `mistakes/mistakes.json` is mutable consolidated state;
- `vocabulary/vocabulary.json` is mutable consolidated state;
- `review/queue.json` is mutable;
- `config/student.json` changes only when the learner configuration actually changes.

Historical data MUST NOT be silently deleted or rewritten for convenience. Clearly justified administrative corrections may be possible, but they are not normal lesson persistence.

## 15. Data Ownership Table

| Data | Source of truth | Mutable |
| --- | --- | --- |
| Student configuration | `config/student.json` | Yes |
| Current progress | `progress/current.json` | Yes |
| Progress history | `progress/history.json` | Append-only |
| Mistakes | `mistakes/mistakes.json` | Yes |
| Vocabulary | `vocabulary/vocabulary.json` | Yes |
| Reviews | `review/queue.json` | Yes |
| Sessions | `sessions/*.md` | Append-only |

## 16. Versioning

Every structured JSON file uses:

```json
{
  "version": 1
}
```

as the format version for that file.

A future breaking format change, such as:

```text
v1 -> v2
```

requires an explicit migration strategy.

Adding a backward-compatible optional field does not necessarily require a new major version.

No migration process is implemented in this stage.

## 17. Out of Scope

This document does not define:

- CEFR assessment algorithm;
- skill scoring formula;
- spaced repetition algorithm;
- backend;
- database;
- API;
- multi-user support;
- authentication;
- JSON Schemas;
- scripts or tooling.

These subjects should be handled separately.
