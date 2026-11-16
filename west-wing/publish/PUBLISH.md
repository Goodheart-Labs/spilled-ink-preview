# Publish the West Wing page on spilled.ink

For a Claude session running on Nathan's Mac, in `~/Documents/Source/agreed-facts`. Nathan approved publishing this page as it is on 2026-11-16. Everything you need is at the URLs below; nothing here needs a secret that is not already in that repo's `.env`.

Base URL: `https://goodheart-labs.github.io/spilled-ink-preview/west-wing/publish/`

| File | What it is |
|---|---|
| `west-wing.html` | The page, one self-contained file. On spilled.ink it reads topic `west-wing` from `facts` and writes ratings to `fact_votes`, the same calls as the other pages. |
| `seed.sql` | Idempotent migration: one `topics` row (slug `west-wing`) and 86 `facts` rows, `suggested_by = 'seed'`. |
| `facts.json` | The same 86 rows as JSON (`topic`, `text`, `sources`, `tags`, `suggested_by`), if you seed over REST with the service-role key as this repo has done before. |

## Steps

1. `git status` first. This repo deploys from the working tree, so note any uncommitted work that is not yours and do not ship changes you did not make without telling Nathan.
2. Download the three files. Put `west-wing.html` in the repo root. Check how `/us-china-ai` is routed (`vercel.json`, clean URLs) and make `/west-wing` serve this file the same way.
3. Seed the database once, by whichever route this repo currently uses for migrations (its notes say they are applied by hand, and that `supabase db push` was blocked by unrecorded migrations). Save `seed.sql` under `supabase/migrations/` for the record. Confirm: `GET /rest/v1/facts?topic=eq.west-wing&select=id` returns 86 rows. Do not run the seed twice after editing any text: it would add the new wording beside the old.
4. Check the page before it is live: deploy a Vercel preview (not `--prod`), open `/west-wing?live` there, and confirm 86 cards under seven character headings. Click "Good call" on one decision and confirm a row for it appears in `fact_votes` and the button's count goes up after a reload. This write path has only been tested against a stand-in for the database, so this step is the real test. If the page was served from a host other than spilled.ink without `?live`, it shows seed data from the file and saves nothing; that is the preview mode, not a bug.
5. `vercel --prod`. Then check `https://spilled.ink/west-wing` returns 200, shows 86 cards and no "Preview: not published" note, and that a rating saves.
6. Commit the page and the migration. Tell Nathan the live URL.

## Things to know

- The rating buttons say Good call, Mixed and Bad call and store `helpful`, `somewhat` and `not_helpful`.
- Tags used: character (`bartlet`, `leo`, `josh`, `toby`, `cj`, `sam`, `hoynes`, `abbey`), air date, `season-1`/`season-2`, `nathan-bad`, and `why:<reason>`. On the standard record view these show as plain tags.
- The page has no AI summary, chat, add-a-statement form or edit button, and no analytics snippet. If the site's pages normally carry PostHog, add the same snippet the other generated pages use.
- The page was written without `tools/gen_stream.py`. Do not regenerate it with that tool unless Nathan asks; it would change the layout he approved.
- The topic row is created visible. If Nathan wants it off the home page's topic list until he posts about it, set `hidden = true` as `us-china-ai` has.
- Nothing after season 2 episode 17 goes on the page, and no verdicts other than Nathan's.

If a step fails, stop and tell Nathan what failed; do not work around it by writing to the live tables some other way.
