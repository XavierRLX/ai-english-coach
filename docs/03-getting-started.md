# Getting Started

This guide takes a new user from the public template to a working private learner repository and a first persisted lesson.

The expected audience is comfortable with Git, GitHub, and a terminal. No backend, database, package installation, or application build is required.

## 1. Architecture in one minute

AI English Coach separates the reusable project from personal learner data.

```text
PUBLIC
XavierRLX/ai-english-coach
    |
    | template / upstream
    v
PRIVATE
YOUR_USER/english-progress
    |
    +-- learner state
    +-- lesson history
    +-- mistakes
    +-- vocabulary
    +-- reviews
```

ChatGPT is the tutor and current lesson runtime.

GitHub is the persistent memory layer.

`AGENTS.md` tells the AI how to teach, load context, interpret project commands, and persist learning state.

## 2. Create the learner repository

Create a new empty GitHub repository.

Recommended name:

```text
english-progress
```

Recommended visibility:

```text
Private
```

Do not initialize it with a README, license, or `.gitignore` if you want to follow the commands below exactly.

## 3. Clone the public project as your learner repository

From your normal projects directory:

```bash
git clone https://github.com/XavierRLX/ai-english-coach.git english-progress
cd english-progress
```

The clone initially points `origin` to the public project. Change that relationship.

Rename the public remote to `upstream`:

```bash
git remote rename origin upstream
```

Add your private repository as `origin`:

```bash
git remote add origin https://github.com/YOUR_USER/english-progress.git
```

Push the existing project history to your private repository:

```bash
git push -u origin main
```

Verify:

```bash
git remote -v
```

Expected relationship:

```text
origin   -> https://github.com/YOUR_USER/english-progress.git
upstream -> https://github.com/XavierRLX/ai-english-coach.git
```

## 4. Initialize real learner files

The public repository contains template files. They are not learner state.

Create the real files:

```bash
cp config/student.example.json config/student.json
cp progress/current.example.json progress/current.json
cp progress/history.example.json progress/history.json
cp mistakes/mistakes.example.json mistakes/mistakes.json
cp vocabulary/vocabulary.example.json vocabulary/vocabulary.json
cp review/queue.example.json review/queue.json
```

Do not rename or copy `sessions/session.example.md` into a real lesson. Real session files are created by successful `/finish` operations.

## 5. Configure the learner

Open:

```text
config/student.json
```

Set the learner name, language preferences, goal, correction style, and interests.

Example:

```json
{
  "version": 1,
  "name": "Alex",
  "nativeLanguage": "pt-BR",
  "targetLanguage": "en-US",
  "goal": {
    "primary": "conversation",
    "description": "Speak English comfortably in daily and professional situations"
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

The initial progress file may contain:

```text
cefr = null
skills = null
sessionsCompleted = 0
```

That is expected. Unknown level does not mean A1.

## 6. Commit the initialized learner state

```bash
git add config progress mistakes vocabulary review
git commit -m "chore: initialize learner profile"
git push origin main
```

Your private repository is now the source of truth for the learner.

## 7. Connect GitHub to ChatGPT

Connect or authorize GitHub in the ChatGPT environment you intend to use.

Make sure the connected GitHub account can access your private learner repository.

Important:

- connecting GitHub does not automatically identify the learner repository;
- repository read/write capabilities depend on the ChatGPT environment and connected integration;
- the workflow must not assume write access is always available.

## 8. Start the first chat

Use a text chat.

Send:

```text
Use YOUR_USER/english-progress as my AI English Coach learner repository.

Read AGENTS.md and use this repository as my persistent English-learning memory.

Then execute /start.
```

The project commands are conventions interpreted from `AGENTS.md`. They are not native ChatGPT commands.

A correct first `/start` should load the real learner files and notice that no reliable assessment exists yet.

If the repository contains:

```text
cefr = null
skills = null
sessionsCompleted = 0
```

the coach must not assume A1. The first lessons can act as a natural diagnostic.

## 9. Start the lesson

Send:

```text
/lesson
```

Continue in text, or switch to Voice after `/start` has already loaded the learner context.

### If you use Voice

Recommended sequence:

```text
TEXT: /start
TEXT: /lesson
      ↓
VOICE: conduct the lesson
      ↓
TEXT: /finish
```

Voice should use the context already present in the conversation.

Do not rely on Voice having direct repository or connected-app access.

The coach must not invent words missing from an incomplete Voice transcript. Transcript text alone must not be treated as reliable pronunciation evidence.

## 10. Finish the lesson

Return to text and send:

```text
/finish
```

When write access is available, a successful finish may update:

```text
sessions/<sessionId>.md
mistakes/mistakes.json
vocabulary/vocabulary.json
review/queue.json
progress/current.json
progress/history.json
```

Only evidence supported by the conversation should be persisted.

A short lesson may legitimately leave:

- CEFR unchanged or `null`;
- skill scores unchanged or `null`;
- vocabulary empty;
- review queue empty;
- duration unknown.

That is preferable to fabricated progress.

## 11. Verify persistence

Check the private repository.

After the first persisted lesson, you should normally see a new file similar to:

```text
sessions/2026-09-18T035400Z.md
```

and:

```text
progress/current.json
```

should reference that session through `lastSessionId`.

Now start again:

```text
/start
```

The coach should use prior state instead of starting from zero. It may consider:

- the latest session;
- current focus;
- strengths;
- weaknesses;
- recurring mistakes;
- due reviews.

This second `/start` is the basic persistence test for the MVP.

## 12. If ChatGPT cannot write to the repository

The learner workflow still works.

Ask the coach to prepare the exact persistence changes required by `/finish`.

The coach should provide enough information to update the affected files manually.

Then apply, commit, and push the changes locally.

Example:

```bash
git add progress mistakes vocabulary review sessions
git commit -m "study: persist English session"
git push origin main
```

The next `/start` can read the updated state.

## 13. Keeping the local clone synchronized

If ChatGPT or another tool writes directly to the remote private repository, your local clone may become behind `origin/main`.

Update it with:

```bash
git pull origin main
```

Do this before making new local edits when remote changes already exist.

## Troubleshooting

### ChatGPT cannot find the repository

Confirm:

1. GitHub is connected to the ChatGPT environment;
2. the connected GitHub account can access the private repository;
3. you gave the full repository name, for example `YOUR_USER/english-progress`.

Then retry the first chat prompt.

### The private repository does not appear

Check GitHub authorization for the connected integration. Private repositories may require explicit access depending on the integration configuration.

### Voice cannot access GitHub

This workflow does not depend on repository access while Voice is active.

Run `/start` in text first, conduct the lesson in Voice, then return to text for `/finish`.

### `/finish` cannot write

Use the manual persistence fallback described above. Repository write capability is environment-dependent.

### The coach is reading example files as learner data

Stop the lesson and point the coach to `AGENTS.md`.

Files matching:

```text
*.example.*
```

are templates.

`sessions/session.example.md` is not a real lesson.

### Local Git is behind after ChatGPT wrote remotely

Run:

```bash
git pull origin main
```

Then continue working locally.
