# ccpace

Am I over or under on Claude Code this week — and what should I do about it tonight?

```
Claude Code  Tue 5:58 PM  · week resets Sat 12:00 AM (3d 6h)

  weekly   ████████████████████░░░░░░░░░░░░░░░░░░░░  51%  [UNDER]  on pace   over  by 3.4%
                                 ▲ plan 54%
  today    ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  0 of 12%  · 13% left to tonight's 64% (1% banked)
                   ▲ plan
  session  ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  5%  · 5h window resets 6:40 PM (42m)

  Quiet so far — you can go 1.4x tonight (13% vs the usual 10%).

  bedtime  Sat 20 · Sun 40 · Mon 52 · [Tue 64] · Wed 76 · Thu 88 · Fri 100
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

## How it gets the numbers

It calls the same usage endpoint Claude Code's `/usage` screen uses, with the
OAuth token Claude Code keeps in `~/.claude/.credentials.json`. The token is
read, sent only to `api.anthropic.com`, and never written, logged, or
refreshed. The endpoint is undocumented and may change. The last response is
cached in `~/.cache/ccpace/` for 60 seconds.
