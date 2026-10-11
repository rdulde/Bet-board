# Bet-board

A one-page NFL bet tracker that pulls live scores from ESPN and shows each ticket's chance to cash as the games run.

**The board:** https://rdulde.github.io/Bet-board/

That link is the current version on every device. Bookmark it on the phone and the iPad. Nothing else needs to be installed or synced.

## How the week works

1. Paste a screenshot of the open bets into Claude.
2. Claude turns it into `bets.json` (format below, no ticket numbers).
3. Put `bets.json` in this repo (see *Updating the repo*).
4. Open the board. If it has no bets it loads them on its own. If it still shows last week, a gold banner says **New bets are posted** with a **Load them** button. The **Load or change bets** panel also has a **Load this week's bets** button for a manual reload.

The board checks `bets.json` when it opens and every 10 minutes after that. It compares the file's contents with what it has already applied, so a changed file shows up as new and an unchanged file is ignored.

GitHub Pages takes about a minute to publish a commit. If the banner does not appear right away, give it a minute and refresh.

## Files

| File | What it is |
| --- | --- |
| `index.html` | The whole app. Edit it to change the board. |
| `bets.json` | This week's tickets. Replaced every week. |
| `bets.example.json` | Format reference with fake bets. Not loaded by the board. |

## `bets.json` format

```json
{
  "week": "2026-W06",
  "label": "NFL Week 6",
  "updated": "2026-10-11T18:00:00Z",
  "tickets": [
    {
      "id": "1",
      "kind": "Parlay · 2 legs",
      "stake": 25,
      "win": 90.91,
      "legs": [
        { "type": "spread", "team": "KC",  "line": -3.5, "odds": -110 },
        { "type": "over",   "gameTeam": "BUF", "line": 44.5, "odds": -105, "label": "Buffalo Bills vs Miami Dolphins" }
      ]
    }
  ]
}
```

- `week` is a sortable key, `label` is what the header shows. Both are optional text.
- `id` is any short string. Use `1`, `2`, `3` rather than the sportsbook ticket number, because the repo is public.
- `kind` is display text: `Straight`, `Parlay · 3 legs`, and so on. Left out, the board fills it in from the leg count.
- `stake` is the risk, `win` is the profit if it cashes. Both are plain numbers.
- Leg `type` is one of `ml`, `spread`, `under`, `over`.
  - `ml` needs `team` and `odds`.
  - `spread` needs `team`, `line` (negative when laying points) and `odds`.
  - `under` and `over` need `gameTeam` (either team in that game), `line` and `odds`. `label` is optional text shown next to the line.
- Teams use the ESPN abbreviations: ARI ATL BAL BUF CAR CHI CIN CLE DAL DEN DET GB HOU IND JAX KC LV LAC LAR MIA MIN NE NO NYG NYJ PHI PIT SF SEA TB TEN WSH. Full team names and the common aliases (WAS, JAC, LA) are accepted too.
- `odds` are American. They seed each leg's pregame win chance, so leaving them out makes the pregame numbers worse but the board still runs.

The board also accepts the older format, a bare array of tickets, which is what **Export JSON** used to produce.

## Updating the repo

No git needed. On GitHub:

1. Open https://github.com/rdulde/Bet-board/upload/main
2. Drag `bets.json` onto the page (or `index.html` too if the board changed).
3. Click **Commit changes**.

Uploading a file with the same name replaces the old one.

## Privacy

The repo is public, so `bets.json` is readable by anyone with the link. It carries picks, stakes and odds but no ticket numbers, account details or sportsbook name. **Export JSON** on the board writes the same shape and renumbers the tickets `1..n` for the same reason. Making the repo private would hide the file but GitHub Pages on a private repo needs a paid plan.

Scores, the trend chart and the applied bets live in the browser's local storage on each device. Nothing is sent anywhere except the request to ESPN's public scoreboard and the fetch of `bets.json`.

## Working on the board locally

`index.html` fetches `bets.json` with a relative URL, so opening the file straight from disk cannot load bets (browsers block fetch from `file://`). Serve the folder instead, for example:

```bash
python -m http.server 8080
```

then open http://localhost:8080/. The screenshot reader, paste parser and manual score entry all work either way.

## How the win chance is modelled

Each leg starts at the fair probability its own odds imply (vig removed). As the game runs, the remaining margin is treated as normal with SD 13.45 × √(minutes left ÷ 60), centred on a projection that blends the pregame expectation (fading as time runs off) with 30% of the scoring rate shown so far. Totals project the rest of the game from the pregame expected total, shifting toward the actual pace. A ticket's chance is the product of its legs. These are modelled numbers, not cash-out prices.
