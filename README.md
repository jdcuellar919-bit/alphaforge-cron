# alphaforge-cron

Scheduled jobs for [AlphaForge](https://www.alphaforgeresearch.com): the shared market-cache refresh and the keep-warm pinger.

**Why a separate public repo:** the app repo went private on 2026-09-06. GitHub meters Actions minutes on private repos (2,000/month free) and these jobs need ~2,700/month — when the quota runs out *every* workflow stops (stale data + a cold backend). Public repos get unlimited minutes, and minutes bill to the repo the workflow runs in. The application code is **not** copied here: `refresh-cache.yml` checks out the private repo read-only.

## One-time setup (owner)
1. Create a fine-grained PAT: GitHub → Settings → Developer settings → Fine-grained tokens → **Repository access: only `alphaforgeresearch`** → Permissions: **Contents: Read-only**. Copy it.
2. Add four secrets to this repo (Settings → Secrets and variables → Actions → New repository secret), or from Terminal (each prompts for the value, nothing echoes):
   ```
   gh secret set MAIN_REPO_TOKEN -R jdcuellar919-bit/alphaforge-cron
   gh secret set POLYGON_API_KEY -R jdcuellar919-bit/alphaforge-cron
   gh secret set SUPABASE_URL -R jdcuellar919-bit/alphaforge-cron
   gh secret set SUPABASE_SERVICE_KEY -R jdcuellar919-bit/alphaforge-cron
   ```
3. Run **Refresh market cache** once by hand (Actions tab → Run workflow, sections `movers growth`). Green = the checkout + secrets work.
4. Then delete the `schedule:` blocks from the main repo's two workflows so nothing double-runs (keep `workflow_dispatch`).

## Schedules
| Job | Cron (UTC) | Sections |
|---|---|---|
| movers + growth | `5 13-21 * * 1-5` | hourly, US market hours |
| calendar + Daily Brief | `15 12 * * 1-5` | pre-market |
| fundamentals pack | `0 9 * * *` | calendar nextearn bthistory fundamentals smartmoney (~40 min) |
| Prime whole-market scan | `30 21 * * 2-6` | swingmarket (~35 min) |
| keep-warm | `*/7`-ish ×3 | /ping + radar cache warmers |
