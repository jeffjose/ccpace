# ccpace

Am I over or under on Claude Code this week — and what should I do about it tonight?

```
Claude Code  Tue 5:58 PM  · week resets Sat 12:00 AM (3d 6h)

  weekly   ████████████████████░░░░░░░░░░░░░░░░░░░░  51%  [UNDER by 3.4%]  on pace   over
                                 ▲ expected by now: 54%
  today    █████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  4 of 17%  · 13% left to tonight's 64%
                                        ▲ expected by now
  session  ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  5%  · 5h window resets 6:40 PM (42m)

  Quiet so far — you can go 1.4x tonight (13% vs the usual 10%).

           Sat  Sun  Mon  Tue  Wed  Thu  Fri
  target    20   40   52   64   76   88  100
  actual    18   41   47   51    ·    ·    ·
```

The weekly meter only means something next to where it *should* be by now.
ccpace reads your live usage, compares it to a plan of how you intend to spend
the week, and tells you whether to ease off or push.

## Install

Needs [uv](https://docs.astral.sh/uv/) and a logged-in Claude Code. No dependencies.

```sh
git clone https://github.com/jeffjose/ccpace && ln -s "$PWD/ccpace/ccpace" ~/.local/bin/ccpace
```

## Usage

```
ccpace              the meter, the plan, and the advice
ccpace -f           skip the 60s cache and re-fetch
ccpace -j           print JSON and exit (scripting / statusline)
ccpace -c FILE      use a different plan file
ccpace --record     log a reading and exit quietly (for cron)
ccpace --used 70 --now 2026-10-07T14:00    what-if, no network
```

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

The table at the bottom shows where the meter should close each day and where
it actually did (`?` when no reading was logged that day).

## The today bar

`4 of 17%` means 4% of the weekly meter used since the day began, out of the
17% it takes to get from there to tonight's bedtime target. Start a day under
plan and the allowance grows; start over and it shrinks.

The API only reports the current meter, so ccpace logs every reading it fetches
to `~/.local/state/ccpace/history.jsonl` and takes the last one before midnight
as the day's starting point. If that reading is old, the bar says so
(`since Mon 8:00 PM`); with no history yet it falls back to the plan. For an
exact figure every day, record on a schedule — one machine is enough, the meter
is account-wide:

```
*/30 * * * * ~/.local/bin/ccpace --record
```

## How it gets the numbers

It calls the same usage endpoint Claude Code's `/usage` screen uses, with the
OAuth token Claude Code keeps in `~/.claude/.credentials.json`. The token is
read, sent only to `api.anthropic.com`, and never written, logged, or
refreshed. The endpoint is undocumented and may change. The last response is
cached in `~/.cache/ccpace/` for 60 seconds.
