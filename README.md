# Volleyball Stat Tracker — Requirements

## Project Description

A mobile-first, single-file web app (HTML + CSS + JavaScript, no external libraries) for tracking a volleyball match live from the sideline. It records every rally outcome for your team, attributes plays to individual players, tracks set and match scores, and produces a full post-game summary. Past games are kept in a history, and optional AI insights come from Groq. The design is dark and modern, with large tap targets.

## Branding
- The app logo (a blue volleyball with green and red stat bars on a dark slate tile) is embedded in the file as data URIs, so the app stays a single file.
- **Header:** a 30px logo next to "Volleyball Stats" on every screen.
- **Browser tab icon:** a 64×64 PNG favicon.
- **Home-screen icon:** a 192×192 icon for Add to Home Screen on Android and iPhone (`apple-touch-icon`), with "VB Stats" as the iPhone home-screen name.

## Features

### Game Setup Screen
- Inputs for your team name and the opponent team name.
- Roster entry: jersey number and name for each player on your team.
- **Saved rosters:** **Save Roster** stores your team name and players on the device under a name you choose; reusing a name replaces that roster. **Load Roster** fills in the team name and players, and × deletes a saved roster after a confirmation.
- Add or remove player rows.
- Validation: both team names are required, at least one named player is required, and jersey numbers can't be duplicated.
- The roster carries over when you start a new game.

### Google Sign-In & Sync
- **Optional:** the app works fully signed out and offline, saving on the device as before.
- **Where:** an account box at the top of the setup screen shows **Sign in with Google**. Once signed in, it shows the user's name, email and photo, the sync status, and **Sync Now** / **Reconnect** and **Sign Out** buttons.
- **Sign-in method:** Google Identity Services token flow (`google.accounts.oauth2.initTokenClient`). Scopes are `openid email profile` and `https://www.googleapis.com/auth/drive.appdata`, which are all non-sensitive.
- **Storage:** one JSON file (`volleyball-stats-sync.json`) in the user's private Drive **appDataFolder**, holding the current game, game history, saved rosters and lists of deleted games and rosters.
- **Per-account data on the device:** signed-in data is cached under keys tied to the Google account. Signing out removes that account's cache from the device, and signed-out data is kept separate.
- **First sign-in on a device:** if the device already has signed-out data, the app asks to **Add to Account** (merge and upload) or **Keep Separate**.
- **Syncing:**
  - It runs about 1.5 seconds after any change, when the app comes back to the foreground, when the connection returns, and every 60 seconds while the app is open.
  - Game history and rosters merge item by item, and deletions win. Rosters with the same name keep the newest copy.
  - The current game keeps whichever device's version was edited most recently. Screen and navigation stay per-device, but if the other device starts, ends or resumes the game, this device follows.
  - Edits made while a sync is running are never overwritten, because that round is skipped and repeated.
- **Status:** shown in the account box, and as a small top-bar pill on other screens (Synced, Saving, Syncing, Offline, Reconnect). Google tokens last about an hour, and **Reconnect** renews one with a single tap.
- **Not synced:** the Groq API key stays on the device.
- **Requirements:** a Google Cloud OAuth Web client ID in `GOOGLE_CLIENT_ID`, the Drive API enabled, and the app hosted over HTTPS (GitHub Pages). See `google-sync-setup.md`. Without a client ID, or when opened as a local file, the sign-in button is disabled with an explanation.

### Simulate Game (testing)
- A **Simulate Game** button sits below Start Game on the setup screen.
- It plays a full random best-of-3 match. Each rally has 0–3 plays with no point (touch stats, or optional plays without End of Rally), then one random play that scores, and about 20% of player plays credit two players. Plays are random, so some sequences won't make volleyball sense.
- It uses the team names and players on the setup screen, or "Sim Team", "Opponents" and six sample players if those are empty.
- It opens a summary labeled "Simulated game · random plays · not saved", where AI insights can be generated. **Simulate Again** makes a new match and **Back to Setup** returns to setup.
- Simulated games and their insights are never saved to Game History or storage. Reloading the page goes back to setup.

