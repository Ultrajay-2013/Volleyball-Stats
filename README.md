# Volleyball Stat Tracker — Requirements

## Project Description

A mobile-first, single-file web app (HTML + CSS + JavaScript, no external dependencies) for tracking a volleyball match live from the sideline. It records every rally outcome for your team, attributes plays to individual players, tracks set and match scores, and produces a full post-game summary. The design is dark and modern, with large tap targets.

## Features

### Game Setup Screen
- Inputs for your team name and the opponent team name.
- Roster entry: jersey number and name for each player on your team.
- Add or remove player rows.
- Validation: both team names are required, at least one named player is required, and jersey numbers can't be duplicated.
- The roster carries over when you start a new game.

### Live Game Screen
- Scoreboard showing the current set, the points in the current set, sets won, and the scores of completed sets.
- **Points Won** buttons: Kill, Ace, Block, Opponent Error.
- **Points Lost** buttons: Attack Error, Service Error, Reception Error, Got Blocked, Opponent Point.
- Each tap updates the score automatically.
- A feed of recent plays.

### Player Attribution
- Flow: you tap the play, then choose the player from a pop-up grid.
- Opponent Error and Opponent Point aren't tied to one of your players, so they skip the picker.

### Set Tracking
- Best of 5 sets.
- Sets 1–4 go to 25 points and set 5 goes to 15, with a 2-point winning margin in every set.
- When a set is won, a confirmation appears: **Start Next Set** or **Undo Last Play**.
- When a team wins 3 sets, a match-won prompt appears: **View Summary** or **Undo Last Play**.

### Undo
- Removes the most recent play, even when that play crossed a set boundary.

### Game Over / Post-Game Summary
- Match result and team score by set.
- Totals for points won and lost.
- Breakdown by category, with a count and a percentage bar for each.
- Individual stats for each player (a count for every category, plus points won, points lost and net).
- You can go back to the game from the summary, or start a new game.

### Persistence
- The full game state is saved to `localStorage` after every change and restored when the page reopens.

## Controls

| Action | Control |
|---|---|
| Record a play | Tap a play button, then tap the player |
| Cancel the player choice | Tap Cancel in the picker |
| Undo | Tap the Undo button in the bottom bar |
| Advance the set | Tap Start Set N in the set-won dialog |
| End the game | Tap Game Over, then confirm |

## Win/Lose Conditions
- **Set win:** reaching 25 points (15 in set 5) with a lead of at least 2.
- **Match win:** the first team to win 3 sets.
- **Manual end:** Game Over can end the match early at any time.

## Tech Stack
- A single `.html` file with inline CSS and vanilla JavaScript (ES2020).
- No libraries, no network requests.
- `localStorage` for persistence.

## Nice-to-Haves (not in this version)
- Export the summary as CSV or image.
- Tracking serve rotation and substitutions.
- Game history across multiple matches.
- Editing the roster mid-game.
