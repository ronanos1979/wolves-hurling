# Wolves Hurling Club Site — Spec

Oct 13, 2026 · Ronan O'Sullivan

## Overview

A private, mobile-friendly website for Wolves Hurling club: board announcements, a match and training schedule people can subscribe to in their calendar, private questions to the board, and a shared photo gallery.

- **Audience:** 30 to 40 adult members. Players, manager, board.
- **Not a native app.** A website that works well on phones. No App Store, one codebase.
- **WhatsApp stays.** The team group remains the place for chat. The site is the official record, and announcements can be pushed into WhatsApp with one tap.
- **Private by default.** Nothing is visible to anyone who is not an approved member.
- **Cheap to run.** Free hosting tiers. Domain name is the only real cost.

## What already exists on GitHub

No MIT-licensed project is a usable match. The one serious candidate is OpenLeague, which is Apache 2.0 licensed. That licence is as permissive as MIT for our purposes.

| Project | Licence | What it has | Verdict |
| --- | --- | --- | --- |
| [OpenLeague](https://github.com/mbeacom/openleague) | Apache 2.0 | Email/password login, invite-only team membership, admin and member roles, games and practices, calendar view, RSVP, email notifications, optional photo galleries. Next.js, Postgres, Vercel. Actively maintained. Also offered free for teams at [openl.app](https://openl.app). | Closest match. Built for ice hockey leagues, so much of it (rinks, gear, paid signups) is noise for us. No board-only announcement feed or calendar subscription feed listed. Fork or borrow patterns. |
| [ClubMate](https://github.com/BEGINNERUSER-git/clubmate) | MIT | Notice board, event calendar, role-based dashboards. | Student project, structure still marked as planned. Not usable. |
| [uni Club Management System](https://github.com/oussemakh1/uni) | MIT | Members, roles, events, alerts. PHP and MySQL. | University clubs, dated stack. Not usable. |
| [Sports League Management System](https://github.com/kaimg/Sports-League-Management-System) | MIT | Teams, matches, scores, standings. | League stats tool, not club communication. Not relevant. |
| [player-availability-tracker](https://github.com/cherbie/player-availability-tracker) | Not stated | Fixtures, game and training calendars, availability, roles. | A one-club project like ours. Useful as a reference only. |

**What this means.** Two realistic paths:

1. **Build our own, small.** Our feature set is narrow. A focused build with Claude Code is a few sessions, not months. We borrow OpenLeague's choices (Next.js, Postgres, Auth.js, invite links) where they fit.
2. **Use openl.app as it is.** Free, already built, includes RSVP. But it is a general team app with no WhatsApp share, no club branding, and it is not ours.

This spec assumes path 1. It is worth a 15-minute look at openl.app before starting, so the board knows what the free option looks like.

## Shortcomings in our plan

The biggest gap is availability. Every comparable app has it, and the manager will use it more than anything else on the list.

| Gap | Why it matters | Fix |
| --- | --- | --- |
| **No availability / RSVP** | Manager needs to know who is coming to Saturday's match. Today that is chased on WhatsApp. | Add Going / Not going / Maybe on every fixture. Manager sees the count and names. Move into v1. |
| **Calendar feed can't use a login** | Phone calendars fetch a plain URL. They can't log in. A shared URL would leak the schedule to anyone it's forwarded to. | Each member gets a private feed URL with a secret token. Removing a member kills their URL. |
| **Calendars refresh slowly** | Google Calendar can take up to a day to re-fetch a subscribed feed. A same-day cancellation won't show in time. | Cancellations always go out as an announcement too (email plus WhatsApp share). Say this on the calendar page. |
| **Open sign-up plus approval invites noise** | Anyone who finds the URL can create a pending account. Low risk, but the board has to filter strangers. | Invite links instead. Admin sends a link; only that link creates an account. Approval stays as a fallback. |
| **Email may land in spam** | Mail from a new sender with no domain setup often goes to junk. Announcements silently missed. | Own domain with SPF and DKIM set up through the email provider. Send a test to Gmail, Outlook and iCloud before launch. |
| **One person holds everything** | Board terms end. If only one person can run the site or knows the logins, it dies when they step down. | At least two admins. Hosting, domain and email accounts owned by a club email address, not personal ones. Short handover note in the repo. |
| **Photos carry privacy risk** | Members may not want their face on the site. People will ask for photos to be taken down. | Opt-out flag on each profile. Uploader or admin can delete any photo. Admin approval before a photo appears. |
| **Questions can fall through the cracks** | A form that just sends email has no owner and no record of what was answered. | Questions stored in the site with status (open, answered). Board sees the open list. Reply goes back to the asker by email. |
| **Removing a member** | Someone leaves the club or falls out. Their access has to end the same day. | Admin "remove member" ends sessions, kills the calendar token, and removes them from the email list. |
| **Adoption** | The site only works if people use it. WhatsApp is the habit. | Launch with the schedule and RSVP already filled in. Put the site link in the WhatsApp group description. Every WhatsApp share links back to the site. |

Not gaps, but worth stating: no dues or payments, no youth safeguarding features, no push notifications. All deliberate for v1.

## Roles and permissions

Three roles. A person has exactly one.

| Action | Member | Board / Manager | Admin |
| --- | --- | --- | --- |
| See schedule, announcements, photos | Yes | Yes | Yes |
| RSVP to fixtures | Yes | Yes | Yes |
| Subscribe to calendar feed | Yes | Yes | Yes |
| Ask the board a question | Yes | Yes | Yes |
| Upload photos (pending approval) | Yes | Yes | Yes |
| Post announcements | No | Yes | Yes |
| See and answer questions | No | Yes | Yes |
| Add and edit fixtures | No | Yes | Yes |
| See RSVP names and counts | Own only | Yes | Yes |
| Approve photos | No | Yes | Yes |
| Invite, approve, remove members | No | No | Yes |
| Change roles | No | No | Yes |

At least two admins at all times. The site refuses to remove or demote the last admin.

## Version 1 features

Six features. Each lists what "done" means.

### 1. Membership and login

- Admin creates an invite for a name and email. The site emails a one-time link, valid 7 days.
- The link opens a short form: name, phone (optional), photo opt-out. Account is active on submit.
- Fallback: a public "request to join" form creates a pending account. Admin approves or rejects. Pending users see only a holding page.
- Login by emailed magic link. No passwords to forget or reset.
- Done when: a person without an invite or approval cannot see any club content, by URL or otherwise.

### 2. Board announcements

- Board and admins post a title, body and optional photo or attachment.
- Members see a feed, newest first. Pinned posts stay at the top.
- No comments or replies. Each post has an "Ask the board about this" link that opens the question form with the post attached.
- On publish, every active member gets an email.
- After publish, the poster sees **Share to WhatsApp**. It opens WhatsApp with the title, a short excerpt and a link back to the post. The poster picks the group and sends.
- Done when: a post goes out by email within a minute, and the WhatsApp share opens correctly on iPhone and Android.

### 3. Schedule and calendar feed

- Board adds fixtures: type (match or training), date, start and end time, venue with map link, opponent, notes.
- List view by default, next fixture first. Past fixtures collapse below.
- Each member has a private calendar feed URL (.ics) with "Add to Apple Calendar" and "Add to Google Calendar" buttons.
- Edits and cancellations update the feed. A cancellation also prompts the board to post an announcement.
- Done when: a member subscribes once and new fixtures appear in their phone calendar without further action.

### 4. Availability (RSVP)

- Going / Not going / Maybe on every fixture, from the list or the fixture page.
- Optional reminder email 48 hours before a match to anyone who hasn't answered.
- Board sees counts and names per fixture.
- Done when: the manager can see who is available for a match on one screen.

### 5. Questions to the board

- Form: subject, message, optional link to an announcement or fixture.
- Visible only to board and admins. Status: open, answered.
- Board replies in the site; the reply is emailed to the asker.
- Done when: the board has one list of open questions and nothing is lost in personal inboxes.

### 6. Photo gallery

- Members upload one or several photos from their phone, optionally tagged to a fixture.
- Photos are resized on upload. Original size capped at 10 MB each.
- New photos wait for board approval before members see them.
- Uploader or admin can delete a photo at any time.
- Done when: a member uploads from a phone at the pitch and the photo appears after approval.

## Not in v1

- Two-way WhatsApp sync. Not possible within WhatsApp's rules.
- Text message (SMS) announcements. Easy to add later through Twilio if email proves too slow.
- Push notifications. Unreliable on iPhone unless the site is added to the home screen.
- Dues and payments.
- Match stats, scores and team selection.
- Native iPhone or Android app.

## Tech stack and running costs

Everything fits in free tiers. Expected cost is a domain name, roughly $15 a year.

| Layer | Choice | Free tier fit for 40 people |
| --- | --- | --- |
| App | Next.js (TypeScript), mobile-first | Same choice as OpenLeague. Well supported by Claude Code. |
| Hosting | Vercel | Hobby tier is enough for a non-commercial club site. |
| Database, login, photo storage | [Supabase](https://supabase.com/pricing) | 500 MB database and 1 GB file storage. Magic-link login built in. |
| Email | [Resend](https://resend.com/docs/knowledge-base/resend-email-quota) | 100 emails a day, 3,000 a month. One announcement to 40 people uses 40. |
| Domain | e.g. wolveshurling.com | Needed for email to land in inboxes. |

Three limits to design around:

- **Supabase pauses idle free projects** after 7 days with no activity, and someone has to restore it by hand. Off-season this will happen. Fix: a daily scheduled job (the RSVP reminder job) that touches the database.
- **Resend's 100-a-day cap.** Two announcements plus reminders in one day could hit it. Fix: batch reminders, and fall back to the $20/month plan only if it becomes a real problem.
- **1 GB of photos.** Resized to around 300 KB each, that is roughly 3,000 photos. Several seasons' worth.

## Data model

Seven tables. Claude Code can generate the schema from this.

| Table | Key fields | Notes |
| --- | --- | --- |
| members | id, name, email, phone, role (member / board / admin), status (pending / active / removed), photo_opt_out, calendar_token, created_at | calendar_token is a long random string, regenerated on removal |
| invites | id, email, name, token, expires_at, used_at, created_by | One-time use, 7-day expiry |
| announcements | id, title, body, attachment_url, pinned, author_id, published_at | No replies table by design |
| fixtures | id, type (match / training), starts_at, ends_at, venue, map_url, opponent, notes, status (scheduled / cancelled), updated_at | Times stored in UTC, shown in America/New_York |
| rsvps | fixture_id, member_id, response (going / not_going / maybe), updated_at | One row per member per fixture |
| questions | id, member_id, subject, body, related_announcement_id, related_fixture_id, status (open / answered), answer, answered_by, answered_at | Board-only visibility |
| photos | id, uploader_id, fixture_id, storage_path, status (pending / approved), approved_by, created_at | Deleted photos removed from storage, not just hidden |

## Security and privacy

The one hard rule: no club content is reachable without an active, approved account.

- Every page and API route checks the member's status is active. Pending and removed accounts see a holding page only.
- Access rules enforced in the database (Supabase row-level security), not just in the page code. A bug in the UI must not leak data.
- Photos stored in a private bucket and served through short-lived signed URLs. A copied photo link stops working.
- Calendar feed URLs carry a per-member secret token. Feed shows fixtures only: no names, phone numbers or RSVPs.
- Removing a member ends their sessions, regenerates their calendar token and drops them from email sends, immediately.
- No phone numbers or emails visible to other members unless the board decides otherwise.
- Every email has a footer explaining why they got it and who to contact.
- Accounts for hosting, database, email and domain registered to a club email address, with two people holding access.

## Build order

Build in this order. Each step is usable on its own, so the site can go live after step 4.

1. **Setup.** Club email address, domain, Vercel, Supabase and Resend accounts. Email domain verification (SPF, DKIM).
2. **Membership and login.** Invites, magic-link login, approval queue, admin member list, remove member. Row-level security from the start.
3. **Schedule and calendar feed.** Fixture admin, list view, private .ics feed, add-to-calendar buttons.
4. **Availability.** RSVP buttons, board view of counts and names. **Soft launch here** with the board and a few players.
5. **Announcements.** Post, email to all, WhatsApp share button, pinned posts.
6. **Questions.** Form, board inbox with status, emailed replies.
7. **Photos.** Upload, resize, approval queue, delete.
8. **Reminders and keep-alive.** Daily job for RSVP reminders that also keeps Supabase awake.
9. **Full launch.** Invite everyone. Link in the WhatsApp group description.

**Handing to Claude Code.** Start each session with one numbered step: "Read SPEC.md. Build step 3. Ask before changing the data model." Test each step on your own phone before moving on. Keep a short `HANDOVER.md` with where each account lives and how to deploy.

## Open decisions for the board

- [ ] Build our own, or try openl.app first?
- [ ] Can members see who else is going to a fixture, or only the board?
- [ ] Who are the two admins?
- [ ] Domain name.
- [ ] Do photos need approval, or is delete-after enough for a 40-person club?
- [ ] Should members see each other's phone numbers?

## Sources

- [OpenLeague on GitHub](https://github.com/mbeacom/openleague)
- [Awesome Sports Club Management list](https://github.com/ishandutta2007/Awesome-Sports-Club-Management)
- [Resend sending limits](https://resend.com/docs/knowledge-base/resend-email-quota)
- [Supabase free tier limits (Makerkit)](https://makerkit.dev/blog/saas/supabase-pricing)
- [WhatsApp Groups API limits (Gupshup docs)](https://docs.gupshup.io/docs/whatsapp-groups-api)
