# CLAUDE.md — Wolves Hurling Club Site

Standing instructions for Claude Code. Read this and SPEC.md at the start of every session.

## What this is
A private, mobile-friendly website for Wolves Hurling club (30-40 adult members): board announcements, match and training schedule with calendar feed, availability (RSVP), private questions to the board, photo gallery. Full requirements are in `SPEC.md`. SPEC.md is the master copy. If code and spec disagree, ask before changing either.

## Stack
- Next.js (App Router) with TypeScript, mobile-first
- Supabase: Postgres, magic-link auth, private storage bucket for photos
- Resend for email
- Hosted on Vercel
- Tailwind for styling

Do not add other frameworks, databases or paid services without asking.

## How we work
- Build one step from the "Build order" section of SPEC.md per session, unless told otherwise.
- Ask before changing the data model. Schema changes go in Supabase migration files, committed to the repo.
- Keep it simple. 40 users. No microservices, no caching layers, no premature abstraction.
- Every page and API route checks the member is active. Access rules are also enforced with Supabase row-level security. Never rely on UI checks alone.
- Times stored in UTC, displayed in America/New_York.
- Test on a phone-width screen before calling a feature done.

## Public repo rules
This repository is public. Before every commit, check:
- No keys, tokens or passwords. Secrets live in `.env.local` (git-ignored) and in Vercel's environment settings. Keep `.env.example` updated with variable names only, no values.
- No real member data. Seed and test data use made-up names, emails and phone numbers.
- No club-private details (venue access codes, fees, personal contact details).
If a secret is ever committed, stop and tell me so the key can be rotated.

## Third-party code
Borrowing ideas and structure from other projects is fine. If you copy actual code from another repository (for example OpenLeague, which is Apache 2.0), tell me first and keep its licence notice.

## End of every session
Append an entry to `PROGRESS.md` at the top, in this format, and commit it:

```
## YYYY-MM-DD — <short title>
- Built: what was completed
- Decided: any decisions made and why
- Next: what the next session should pick up
- Open questions: anything needing a decision from Ronan or the board
```

Use conventional commit messages (feat:, fix:, docs:, chore:).
