# ASC Daily Check

A simple automation to fetch daily App Store Connect sales data, generate a tweet-ready summary, and optionally post to social platforms via [Post Bridge](https://post-bridge.com).

## Setup

### 1. App Store Connect (required)

Make sure you have `asc` CLI installed and authenticated:
```bash
asc auth login
```

Set your vendor number (find it in App Store Connect → Sales and Trends → Reports URL):
```bash
export ASC_VENDOR_NUMBER="92465427"
```

Or add it to `~/.asc/config.json`:
```json
{
  "vendor_number": "92465427"
}
```

### 2. Post Bridge (optional, for auto-posting)

To automatically post to Twitter/X, Bluesky, Threads, etc., configure Post Bridge:

```bash
node daily-sales.js --setup
```

This will:
1. Prompt for your Post Bridge API key
2. Show your connected social accounts
3. Let you set a default account for daily posts

## Usage

### Generate tweet (no posting)
```bash
node daily-sales.js                    # Yesterday's sales
node daily-sales.js --date 2026-04-04  # Specific date
```

### Generate and post
```bash
node daily-sales.js --post                    # Post to default account
node daily-sales.js --post --account 123      # Post to specific account
node daily-sales.js --post --account 123,456  # Post to multiple accounts
node daily-sales.js --draft                   # Create as draft (don't publish)
```

### Manage accounts
```bash
node daily-sales.js --accounts  # List connected social accounts
node daily-sales.js --setup     # Configure API key and default account
```

### Output formats
```bash
node daily-sales.js --json  # JSON output
node daily-sales.js --all   # JSON with tweet included
```

## Example Output

```
📱 Apr 4 App Store

📥 35 downloads
💰 $12.78 (2 purchases)

Sync.md: 28 (2 paid)
Health.md: 5
Instarep.ly: 1
Voxboard: 1

🇺🇸🇻🇳🇯🇵🇳🇱🇯🇴
```

## Daily Automation (macOS)

To run and post automatically every day at 9 AM:

```bash
# Edit the plist to add --post flag if desired
# Then install the launchd job
cp com.bontecou.asc-daily-check.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.bontecou.asc-daily-check.plist
```

To unload:
```bash
launchctl unload ~/Library/LaunchAgents/com.bontecou.asc-daily-check.plist
```

## Daily Automation (cron)

Add to your crontab (`crontab -e`):

```cron
# Generate tweet at 9 AM (no auto-post)
0 9 * * * cd /Users/codybontecou/dev/asc-daily-check && /usr/local/bin/node daily-sales.js >> /tmp/asc-daily.log 2>&1

# Generate and post at 9 AM
0 9 * * * cd /Users/codybontecou/dev/asc-daily-check && /usr/local/bin/node daily-sales.js --post >> /tmp/asc-daily.log 2>&1
```

## Monthly Automation (GitHub Actions)

The repo includes a GitHub Actions workflow that runs on the **1st of every month at 8:10 AM PST** and automatically posts to Twitter/X, LinkedIn, Threads, and Bluesky.

### Required Secrets

Add these secrets in your GitHub repo (Settings → Secrets and variables → Actions):

| Secret | Description |
|--------|-------------|
| `ASC_KEY_ID` | App Store Connect API Key ID |
| `ASC_ISSUER_ID` | App Store Connect Issuer ID |
| `ASC_API_KEY` | Contents of your .p8 private key file |
| `ASC_VENDOR_NUMBER` | Your vendor number (from Sales & Trends) |
| `POST_BRIDGE_API_KEY` | Post Bridge API key (required for posting) |
| `POST_BRIDGE_ACCOUNT_IDS` | Comma-separated account IDs (e.g., `123,456,789,012`) |

### Finding Your ASC Credentials

1. Go to [App Store Connect → Users and Access → Integrations → App Store Connect API](https://appstoreconnect.apple.com/access/integrations/api)
2. Create a new key with "Sales and Reports" access
3. Download the .p8 file and note the Key ID
4. Copy the Issuer ID from the top of the page

### Finding Your Post Bridge Account IDs

1. Get your API key from [post-bridge.com](https://post-bridge.com)
2. Run locally to see your connected accounts:
   ```bash
   node daily-sales.js --setup    # Configure API key
   node daily-sales.js --accounts # List accounts with IDs
   ```
3. Copy the IDs for Twitter/X, LinkedIn, Threads, and Bluesky
4. Add them as a comma-separated string to `POST_BRIDGE_ACCOUNT_IDS`

Example output:
```
  𝕏 twitter: @codybontecou (ID: 123)
  💌 linkedin: @cody-bontecou (ID: 456)
  🧵 threads: @codybontecou (ID: 789)
  🦋 bluesky: @cody.bsky.social (ID: 012)
```

Secret value: `123,456,789,012`

### Manual Trigger

You can manually run the workflow from the Actions tab in GitHub.

## Integration with PostRelay

This tool uses the same **Post Bridge API** as the PostRelay iOS app. If you have accounts connected in PostRelay, they'll be available here too (using the same API key from post-bridge.com).
