# Phone reminder setup (ntfy.sh + GitHub Actions)

This gives you a real push notification on your phone between 6-8pm on days you haven't
practiced (or haven't finished today's reviews) — even if the Claude tab is closed.

No servers, no accounts to pay for. About 10 minutes, one-time.

## 1. Pick a topic name

Pick something private and hard to guess, e.g. `jh-english-notes-7q2x9`.
Anyone who knows this exact string can read/post to it, so don't reuse a name you use
publicly anywhere else, and don't put anything sensitive in the messages.

## 2. Install ntfy on your phone

- iOS: search "ntfy" on the App Store
- Android: search "ntfy" on Google Play (or F-Droid)

Open the app → Subscribe to topic → enter `<your-topic>-alerts` (e.g.
`jh-english-notes-7q2x9-alerts`). That's it — no account, no login.

## 3. Update the artifact

In `phrase_field_notes.jsx`, find this near the top:

```js
const NTFY_BASE_TOPIC = "CHANGE-ME-your-own-topic-9f3k2";
```

Replace the placeholder with your topic name (e.g. `"jh-english-notes-7q2x9"`), then
re-upload/re-save the artifact in Claude. Once this is set, the "Phone reminders aren't
set up yet" hint in the header disappears — that's your confirmation it's live.

## 4. Create a GitHub repo for the reminder job

1. Create a new repository on GitHub (public is simplest — unlimited free Actions
   minutes; private also works, you just get 2,000 free minutes/month, which is far
   more than this needs).
2. Add the file `study-reminder.yml` (provided alongside this guide) at exactly this
   path: `.github/workflows/study-reminder.yml`.
3. Go to the repo's **Settings → Secrets and variables → Actions → New repository
   secret**. Name it `NTFY_TOPIC_BASE`, value = your topic name from step 1 (just the
   base, e.g. `jh-english-notes-7q2x9`, without `-status` or `-alerts`).

## 5. Test it

Go to the repo's **Actions** tab → "Study Reminder" workflow → **Run workflow** (this is
the `workflow_dispatch` trigger, lets you fire it manually instead of waiting for the
schedule). Check the run log:
- If it says "Not nudge time, skipping" — that's correct outside 6-8pm Auckland time; it
  worked, it just didn't need to nudge you right now.
- To force a real test, temporarily edit the `HOUR` check in the workflow file to match
  whatever hour it is right now, run it once, then change it back.

## Known limitations (being upfront about these)

- **GitHub's scheduler isn't exact.** It can run a bit late (occasionally 10-30+ minutes
  during high load). The workflow checks a 2-hour window (18:00-19:59 Auckland), not an
  exact minute, specifically to absorb this.
- **60-day auto-disable.** If a repo gets zero commits/issues/PRs for 60 straight days,
  GitHub silently turns off its scheduled workflows (you'd get one email about it, easy
  to miss). If reminders quietly stop, this is the most likely reason — a small commit
  resets the clock.
- **ntfy.sh topic names are "secret by obscurity," not real security.** Fine for this
  use case, but don't treat it as private in any serious sense.
- **This only tells you *whether* you're behind, using the due-count snapshot the
  artifact last sent** — it can't see the artifact's live state at the exact moment of
  the check, just whatever was last reported when you had it open.