### How to Use Page
- A **How to Use** button in the top-right corner of the setup screen opens the guide. It only appears on the setup screen.
- Sections: Quick start, Recording plays (including selecting players, End of Rally and Undo), Which plays score, Sets & scoring, After the game, Saved rosters, AI insights (with how to get a Groq key) and Your data.
- The "Which plays score" list is generated from the play definitions, so it updates itself when plays change.
- **Back** returns to setup, and anything typed on the setup screen is kept.

### Live Game Screen
- Scoreboard showing the current set, the points in the current set, sets won, and the scores of completed sets.
- **Good Plays** buttons (formerly "Point Won"): Attack, Tip, Setter Tip, Serve, Block, Free Ball, Receive, Serve Receive, Dig, Set, Opponent Error. (Overpass was removed, but older games that used it still show it.)
- **Bad Plays** buttons (formerly "Point Lost"): Bad Attack, Bad Serve, Bad Receive, Bad Serve Receive, Bad Set, Got Blocked, Miscommunication, Opponent Point.
- **Buttons are grouped by action**, in rally order, each group under a small heading. Colors are unchanged: green for good plays, red for bad plays.
  - Serve: Serve, Bad Serve
  - Serve Receive: Serve Receive, Bad Serve Receive
  - Receive: Receive, Dig, Bad Receive
  - Set: Set, Setter Tip, Bad Set
  - Attack: Attack, Tip, Bad Attack
  - Block: Block, Got Blocked
  - Free Ball and Miscommunication: one button each, side by side
  - Opponent: Opponent Error, Opponent Point
- A play that isn't listed in a group automatically appears under "Other", so new plays can't go missing.
- Renamed from earlier versions: Kill → Attack, Ace → Serve, Attack Error → Bad Attack, Service Error → Bad Serve, Reception Error → Bad Receive, Serve Receive Error → Bad Serve Receive. Saved games show the new names.
- **Receive**, **Serve Receive**, **Dig** and **Set** are touch stats. They're credited to players (multi-select works) but never score, and the picker has no End of Rally button for them.
- **End of Rally:** Attack, Tip, Setter Tip, Serve, Block, Free Ball, Bad Attack, Bad Receive, Bad Serve Receive, Bad Set and Got Blocked are recorded as stats only, with no point, unless **End of Rally** is turned on in the picker. It sits between Cancel and Done, starts off every time the picker opens, and a hint shows who gets the point.
- Bad Serve, Miscommunication, Opponent Error and Opponent Point always score.
- A feed of recent plays.

### Player Attribution
- Flow: you tap the play, then choose the player from a pop-up grid.
- **Player picker:** it's always in multi-select mode. Tap one or more players (tap again to unselect), then tap **Done (n)**. Done stays disabled until at least one player is picked. The buttons are Cancel, End of Rally (only on optional plays) and Done.
- A shared play adds 1 point to the score. Each chosen player gets full credit in that stat and in +Pts/−Pts, while the Team row counts the play once.
- Opponent Error and Opponent Point aren't tied to one of your players, so they skip the picker.

### Set Tracking
- Best of 3 sets.
- Sets 1–2 go to 25 points and set 3 goes to 15, with a 2-point winning margin in every set.
- When a set is won, a confirmation appears: **Start Next Set** or **Undo Last Play**.
- When a team wins 2 sets, a match-won prompt appears: **View Summary** or **Undo Last Play**.

### Undo
- Removes the most recent play, even when that play crossed a set boundary.

### Game Over / Post-Game Summary
- Match result and team score by set.
- Totals for points won and lost.
- Breakdown by category, with a count and a percentage bar for each. Only rally-ending plays count here, and each category also shows how many of its plays had no point. Receive, Serve Receive, Dig and Set aren't shown here because they can't score.
- Player stat columns count every play. +Pts/−Pts, Net, totals and win rate only count plays that ended the rally.
- **Player stats by category:** there's one table per action category, matching the game-screen groups (Serve, Serve Receive, Receive, Set, Attack, Block, Free Ball, Miscommunication). Rows are players plus a Team row.
  - Columns: **Good pts**, **Good no pt**, **Bad pts**, **Bad no pt**, **Net pts** (Good pts − Bad pts) and **Net all** (every good play − every bad play).
  - A column is left out when no play in that category can produce it. For example, Receive has no "Good pts" because Receive and Dig never score, and Serve has no "Bad no pt" because Bad Serve always scores.
  - Shared plays count for every credited player, but only once in the Team row. Opponent Error and Opponent Point have no table because they aren't tied to a player.
