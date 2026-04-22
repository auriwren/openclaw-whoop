# whoop-cli

A small, zero-dependency Node.js CLI for the Whoop developer API. Useful as a deterministic data-access layer underneath dashboards, reports, and scheduled jobs.

Two scripts. Node 18+ only (uses the built-in `fetch`).

- `whoop` — the CLI itself. Commands: `summary` (default), `recovery`, `sleep`, `strain`, `workouts`, `refresh`. All support `--json` for machine-readable output. Access tokens auto-refresh on 401.
- `whoop-authorize` — one-time OAuth helper. Opens your browser, catches the redirect on `localhost:8080`, writes access + refresh tokens to your env file.

## Installation

```bash
git clone https://github.com/auriwren/whoop-cli.git ~/projects/whoop-cli
cd ~/projects/whoop-cli
chmod +x whoop whoop-authorize

# Put both scripts on your PATH (pick one)
ln -s "$PWD/whoop"           ~/.local/bin/whoop
ln -s "$PWD/whoop-authorize" ~/.local/bin/whoop-authorize
# or:
echo "export PATH=$PWD:\$PATH" >> ~/.zshrc && source ~/.zshrc
```

## Configuration

The CLI reads credentials from `~/.config/whoop-cli/whoop.env` by default — the standard XDG Base Directory location. Override by setting `WHOOP_ENV_FILE` to any path.

```bash
export WHOOP_ENV_FILE=~/.config/whoop-cli/whoop.env
```

## OAuth setup

### 1. Create a Whoop developer app

1. Sign in at [developer-dashboard.whoop.com](https://developer-dashboard.whoop.com) with your Whoop account.
2. Click **Create New App**.
3. Redirect URI: `http://localhost:8080/callback`
4. Scopes: `read:recovery`, `read:sleep`, `read:cycles`, `read:workout`, `read:profile`, `read:body_measurement`, `offline`.
5. Copy the **Client ID** and **Client Secret**.

### 2. Create your credentials file

```bash
mkdir -p ~/.config/whoop-cli
cp whoop.env.example ~/.config/whoop-cli/whoop.env
chmod 600 ~/.config/whoop-cli/whoop.env
```

Edit `~/.config/whoop-cli/whoop.env` and paste in your Client ID and Client Secret.

### 3. Get initial tokens

```bash
whoop-authorize
```

Opens your browser, catches the redirect, exchanges the code for tokens, and writes them back to your env file. From this point forward the main `whoop` CLI refreshes the access token automatically on 401.

If you'd rather do it by hand, the authorization URL is:

```
https://api.prod.whoop.com/oauth/oauth2/auth?client_id=YOUR_CLIENT_ID&redirect_uri=http://localhost:8080/callback&response_type=code&scope=read:recovery read:sleep read:workout read:cycles read:profile offline&state=random123
```

And the token exchange:

```bash
curl -s -X POST "https://api.prod.whoop.com/oauth/oauth2/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE_FROM_REDIRECT" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "redirect_uri=http://localhost:8080/callback"
```

Copy the `access_token` and `refresh_token` into your env file.

### 4. Test

```bash
whoop summary
```

## Usage

```bash
whoop                    # daily summary (default)
whoop summary            # same as above
whoop summary --json     # machine-readable JSON
whoop recovery           # recovery score only
whoop sleep              # sleep details only
whoop strain             # day strain only
whoop workouts           # recent workouts
whoop workouts --limit 3 # limit workout count
whoop refresh            # manually refresh OAuth token
```

All subcommands support `--json` for structured output. Token auto-refreshes on 401, so `whoop refresh` is rarely needed manually.

## Using it as a data source for dashboards

The CLI is intentionally boring. Anything downstream — a medical report, a training-load chart, a weekly email, a cross-metric scan — is built on top of `whoop <command> --json` output. The CLI handles OAuth, refresh, and endpoints; you handle the rest.

A typical dashboard pipeline looks like:

```bash
whoop recovery --json > out/recovery.json
whoop sleep    --json > out/sleep.json
whoop strain   --json > out/strain.json

# Your renderer reads those three files and writes an HTML page.
node render-report.js
```

This repo stops at the CLI. The renderer is yours.

## Endpoints used

- `GET /developer/v2/recovery?limit=1`
- `GET /developer/v2/activity/sleep?limit=1`
- `GET /developer/v2/cycle?limit=1`
- `GET /developer/v2/activity/workout?limit=N`
- `POST https://api.prod.whoop.com/oauth/oauth2/token` (refresh + initial exchange)

Full API reference: [developer.whoop.com](https://developer.whoop.com).

## Troubleshooting

**`401` on every call and refresh fails.** Your refresh token has been revoked. Re-run `whoop-authorize` to redo the one-time OAuth handshake.

**`429 Rate Limited`.** Whoop allows 100 requests per minute per client. Back off or paginate.

**`Env file not found`.** Set `WHOOP_ENV_FILE` explicitly, or create the file at the default location (`~/.config/whoop-cli/whoop.env`).

**Client secret committed to git by accident.** Rotate it immediately in the Whoop developer dashboard. The old value lives on GitHub's servers for some time even after deletion — rotation is the only real fix.

## License

MIT.
