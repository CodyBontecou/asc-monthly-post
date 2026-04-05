# ASC Daily Check

Generate tweet-ready App Store sales summaries from your App Store Connect data. Optionally auto-post to Twitter/X, Bluesky, Threads, and more via [Post Bridge](https://post-bridge.com).

## Example Output

**Daily:**
```
I made $47.99 on Apr 4.

📝 Sync․md — 12 downloads ($23.99)
❤️ Health․md — 8 downloads
💬 Instarep․ly — 3 downloads ($24.00)
```

**Monthly:**
```
I made $327.25 in March 2026.

❤️ Health․md — 570 downloads ($263.99)
📝 Sync․md — 577 downloads ($63.26)
🖼️ imghost — 11 downloads
🎙️ Voxboard — 11 downloads
💬 Instarep․ly — 5 downloads
```

## Prerequisites

1. **Node.js 18+**
2. **[asc CLI](https://github.com/mariozechner/App-Store-Connect-CLI)** - The App Store Connect CLI tool
3. **App Store Connect API credentials** - An API key from App Store Connect

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/asc-daily-check.git
cd asc-daily-check
```

### 2. Install and configure the asc CLI

```bash
# Install globally
npm install -g @mariozechner/asc

# Configure with your App Store Connect API credentials
asc setup
```

During setup, you'll need:
- **Issuer ID** - Found in App Store Connect → Users and Access → Integrations → App Store Connect API
- **Key ID** - The ID of your API key
- **Private Key (.p8 file)** - Download this when creating your API key

### 3. Create your configuration file

```bash
cp config.example.json config.json
```

Edit `config.json` with your details:

```json
{
  "asc": {
    "vendorNumber": "YOUR_VENDOR_NUMBER"
  },
  "postBridge": {
    "apiKey": "YOUR_POST_BRIDGE_API_KEY",
    "defaultAccountId": null
  }
}
```

**Finding your Vendor Number:**
- Go to [App Store Connect → Sales and Trends](https://appstoreconnect.apple.com/trends)
- Click on any report filter — your vendor number appears in the URL or report options
- It's typically an 8-digit number like `92465427`

## Usage

### Basic Usage

```bash
# Generate tweet for yesterday's sales
node daily-sales.js

# Specific date
node daily-sales.js --date 2026-04-04

# Output as JSON
node daily-sales.js --json

# Output JSON with tweet included
node daily-sales.js --all
```

### Monthly Reports

```bash
# Generate monthly summary (aggregates all days, converts to USD)
node daily-sales.js --month 2026-03

# Previous month (auto-calculated)
node daily-sales.js --month last

# Monthly report as JSON
node daily-sales.js --month last --json

# Post monthly report to social media
node daily-sales.js --month last --post --account 54185
```

Monthly reports:
- Fetch daily sales data for every day in the month
- Convert all currencies to USD using built-in exchange rates
- Aggregate downloads and revenue by app
- Generate a tweet-ready summary

### Using npm scripts

```bash
npm run check              # Yesterday's sales
npm run check:yesterday    # Same as above
npm run check:date 2026-04-04  # Specific date
npm run check:month 2026-03    # Specific month
npm run check:last-month       # Previous month
```

## Optional: Auto-Post to Social Media

This tool integrates with [Post Bridge](https://post-bridge.com) to automatically post your sales updates to Twitter/X, Bluesky, Threads, LinkedIn, and more.

### Setup Post Bridge

1. **Get an API key** from [post-bridge.com](https://post-bridge.com)
2. **Connect your social accounts** in the Post Bridge dashboard
3. **Run the setup wizard:**

```bash
node daily-sales.js --setup
```

This will prompt you for your API key and let you set a default account.

### Posting Commands

```bash
# List connected accounts
node daily-sales.js --accounts

# Post to your default account
node daily-sales.js --post

# Post to specific account(s)
node daily-sales.js --post --account 123
node daily-sales.js --post --account 123,456,789

# Create a draft instead of posting
node daily-sales.js --draft
```

## Optional: Automated Daily Posts (macOS)

Use macOS launchd to run the script automatically every morning.

### 1. Copy and customize the plist

```bash
cp com.example.asc-daily-check.plist ~/Library/LaunchAgents/com.yourname.asc-daily-check.plist
```

### 2. Edit the plist file

Update these values in the plist:
- Path to Node.js (run `which node` to find it)
- Path to this project directory
- Your vendor number
- Optionally add `--post` to auto-publish

```xml
<key>ProgramArguments</key>
<array>
    <string>/opt/homebrew/bin/node</string>
    <string>/Users/yourname/dev/asc-daily-check/daily-sales.js</string>
    <!-- Uncomment to auto-post -->
    <!-- <string>--post</string> -->
</array>

<key>WorkingDirectory</key>
<string>/Users/yourname/dev/asc-daily-check</string>

<key>StartCalendarInterval</key>
<dict>
    <key>Hour</key>
    <integer>9</integer>  <!-- 9 AM -->
    <key>Minute</key>
    <integer>0</integer>
</dict>
```

### 3. Load the launch agent

```bash
launchctl load ~/Library/LaunchAgents/com.yourname.asc-daily-check.plist
```

### Managing the launch agent

```bash
# Unload (disable)
launchctl unload ~/Library/LaunchAgents/com.yourname.asc-daily-check.plist

# Run immediately (test)
launchctl start com.yourname.asc-daily-check

# Check logs
tail -f /tmp/asc-daily-check.log
tail -f /tmp/asc-daily-check.error.log
```

## Optional: GitHub Actions (Monthly Reports)

The included GitHub Action runs on the **2nd of each month** at 8:10 AM PST to generate and post a monthly sales summary. It runs on the 2nd to ensure all data from the previous month is available.

### What it does

1. Fetches daily sales data for every day of the previous month
2. Aggregates downloads and revenue by app (with USD conversion)
3. Generates a tweet-ready summary
4. Posts to your configured social accounts
5. Saves the JSON report as a GitHub artifact (90-day retention)

### Setup

1. Go to your repo's **Settings → Secrets and variables → Actions**
2. Add these secrets:

| Secret | Description |
|--------|-------------|
| `ASC_API_KEY` | Contents of your .p8 private key file |
| `ASC_KEY_ID` | Your API key ID |
| `ASC_ISSUER_ID` | Your issuer ID |
| `ASC_VENDOR_NUMBER` | Your vendor number |
| `POST_BRIDGE_API_KEY` | Your Post Bridge API key |
| `POST_BRIDGE_ACCOUNT_IDS` | Comma-separated account IDs (e.g., `54185,54226`) |

### Manual Trigger

You can trigger the workflow manually from the **Actions** tab in GitHub:

1. Go to **Actions → Monthly Sales Report**
2. Click **Run workflow**
3. Optionally specify a month (e.g., `2026-03`) or leave as `last` for previous month
4. Click **Run workflow**

The workflow will generate the report, post to social media, and save the JSON as a downloadable artifact.

## Customization

### App Emojis

Edit the `APP_EMOJIS` object in `daily-sales.js` to customize the emoji shown for each of your apps:

```javascript
const APP_EMOJIS = {
  "Sync.md": "📝",
  "Health.md": "❤️",
  "Instarep.ly": "💬",
  "Your App Name": "🚀",
};
```

### Tweet Format

The `generateTweet()` function in `daily-sales.js` controls the output format. Modify it to match your preferred style.

## Troubleshooting

### "No sales data available for this date"

- Sales data is typically available ~24 hours after the sale
- Check that your vendor number is correct
- Verify your asc CLI is configured properly: `asc apps list`

### "Post Bridge not configured"

Run the setup wizard:
```bash
node daily-sales.js --setup
```

### "No account specified"

Either set a default account during setup, or specify one with `--account`:
```bash
node daily-sales.js --accounts  # List available accounts
node daily-sales.js --post --account 123
```

### launchd not running

Check that the plist is valid:
```bash
plutil -lint ~/Library/LaunchAgents/com.yourname.asc-daily-check.plist
```

Ensure paths in the plist are absolute and correct.

## License

MIT
