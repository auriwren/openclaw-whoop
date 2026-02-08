# openclaw-whoop

Whoop recovery, sleep, strain, and workout data for [OpenClaw](https://github.com/openclaw).

## Features

- **Recovery scores** -- HRV (RMSSD), resting heart rate, SpO2 percentage, color-coded status (green/yellow/red)
- **Sleep analysis** -- sleep performance, efficiency, time in bed, deep sleep, REM, disturbance count
- **Day strain** -- current strain score, average and max heart rate, calories burned
- **Recent workouts** -- last 5 workouts with sport type, strain, and heart rate data
- **Auto token refresh** -- automatically refreshes expired tokens on 401 responses
- **JSON output mode** -- structured JSON output for piping to other tools

## Requirements

- bash
- curl
- jq
- bc
- A Whoop account with developer API access

## Setup

### Creating a Whoop Developer App

1. Go to [developer.whoop.com](https://developer.whoop.com)
2. Sign in with your Whoop account
3. Click **Create App** or navigate to the developer dashboard
4. Fill in your app details:
   - **Name**: anything you like (e.g. "My OpenClaw Integration")
   - **Description**: brief description of your integration
   - **Redirect URI**: use `http://localhost:8080/callback` for local development
5. Select the following scopes:
   - `read:recovery`
   - `read:cycles`
   - `read:workout`
   - `read:sleep`
   - `read:profile`
   - `read:body_measurement`
6. Note your **Client ID** and **Client Secret**
7. Copy the example env file and fill in your credentials:

```bash
cp whoop.env.example ~/.openclaw/credentials/whoop.env
# Edit the file and add your Client ID and Client Secret
```

### Getting Your First Token

Whoop uses OAuth 2.0 Authorization Code flow. You need to do this once to get your initial tokens.

**Step 1: Open the authorization URL in your browser**

Replace `CLIENT_ID` and `REDIRECT_URI` with your values:

```
https://api.prod.whoop.com/oauth/oauth2/auth?client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&response_type=code&scope=read:recovery read:cycles read:workout read:sleep read:profile read:body_measurement offline&state=random123
```

The `offline` scope is important -- it ensures you receive a refresh token.

**Step 2: Authorize the app**

Sign in and approve the requested permissions.

**Step 3: Get the authorization code**

After authorizing, you'll be redirected to your redirect URI with a `code` parameter in the URL:

```
http://localhost:8080/callback?code=AUTHORIZATION_CODE&state=random123
```

Copy the `code` value.

**Step 4: Exchange the code for tokens**

```bash
curl -X POST https://api.prod.whoop.com/oauth/oauth2/token \
  -d "grant_type=authorization_code" \
  -d "code=YOUR_CODE" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "redirect_uri=YOUR_REDIRECT_URI"
```

**Step 5: Save your tokens**

Copy the `access_token` and `refresh_token` from the response into your `whoop.env` file:

```bash
WHOOP_ACCESS_TOKEN="eyJhbG..."
WHOOP_REFRESH_TOKEN="abc123..."
```

## Usage

### Pretty-printed summary

```bash
whoop-summary
```

Output:

```
═══════════════════════════════════════
        🏋️  WHOOP DAILY SUMMARY
═══════════════════════════════════════

RECOVERY: 🟢 82%
  ❤️  RHR: 52 bpm
  📈 HRV: 68 ms
  🫁 SpO2: 97%

SLEEP: 91% performance
  🛏️  Time in bed: 7.8h
  🌊 Deep sleep: 1.4h
  💭 REM: 2.1h
  📊 Efficiency: 94%
  🔔 Disturbances: 3

DAY STRAIN: 8.2
  💓 Avg HR: 72 bpm | Max: 145 bpm
  🔥 Calories: 2150 kcal

RECENT WORKOUTS:
  🏃 Running: strain 12, 148avg/172max HR
═══════════════════════════════════════
```

### JSON output

```bash
whoop-summary --json
```

Pipe to jq for specific fields:

```bash
# Just recovery score
whoop-summary --json | jq '.recovery.score.recovery_score'

# Sleep stages
whoop-summary --json | jq '.sleep.score.stage_summary'
```

### Manual token refresh

```bash
whoop-refresh
```

### Custom env file location

```bash
WHOOP_ENV_FILE=/path/to/my/whoop.env whoop-summary
```

## Integration with OpenClaw

1. Copy the scripts to your OpenClaw tools directory:

```bash
cp whoop-summary whoop-refresh ~/.openclaw/workspace/tools/
chmod +x ~/.openclaw/workspace/tools/whoop-*
```

2. Make sure `~/.openclaw/workspace/tools` is on your PATH

3. Use in morning briefings, health dashboards, or any automation that needs your Whoop data

## Token Management

Whoop access tokens expire every hour. The scripts handle this automatically:

- `whoop-summary` detects 401 responses and calls `whoop-refresh` automatically
- `whoop-refresh` exchanges your refresh token for new access and refresh tokens
- Updated tokens are written back to your env file

You can also run `whoop-refresh` on a cron schedule (e.g. every 50 minutes) if you prefer proactive refresh.

## License

MIT
