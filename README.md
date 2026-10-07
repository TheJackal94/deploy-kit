# deploy-kit

One reusable deploy workflow for all of Riaan's projects. Push to `main` and the project builds (if it needs to),
deploys to wherever it's hosted, checks the live site loads, and sends a ✅ or 🔴 to the Ops bot on Telegram.

## Targets

| target | For | Secrets the project needs |
|---|---|---|
| `cpanel-ftp` | Any cPanel host (Axxess, Afrihost, …). Uploads only changed files. | `FTP_SERVER`, `FTP_USERNAME`, `FTP_PASSWORD` |
| `sftp` | Hosts with SFTP but no encrypted FTP, e.g. **xneelo**. Password login; uploads new and changed files, never deletes. | `FTP_SERVER`, `FTP_USERNAME`, `FTP_PASSWORD` (+ `FTP_PORT` if not 22) |
| `cpanel-ssh` | cPanel or a VPS with SSH. Faster; also removes deleted files. | `SSH_HOST`, `SSH_USER`, `SSH_KEY` (+ `SSH_PORT` if not 22) |
| `vercel` | Vercel projects (static, Next.js, …) | `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID` |
| `netlify` | Netlify sites | `NETLIFY_AUTH_TOKEN`, `NETLIFY_SITE_ID` |
| `supabase` | Database migrations (`supabase/migrations`) and edge functions (`supabase/functions`) | `SUPABASE_ACCESS_TOKEN`, `SUPABASE_PROJECT_REF`, `SUPABASE_DB_PASSWORD` |

Every project also gets `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` for the result message.

A project with a website on one host and a database on another (e.g. Vercel + Supabase) simply calls the
workflow twice, as two jobs.

## Adding a project

1. Copy `templates/deploy.yml` to `.github/workflows/deploy.yml` in the project and fill in the `with:` lines.
2. In the project on GitHub: **Settings → Secrets and variables → Actions → New repository secret**, add the
   secrets for its target (table above). Passwords live only there, encrypted.
3. Push. The first cPanel deploy uploads everything; after that only changes.

Manual redeploy: the project's **Actions** tab → **Deploy** → **Run workflow**.
Roll back: revert the commit in GitHub (or `git revert`) and push; it redeploys the previous version.

## One-time GitHub setting (private deploy-kit)

If this repo is private, allow your other repos to use it: **deploy-kit → Settings → Actions → General →
Access → "Accessible from repositories owned by the user"**.

## Safety lock

On its first deploy, each project leaves a `.deploy-owner` file in its host folder naming its repo. Every later
deploy (cPanel FTP/SSH) reads that file first: if it names a different repo, for example because one site's
FTP login was pasted into another repo's secrets, the deploy is refused before anything is uploaded and
Telegram gets a 🔒 message. To hand a folder to another repo on purpose, delete `.deploy-owner` on the host.

## cPanel notes

- FTP details: cPanel → **FTP Accounts**. Best practice is a dedicated FTP account per site, limited to that
  site's folder; then set `server_dir: /` in the project.
- The FTP deploy keeps a small `.ftp-deploy-sync-state.json` on the server to know what changed. Leave it there.
- SSH: if cPanel shows **SSH Access** / **Terminal**, generate a key there, authorise it, put the private key in
  `SSH_KEY`, and switch the project to `target: cpanel-ssh`.
