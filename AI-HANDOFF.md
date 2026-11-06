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
