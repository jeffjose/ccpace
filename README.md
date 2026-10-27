# ccpace

Am I over or under on Claude Code this week — and what should I do about it tonight?

```
Claude Code  Wed 8:00 PM · resets Sat 12:00 AM (2d 4h)

  week  █████████████████████████░░░┃░░░░░░░░░░░  62%  plan 70%  bed 76%
        Sat 15/20  Sun 13/20  Mon 14/12  Tue 9/12  Wed 11/12  Thu ·/12  Fri ·/12

  ┌ today ───────────────┐ ┌ speed %/h ────────────────────┐ ┌ time left ───────────┐
  │ 11 of 25%    14 left │ │ last hour 1.7?     target 3.5 │ │ tonight        4h 0m │
  │ █████████░░░░░░┃░░░░ │ │ 15m 1.7? · 6h 1.0    plan 1.6 │ │ session            · │
  │ by now 19    bed 76% │ │ → 95%? at reset               │ │ week           2d 4h │
  └──────────────────────┘ └───────────────────────────────┘ └──────────────────────┘

  ? single tick

 [UNDER 7.6]  Go harder.  2.2× pace · 62% of 70%
  Use 14% by midnight — 3.5%/h, 2.2× the planned pace.
```

The weekly meter only means something next to where it *should* be by now.
ccpace reads your live usage, compares it to a plan of how you intend to spend
the week, and tells you whether to ease off or push. The card reads from the
bottom up: the verdict sits last, nearest the prompt, with three tiles (today,
speed, time left) above it and the week at the top. In a terminal the card is
boxed and coloured; the sample above is `--plain`. `--png` draws a fuller
version as an image, with the week as a chart (see [Unattended runs](#unattended-runs)).

## Install

Needs [uv](https://docs.astral.sh/uv/) and a logged-in Claude Code. uv fetches
the one dependency, Pillow, on the first run; it is only loaded for `--png`.

```sh
git clone https://github.com/jeffjose/ccpace && ln -s "$PWD/ccpace/ccpace" ~/.local/bin/ccpace
```

## Usage

```
ccpace              the card: the week, today, the speed, and what to do now
ccpace -f           skip the 60s cache and re-fetch
ccpace -w 30        redraw every 30s (bare -w: every 60); r re-fetches now, q quits
ccpace -j           print JSON and exit (scripting / statusline)
ccpace --plain      no box and no colour, whatever the terminal
ccpace --png FILE   write the card as an image (- for stdout)
ccpace --strict     exit non-zero, with the reason, rather than show a cached reading
ccpace -c FILE      use a different plan file
ccpace --record     log a reading and exit quietly (for cron)
ccpace --used 70 --now 2026-10-07T14:00    what-if, no network
```

## Reading the card

**week** — the weekly meter. The `┃` struck through the bar is where the plan
has it by now; `bed` is where the plan has it when today ends. The line under
it is each day's use out of its share of the week: amber is a day that went
over, `·` a day still to come.

**today** — what you've used since the day began, out of what it takes to reach
tonight's target from there (so a day started under plan gets more). `by now`
is how much of that the plan expected to be gone already.

**speed %/h** — how fast the meter moved over the last hour, 15 minutes and 6
hours, in points per hour. `target` is the rate that lands tonight's target from
here, and `plan` is the rate the plan itself has for the rest of today: the
gap between `last hour` and `target` is what to change. The arrow is where the
week ends if the rest of it runs at the last hour's multiple of the plan, and
`full` is when the meter would hit 100% at that pace.

**time left** — until the day ends, until the 5-hour session resets (with how
much of it is used), and until the week resets.

`~` marks a figure interpolated across a gap in the history, and `?` a
projection that rests on a single tick of the meter (it moves in whole
points, so one tick is not a rate).

## The plan

With no config, every day gets an equal share of the week, spread flat from
9am to midnight. To change that, copy `config.example.toml` to
`~/.config/ccpace/config.toml`:

```toml
[days]                 # share of the week; only the ratios matter
weekday = 12
weekend = 20

[hours]                # [from_hour, to_hour, fraction] blocks per day
weekday = [[9, 18, 0.2], [18, 24, 0.8]]
weekend = [[9, 24, 1.0]]
```

Keys are `mon`…`sun` or the groups `weekday` / `weekend` / `all`. The last
block of a day is the "prime" block the advice is phrased around.

If you work past midnight, set `day_starts` so a late night counts toward the
evening it began in, and let a block run past midnight:

```toml
day_starts = 2         # Tuesday runs Tue 2am -> Wed 2am

[hours]
weekday = [[2, 8, 0.10], [8, 18, 0.15], [18, 2, 0.75]]
```

The plan file is, in order: `-c FILE`, the `CCPACE_CONFIG` environment
variable, then `~/.config/ccpace/config.toml`.

### Timezone

The plan's hours only mean something in one zone. By default that is the
machine's own; to pin it, add a top-level key:

```toml
timezone = "America/Los_Angeles"
```

With it set, "now", the reset times from the API, the day rollover, the plan
blocks and every time on the card are on that zone's clock, wherever ccpace
runs — a server on UTC shows the same card as your laptop. `--now` is read on
that clock too (give it a UTC offset to name an exact instant instead). In
`-j`, every time is that zone's wall-clock except `fetched_at`, which carries
its offset. Zone names come from the system's tz database.

## History

The API only reports the current meter, so ccpace logs every reading it
fetches to `~/.local/state/ccpace/history.jsonl` (about two weeks are kept).
The start of today, the speed rows and the projections are all read back from
it. Where readings are sparse the card says so: the today row notes
`since Mon 8:00 PM`, and interpolated figures carry a `~`. For exact figures,
record on a schedule — one machine is enough, the meter is account-wide:

```
*/10 * * * * ~/.local/bin/ccpace --record
```

## How it gets the numbers

It calls the same usage endpoint Claude Code's `/usage` screen uses, with the
OAuth token Claude Code keeps in `~/.claude/.credentials.json`. The token is
read, sent only to `api.anthropic.com`, and never written, logged, or
refreshed. The endpoint is undocumented and may change. The last response is
cached in `~/.cache/ccpace/` for 60 seconds, and after an HTTP 429 ccpace
leaves the endpoint alone for as long as it asks (three minutes if it doesn't
say), `-f` included.

## Unattended runs

Because the token is never refreshed here, it only stays valid while Claude
Code itself is being used on that machine. On a box where Claude Code is not
used interactively, expect the fetch to start failing as `expired` within
hours of logging in. Plan for that rather than treating it as an outage.

When a fetch fails and a cached reading exists, ccpace shows the cached one,
adds a `!` note to the card, and exits 0. Two ways to see that from a script:

`-j` reports it in three fields:

```json
"fetch": "ok",
"fetched_at": "2026-10-09T22:14:01-07:00",
"stale": false
```

| `fetch`        | meaning                                                  |
|----------------|----------------------------------------------------------|
| `ok`           | fetched now, or a cached reading under 60 seconds old    |
| `expired`      | the API rejected the token (HTTP 401)                    |
| `no_login`     | no credentials file, or no token in it                   |
| `rate_limited` | HTTP 429, or still inside the back-off after one         |
| `unreachable`  | anything else: network, timeout, an unexpected response  |

`fetched_at` is when the reading shown was fetched, and `stale` is true
whenever `fetch` is not `ok`. In a full what-if (`--used` with `--now`)
nothing is fetched and `fetch` and `fetched_at` are null.

`--strict` refuses the fallback: the reason goes to stderr, nothing goes to
stdout, and the exit code says which:

| exit | reason                          |
|------|---------------------------------|
| 2    | token expired                   |
| 3    | no login                        |
| 4    | unreachable, or rate limited    |

A cached reading under 60 seconds old still counts as fresh; add `-f` to
insist on a request. Without `--strict` and with no cached reading at all,
ccpace exits 1 as before.

`--plain` prints the card with no box and no colour regardless of the
terminal, which is the form to paste or post.

`--png FILE` writes the card as an image instead, 1800 pixels wide, laid out
to be read shrunk in a chat feed: the verdict in large type with where the
meter stands beside it, the three tiles, and the week as a chart of the meter
against the plan (with the catch-up line to tonight's target and where the
current pace ends the week) over one bar per day. Nothing is printed. It
combines with `--strict`, in which case a failed fetch exits before any file
is written. The type is [Geist](https://vercel.com/font), fetched once from
jsDelivr into `~/.cache/ccpace/fonts/`; if that can't be reached, DejaVu Sans
stands in. To post it to a Discord webhook:

```sh
ccpace --strict --png card.png && curl -F "file1=@card.png" "$DISCORD_WEBHOOK_URL"
```
