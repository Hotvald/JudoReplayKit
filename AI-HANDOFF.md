# AI Contributor Handoff

This repository is the shared, public project log for JudoReplayKit.

## Before doing work

1. Read `README.md`, `PROJECT-LOG.md`, `KNOWN-ISSUES.md`, `TEST-LOG.md`, and `DECISIONS.md`.
2. Treat the project log as context, not as proof that a local file currently has the same contents.
3. Ask for or inspect the exact current Rev.8 files before reviewing or editing them.
4. Never treat an old production file as the current Rev.8 file.

## Safety boundaries

- Production installation: `D:\JudoReplayKit`. Do not modify it.
- Isolated Rev.8 test installation: `D:\JudoReplayKit_Rev8_Test`.
- Do not stop or kill running services unless there is a clear, approved reason.
- Do not repeat checks already marked as passed unless a specific change warrants retesting.

## Engineering expectations

- The live stream and continuous recording must not be interrupted by replay.
- Replay must be independent and immediately accessible as an overlay on the live picture.
- Keep confirmed facts, observed symptoms, and hypotheses clearly separated.
- Collect all defects and present one consolidated repair plan before editing.
- Do not claim a fix is complete without testing the affected behavior.

## Public repository hygiene

Never commit:
- Chat exports or raw private conversations.
- Passwords, API keys, access tokens, or other secrets.
- Personal data, private IP/network details, or private match recordings.
- Unreviewed logs that may contain credentials or personal information.

Only publish curated technical information that is appropriate for a public repository.


## Shared history is mandatory

Before making suggestions, reviewing code, diagnosing defects, or proposing changes, every AI assistant must read the relevant history in this public repository. At minimum, read `README.md`, `PROJECT-LOG.md`, `KNOWN-ISSUES.md`, `TEST-LOG.md`, `DECISIONS.md`, and this file. The repository is the shared historical knowledge base; an assistant must not assume that its own conversation contains the complete project history.

## Recording contributions and disagreements

- Contributions from the user and from each AI assistant must be recorded with source attribution and date/revision where available.
- Preserve original wording when archiving a contribution. Do not silently rewrite, filter, or replace another contributor's analysis.
- If another assistant disagrees, add the disagreement as a separate, attributed entry. Keep both positions visible until the user resolves the issue or evidence does.
- Corrections and later clarifications are additive historical entries; do not erase the earlier record.
- A summary must be labelled as a summary. Never present it as a verbatim archive.
- Public-repository hygiene still applies: do not publish secrets, credentials, personal data, private network details, or private recordings. If a source contains such material, flag it and agree a safe archival treatment with the user rather than silently altering the source.
- AI assistants without write access can contribute by giving their complete response to the user, who can pass it to the designated log maintainer for archival.
