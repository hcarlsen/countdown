# countdown

Count-down to a specific time. A single self-contained `index.html` — no build step,
no server. Open the file and it runs.

## Usage

Open `index.html` in a browser. Set an optional title, then pick either:

- **Dato & tid** — an absolute date and time, or
- **Om X tid** — a duration in hours/minutes, converted to an absolute time on submit.

The countdown shows days/hours/minutes/seconds, the end time, and a fullscreen button.
"Nulstil eller skift tidspunkt" clears the counter and returns to setup.

## Persistence

A running countdown is stored in the URL hash (`#t=<epochMs>&n=<title>`), so it survives
reload, hibernate, and a browser restart that restores the tab. The target is stored as an
**absolute** timestamp, so time passing while the machine is off or asleep doesn't matter —
on load the page reads the target, subtracts the current time, and renders the remainder.
An expired countdown resumes as "Tiden er udløbet" rather than dropping back to setup.

The hash is per-tab, so **multiple countdowns run in multiple tabs without interfering** —
e.g. one tab counting to a weekly deadline, another to a daily one. The tab title updates
each second (`Claude weekly · 01:59:59`) so tabs stay distinguishable from the tab strip.

`sessionStorage` mirrors the state as a backup in case the hash is stripped, but it does not
survive a browser quit — the hash is what makes a restart work.

### Limitations

- **It does not ring.** There is no sound and no notification. When the time runs out the
  page shows "Tiden er udløbet", which you only see if you look at the tab. If the machine
  is off at the target time, nothing happens at that time — you just see the expired state
  next time you open the page.
- **Resuming depends on the tab coming back.** After a restart it works if the browser is set
  to reopen your tabs. Close the tab outright, or open the file fresh, and there is no hash
  and nothing to resume. Bookmark the URL while a counter is running to keep it.

## Development notes

- State lives in the URL **hash**, not a query string: `history.replaceState` with a query
  string throws `SecurityError` on `file://` (origin `null`). The hash works on both
  `file://` and http, so there is no fallback path. Don't convert it back to a query string.
- **Don't use `localStorage`.** It is shared per origin, so two tabs would overwrite each
  other's countdown. Per-tab storage is the whole point.
