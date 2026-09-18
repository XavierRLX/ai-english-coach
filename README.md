# AI English Coach

Open-source English coaching workflow that uses ChatGPT for lessons and GitHub as persistent, versioned learning memory.

## What it is

AI English Coach is not a web app. It is a repository contract for running English lessons with ChatGPT while keeping learner progress outside a single chat.

```text
ChatGPT conversation
        ↓
      lesson
        ↓
     /finish
        ↓
private learner repository
        ↓
next /start loads prior context
```

The public repository provides the behavior contract, schemas, documentation, and example files. Your real learning data should live in a separate private repository.

## Why

A chat is useful for the current lesson, but long-term learning needs structured state.

This project keeps:

- learner configuration;
- current progress;
- progress history;
- recurring mistakes;
- vocabulary;
- review state;
- session history.

Git provides versioning. ChatGPT provides the tutoring experience.

## Quick Start

### 1. Create a private learner repository

Create an empty private GitHub repository, for example:

```text
english-progress
```

### 2. Clone this template into your learner folder

```bash
git clone https://github.com/XavierRLX/ai-english-coach.git english-progress
cd english-progress
```

### 3. Keep the public project as `upstream`

```bash
git remote rename origin upstream
git remote add origin https://github.com/YOUR_USER/english-progress.git
git push -u origin main
```

### 4. Initialize real learner files

```bash
cp config/student.example.json config/student.json
cp progress/current.example.json progress/current.json
cp progress/history.example.json progress/history.json
cp mistakes/mistakes.example.json mistakes/mistakes.json
cp vocabulary/vocabulary.example.json vocabulary/vocabulary.json
cp review/queue.example.json review/queue.json
```

Edit `config/student.json`, then commit and push:

```bash
git add config progress mistakes vocabulary review
git commit -m "chore: initialize learner profile"
git push origin main
```

### 5. Connect GitHub to ChatGPT

Authorize ChatGPT to access your learner repository. Repository access and write capabilities depend on the ChatGPT environment and connected GitHub integration.

Connecting GitHub does **not** automatically tell ChatGPT which repository is your learner memory.

## First Chat Prompt

Start a new text chat and send:

```text
Use YOUR_USER/english-progress as my AI English Coach learner repository.

Read AGENTS.md and use this repository as my persistent English-learning memory.

Then execute /start.
```

The commands below are project conventions defined by `AGENTS.md`. They are **not native ChatGPT slash commands**.

## Commands

| Command | Purpose |
| --- | --- |
| `/start` | Load learner memory and prepare the next lesson. Does not change state. |
| `/lesson` | Start or continue the planned lesson. |
| `/finish` | Finish the lesson and persist only evidence supported by the conversation. |
| `/review` | Focus on due review content. |
| `/status` | Show current progress without changing state. |

## Text + Voice workflow

Recommended flow:

```text
TEXT
/start
  ↓
load GitHub learner context
  ↓
/lesson
  ↓
TEXT or VOICE
  ↓
return to TEXT
  ↓
/finish
  ↓
persist session + learning state
```

Run `/start` in text mode so repository context can be loaded before the lesson.

Voice can continue the same lesson using context already present in the conversation, but the workflow must not assume repository tools are available while Voice is active.

After a Voice lesson, return to text and run `/finish`.

Voice transcripts may be incomplete. Missing speech must not be reconstructed, and transcript text alone is not reliable pronunciation evidence.

## Public template vs private learner repository

### Public: `ai-english-coach`

Contains reusable project files:

- `AGENTS.md`;
- documentation;
- schemas;
- `*.example.*` templates.

It should not contain your personal learning history.

### Private: your learner repository

Recommended example:

```text
YOUR_USER/english-progress
```

It contains real learner state:

```text
config/student.json
progress/current.json
progress/history.json
mistakes/mistakes.json
vocabulary/vocabulary.json
review/queue.json
sessions/*.md
```

Recommended remotes:

```text
origin   -> your private learner repository
upstream -> XavierRLX/ai-english-coach
```

## If ChatGPT cannot write to GitHub

The project can still be used.

The coach should prepare the exact file changes needed for `/finish`. Apply them locally, commit, and push them yourself. The next `/start` can then load the updated state.

## Repository structure

```text
AGENTS.md                    AI behavior contract
config/                      learner configuration
progress/                    current state and history
mistakes/                    recurring mistake state
vocabulary/                  vocabulary state
review/                      review queue
sessions/                    historical lesson records
schemas/                     JSON data contracts
docs/                        detailed project documentation
```

Files matching `*.example.*` are templates, not learner evidence. In particular, `sessions/session.example.md` is not a real lesson.

## Privacy

Learner repositories may contain personal conversation history and learning data.

Use a private repository for real learner state.

Do not store passwords, API keys, access tokens, secrets, or sensitive information that is unnecessary for language learning.

## Documentation

- [Getting Started](docs/03-getting-started.md)
- [Study Flow](docs/04-study-flow.md)
- [Data Model](docs/05-data-model.md)
- [Agent Behavior](docs/06-agent-behavior.md)
