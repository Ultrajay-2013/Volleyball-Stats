# Google Sign-In & Sync — One-Time Setup

Signing in with Google lets the app save each coach's current game, game history and saved rosters to their own Google account. That data then syncs to any phone or computer where they sign in.

You only do this setup once. After that, anyone using your app just taps **Sign in with Google**.

## How it works

- **Storage:** data is saved in a hidden, private app folder in each user's own Google Drive (the Drive "appDataFolder"). Only this app can read it, it doesn't appear in the user's Drive file list, and there's no server or database for you to run.
- **Permissions:** the app asks for basic profile info (name, email, photo) and access to that private app folder only. It can't see any of the user's other Drive files.
- **Offline use:** the app still works signed out and offline. When signed in, changes are saved on the phone right away and sync whenever there's internet.
- **Not synced:** the Groq API key stays on each device and is never synced.

## What you need

- A free Google account, for the Google Cloud setup.
- A free GitHub account, to host the app. Google sign-in only works from a real web address, not from a file opened directly on the phone.

## Step 1: Put the app on GitHub Pages

1. Sign in at [github.com](https://github.com) and create a new **public** repository, for example `volleyball-stats`.
2. Rename `volleyball-stats.html` to **`index.html`**.
3. In the repository, choose **Add file → Upload files**, upload `index.html`, then commit.
4. Go to **Settings → Pages**. Under "Build and deployment", set the source to **Deploy from a branch**, choose branch **main** and folder **/ (root)**, then click **Save**.
5. After a few minutes your app is live at
   `https://YOUR-GITHUB-USERNAME.github.io/volleyball-stats/`

Write down the **origin**, which is just the start of that address:
`https://YOUR-GITHUB-USERNAME.github.io`
Leave off the `/volleyball-stats/` part.

## Step 2: Create a Google Cloud project

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and sign in.
2. Use the project picker at the top to create a **New Project** (for example "Volleyball Stats"), then select it. No billing is needed.

## Step 3: Turn on the Google Drive API

1. Go to **APIs & Services → Library**.
2. Search for **Google Drive API**, open it, and click **Enable**.

Sync will fail if you skip this step.

## Step 4: Set up the sign-in screen (Google Auth Platform)

1. Go to **APIs & Services → OAuth consent screen**, which opens **Google Auth Platform**, and click **Get started**.
2. **App information:** enter the app name (for example "Volleyball Stats") and your support email.
3. **Audience:** choose **External**.
4. **Contact information:** enter your email, accept the policy, and click **Create**.
5. Open **Data Access → Add or remove scopes** and add these, then click **Update** and **Save**:
   - `openid`
   - `.../auth/userinfo.email`
   - `.../auth/userinfo.profile`
   - `.../auth/drive.appdata` (search "drive.appdata" and pick "See, create, and delete its own configuration data in your Google Drive")
6. Open **Audience**. While the app is in **Testing**, only the emails you add under **Test users** can sign in. You have two choices:
   - **Just you or a few coaches:** add their Gmail addresses as test users.
   - **Anyone:** click **Publish app** to move it to **In production**. These scopes are all non-sensitive, so Google doesn't require a review for them.

## Step 5: Create the Client ID

1. Go to **Google Auth Platform → Clients → Create client**.
2. Set **Application type** to **Web application** and give it a name such as "GitHub Pages".
3. Under **Authorized JavaScript origins**, click **Add URI** and enter your origin from Step 1, for example
   `https://YOUR-GITHUB-USERNAME.github.io`
   - Leave **Authorized redirect URIs** empty, because this app doesn't use them.
4. Click **Create** and copy the **Client ID**. It looks like `1234567890-abc123.apps.googleusercontent.com`. You don't need the client secret, and it shouldn't go in the app.

The Client ID isn't a secret. It's safe to have in a public GitHub repository.

## Step 6: Add the Client ID to the app

1. Open `index.html` in a text editor (or click the file on GitHub, then the pencil icon to edit it).
2. Find this line, which is near the "Google sign-in & Drive sync" section of the script:
   ```js
   const GOOGLE_CLIENT_ID = '';
   ```
3. Paste your Client ID between the quotes:
   ```js
   const GOOGLE_CLIENT_ID = '1234567890-abc123.apps.googleusercontent.com';
   ```
4. Save and commit. GitHub Pages updates within a few minutes.

## Step 7: Try it

1. Open your GitHub Pages address on your phone.
2. Tap **Sign in with Google** at the top of the setup screen and choose your account. Allow the permissions.
3. If the phone already has saved games, the app asks whether to **Add to Account**. Choosing that uploads them.
4. Sign in on a second device and your games and rosters appear.

**Tip:** on your phone, use the browser's **Add to Home Screen** option to open the app like a regular app.

## Using sync day to day

| What you see | What it means |
|---|---|
| **Synced** (green) | Everything is saved to your Google account |
| **Saving / Syncing** | Changes are being uploaded |
| **Offline** | No internet. Changes are safe on the phone and sync later |
| **Reconnect** (yellow) | Google sign-in lasts about an hour. Tap it to renew; it's one tap, with no password |

- **Where the status shows:** in the account box at the top of the setup screen, and as a small pill in the top bar on the other screens.
- **Signing out:** this removes your account's data from that device. It stays safe in your Google account.
- **Track a live game on one device at a time.** If two devices record plays in the same game at the same moment, the most recent device's version wins. Game history, rosters and deletions merge safely across devices.

## Troubleshooting

| Problem | Fix |
|---|---|
| Sign-in button is greyed out: "isn't set up yet" | The Client ID is missing from `index.html` (Step 6) |
| Sign-in button is greyed out: "works when the app is opened from its web address" | Open the GitHub Pages link, not the downloaded file |
| Google shows **Error 400: origin_mismatch** or "not a valid origin" | The Authorized JavaScript origin must exactly match your site, like `https://YOUR-GITHUB-USERNAME.github.io`, with no path and no trailing slash (Step 5) |
| Google shows **Access blocked: app has not completed the Google verification process** | The app is in Testing mode. Add the email as a test user, or publish the app (Step 4) |
| Signed in, but it says "Couldn't sync" | Make sure the **Google Drive API** is enabled (Step 3) |
| "Allow Google Drive access so your games can sync" | During sign-in, leave the Drive permission checkbox ticked |
| Nothing happens when tapping Sign in | Allow pop-ups for the site in the phone's browser settings |