- You can go back to the game from the summary, or start a new game.

### Game History
- A **Game History** button sits below Start Game on the setup screen and shows how many games are saved.
- A game is archived when **New Game** is tapped, as long as it has at least one play.
- The list shows each game's matchup, date/time, and a result badge (W, L or Unfinished), newest first.
- Tap a game to open its full summary, AI insights included.
- **Delete Game** removes a game permanently, after a confirmation.
- **Select** (bottom right of the history list) turns on select mode. Tap games to check them, or use **Select All** / **Clear All** at the top, then tap **Delete (n)** and confirm. **Cancel** leaves select mode, and select mode turns off whenever you leave the list.

### AI Insights (Groq)
- An AI Insights card appears on every summary, for the current game and for past games.
- It gives 3 insights: a team strength, an individual player weakness, and one more (a strength, weakness or tip). Each one cites numbers from the game.
- It uses the Groq API free tier (`openai/gpt-oss-120b`, falling back to `openai/gpt-oss-20b` if that model is rate-limited or unavailable), with strict JSON-schema output.
- You enter your Groq API key (from console.groq.com/keys) once, in the app; it's stored only on the device. The **API key** link changes or removes it.
- The AI also gets the plays that didn't end the rally (team-wide and per player) and is told to look for patterns in them.
- Insights are saved with the game. They're cleared if the game is edited afterwards, and **Regenerate** gets a fresh set.
- Clear error messages cover no internet, a bad key (which reopens the key prompt), rate limits and timeouts.
- Team names, player names and stats are sent to Groq only when **Generate Insights** is tapped.

### Persistence
- The full game state is saved to `localStorage` after every change and restored when the page reopens.
- Game history, saved rosters and the API key are also saved in `localStorage`.
- When signed in, the current game, history and rosters also sync to the user's Google Drive app folder.

## Controls

| Action | Control |
|---|---|
| Record a play | Tap a play button, tap the player(s), then tap Done |
| Award the point on a rally play | Tap End of Rally in the picker, then pick the player(s) |
| Test AI insights | Tap Simulate Game on the setup screen, then Generate Insights |
| Delete several games | Game History, then Select, tap the games, then Delete |
| Sign in / sync | Setup screen, then Sign in with Google (tap the top pill to Reconnect) |
| Open the guide | Tap How to Use (top right of the setup screen) |
| Save or load a roster | Tap Save Roster or Load Roster on the setup screen |
| Credit one or more players | Tap the players in the picker, then tap Done |
| Cancel the player choice | Tap Cancel in the picker |
| Undo | Tap the Undo button in the bottom bar |
| Advance the set | Tap Start Set N in the set-won dialog |
| End the game | Tap Game Over, then confirm |
| View past games | Setup screen, then Game History, then tap a game |
| Get AI insights | Tap Generate Insights on any summary |

## Win/Lose Conditions
- **Set win:** reaching 25 points (15 in set 3) with a lead of at least 2.
- **Match win:** the first team to win 2 sets.
- **Manual end:** Game Over can end the match early at any time.

## Tech Stack
- A single `.html` file with inline CSS and vanilla JavaScript (ES2020).
- No bundled libraries. The optional network requests are Groq's Chat Completions API (AI insights), and, when Google sign-in is configured, Google Identity Services (`accounts.google.com/gsi/client`), the userinfo endpoint and the Drive v3 REST API.
- Hosted on GitHub Pages (HTTPS), which Google sign-in requires.
- `localStorage` for persistence.

## Nice-to-Haves (not in this version)
- Export the summary as CSV or image.
- Tracking serve rotation and substitutions.
- Season stats combined across games in history.
- Editing the roster mid-game.
