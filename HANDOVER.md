# Handover

Where everything lives, for whoever runs this site next.

## Accounts

| Service | What it does | Owned by | Sign-in |
| --- | --- | --- | --- |
| GitHub | Code repository | Ronan O'Sullivan (personal account `ronanos1979`) | GitHub |
| Supabase | Database, login, photo storage | Ronan O'Sullivan | GitHub (personal) |
| Vercel | Hosting | Ronan O'Sullivan | GitHub (personal) |
| Resend | Sending email | Ronan O'Sullivan | GitHub (personal) |
| Club Gmail | Sender and reply-to address for club emails | Club | Password held by Ronan |
| Domain | Not registered yet | — | — |

All hosting accounts are under Ronan's personal GitHub. To hand the site over:

1. Transfer the GitHub repository to the new owner, or have them fork it.
2. Add the new owner to the Supabase organization, then make them owner.
3. Transfer the Vercel project to the new owner's account.
4. Add the new owner to the Resend team.
5. Hand over the club Gmail password and the domain registrar login.

## Secrets

Keys and passwords are never stored in this repository. They live in:
- `.env.local` on the developer's machine (not committed)
- Vercel project settings → Environment Variables

`.env.example` lists the variable names needed.

## Deploying

Pushing to the `main` branch deploys automatically through Vercel.
