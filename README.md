# openclaw-whoop

Single Node.js CLI for the Whoop API. No dependencies — uses only Node built-ins.

## Installation

```bash
cp whoop ~/bin/whoop   # or anywhere on PATH
chmod +x whoop
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

All subcommands support `--json` for structured output.

Token auto-refreshes on 401, so `whoop refresh` is rarely needed manually.

## Configuration

Set `WHOOP_ENV_FILE` to point to your credentials file, or use the default `~/.openclaw/credentials/whoop.env`.

```bash
export WHOOP_ENV_FILE=~/.openclaw/credentials/whoop.env
```

## OAuth Setup

### 1. Create a Whoop Developer App

1. Go to [developer.whoop.com](https://developer.whoop.com)
2. Create an application
3. Set redirect URI to `http://localhost:8080/callback`
4. Note your Client ID and Client Secret

### 2. Configure Credentials

```bash
cp whoop.env.example ~/.openclaw/credentials/whoop.env
```

Edit the file with your Client ID and Client Secret.

### 3. Get Initial Tokens

Build the authorization URL:

```
https://api.prod.whoop.com/oauth/oauth2/auth?client_id=YOUR_CLIENT_ID&redirect_uri=http://localhost:8080/callback&response_type=code&scope=read:recovery read:sleep read:workout read:cycles read:profile offline&state=random123
```

Open it in a browser, authorize, then exchange the code:

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

## License

MIT
