# Anki Claude Plugin

A Claude Code plugin for creating and managing Anki flashcards directly from your terminal.

## Installation

```
/plugin install jtmuller5/anki-claude-plugin
```

## Develop Locally
Open the project directory and run:
```
claude --plugin-dir .
```

## What This Plugin Does

This plugin integrates Claude Code with Anki via the AnkiConnect API, allowing you to:

- **Create flashcards** with custom front/back content, deck assignment, and tags
- **Search flashcards** using keywords or advanced Anki query syntax
- **Review due cards** to see what flashcards need studying
- **Create new decks** to organize your flashcards
- **List all decks** with statistics about cards and reviews
- **Batch operations** through the flashcard manager agent for creating multiple cards at once

## Available Commands

### `/create-flashcard`
Create a new Anki flashcard - provide both sides or just one and let Claude generate the other.

**Quick mode (recommended):**
```bash
/create-flashcard --content "Paris is the capital of France"
# Will prompt you to select a deck from your available decks

/create-flashcard --content "What is photosynthesis?" --deck "Biology"
# Skips prompt - uses specified deck
```

**Full specification:**
```bash
/create-flashcard --front "What is the capital of France?" --back "Paris" --deck "Geography" --tags "europe,capitals"
```

**Arguments:**
- `--content` (optional): Single piece of content - Claude will generate the complementary side
- `--front` (optional): The question or prompt
- `--back` (optional): The answer or explanation
- `--deck` (optional): The deck to add the card to (prompts if not provided)
- `--tags` (optional): Comma-separated tags

**Smart Features:**
- **Content generation:** If you only provide one side, Claude will intelligently generate the other based on context. Questions become answers, facts become questions.
- **Interactive deck selection:** If you don't specify a deck, you'll be prompted with a list of your available Anki decks to choose from.

### `/find-cards`
Search for Anki flashcards using either simple keywords or Anki's advanced query syntax.

**Simple keyword search:**
```bash
/find-cards --query "photosynthesis"
# Searches for "photosynthesis" in card fronts

/find-cards --query "capital" --max 5
# Show only first 5 results
```

**Advanced Anki syntax:**
```bash
/find-cards --query "deck:Memory tag:important"
# Find important cards in Memory deck

/find-cards --query "is:due" --sort ease --max 10
# Show 10 most difficult due cards

/find-cards --query "deck:Biology -is:suspended"
# Non-suspended Biology cards
```

**Arguments:**
- `--query` (required): Search term or Anki query syntax
- `--max` (optional): Maximum results to display (default: 20)
- `--sort` (optional): Sort by `interval`, `ease`, `deck`, or `created`

**Output shows:** Front text, deck, tags, interval, ease percentage, and card ID for each match.

**Common query operators:** `deck:Name`, `tag:foo`, `is:due`, `is:new`, `is:suspended`, `-tag:bar` (negation), `prop:ivl>21` (interval), `rated:1` (studied today)

### `/review-due`
Fetch flashcards that are currently due for review.

```bash
/review-due
/review-due --deck "Spanish Vocabulary" --limit 5
```

**Arguments:**
- `--deck` (optional): Filter by specific deck
- `--limit` (optional): Maximum number of cards to display (default: 10)

### `/create-deck`
Create a new Anki deck.

```bash
/create-deck --name "Spanish Vocabulary"
/create-deck --name "Computer Science::Algorithms"
```

**Arguments:**
- `--name` (required): Name of the deck (supports nested decks with `::`\)

### `/list-decks`
List all existing Anki decks with statistics.

```bash
/list-decks
```

**No arguments required.** Shows deck names, total cards, due cards, and new cards.

## Setup Instructions

### Local Computer Setup (Mac/Linux/Windows)

#### 1. Install Anki Desktop

Download and install Anki from the official website:
- Visit: https://apps.ankiweb.net/
- Download the version for your operating system
- Install and run Anki

#### 2. Install AnkiConnect Add-on

AnkiConnect is required for this plugin to communicate with Anki.

1. Open Anki Desktop
2. Go to **Tools → Add-ons**
3. Click **Get Add-ons...**
4. Enter the code: `2055492159`
5. Click **OK** and restart Anki

#### 3. Verify AnkiConnect is Running

- AnkiConnect runs on `http://localhost:8765` by default
- Keep Anki Desktop open whenever you want to use this plugin
- Test the connection by running: `/list-decks`

### Mobile Device Setup (Android)

#### 1. Install AnkiDroid

- Download **AnkiDroid** (free) from the Google Play Store
- AnkiDroid is the official Anki client for Android

**Note for iOS users:** Use **AnkiMobile** (paid app) from the App Store instead.

#### 2. Set Up AnkiWeb Sync

AnkiWeb enables syncing between your computer and mobile device.

1. **Create an AnkiWeb account:**
   - Visit: https://ankiweb.net/account/register
   - Create a free account (no cost, unlimited flashcards)

2. **Configure sync on your Mac/PC:**
   - Open Anki Desktop
   - Click the **Sync** button (or press `Y`)
   - Enter your AnkiWeb credentials
   - Wait for initial upload to complete

3. **Configure sync on your Android/iOS device:**
   - Open AnkiDroid (or AnkiMobile)
   - Go to **Settings → AnkiWeb account**
   - Enter your AnkiWeb credentials
   - Tap the sync button to download your collection

#### 3. Syncing Workflow

To keep your devices in sync:

1. **Before studying on any device:** Tap/click the sync button
2. **After finishing a session:** Tap/click the sync button again
3. **On your computer:** Sync before and after creating cards with this plugin

**Important notes:**
- Syncing is bi-directional (changes sync both ways)
- First sync may take time depending on collection size
- Media files (images, audio) sync but may take longer
- Always sync before switching devices

## How It Works

This plugin uses the **AnkiConnect API** to interact with Anki locally on your computer. The API only works when Anki Desktop is running on the same machine where you're using Claude Code.

For cross-device syncing (Mac ↔ Android/iOS), you'll use **AnkiWeb's built-in sync** through the Anki/AnkiDroid/AnkiMobile apps directly.

## Workflow Example

```bash
# Create a new deck
/create-deck --name "Japanese Vocabulary"

# Quick flashcard creation (Claude generates the other side, prompts for deck)
/create-flashcard --content "こんにちは means hello in Japanese"
# → You'll be prompted to select from: "Default", "Japanese Vocabulary", etc.
# → Select "Japanese Vocabulary"

# Even quicker - specify the deck to skip prompt
/create-flashcard --content "Tokyo is the capital of Japan" --deck "Japanese Vocabulary" --tags "geography"

# Or provide both sides explicitly
/create-flashcard --front "さようなら" --back "Goodbye" --deck "Japanese Vocabulary" --tags "greetings,common"

# Check what's due for review
/review-due --deck "Japanese Vocabulary"

# Search for specific cards
/find-cards --query "deck:Japanese Vocabulary tag:greetings"

# Find cards you're struggling with
/find-cards --query "is:due" --sort ease --max 10

# Sync to mobile (open Anki Desktop and click Sync)
# Study on your phone using AnkiDroid/AnkiMobile
# Sync back when done
```

## Troubleshooting

### "Connection refused" or "Cannot connect to Anki"
- Make sure Anki Desktop is running
- Verify AnkiConnect add-on is installed (check Tools → Add-ons)
- Restart Anki if you just installed AnkiConnect

### "Deck not found"
- Use `/list-decks` to see available decks
- Create the deck first with `/create-deck --name "YourDeck"`

### Cards not appearing on mobile
- Ensure you've synced on your computer (click Sync in Anki)
- Open your mobile app and tap the sync button
- Check your internet connection

### Sync conflicts
- If you edited cards on both devices without syncing, you may see a conflict dialog
- Choose which version to keep (usually the most recent)
- Always sync before and after studying to avoid conflicts

## Requirements

- **Anki Desktop** (Mac/Windows/Linux)
- **AnkiConnect add-on** (code: 2055492159)
- **AnkiWeb account** (free, for mobile sync)
- **AnkiDroid** (Android) or **AnkiMobile** (iOS) for mobile studying

## Links

- [Anki Desktop](https://apps.ankiweb.net/)
- [AnkiConnect Add-on](https://ankiweb.net/shared/info/2055492159)
- [AnkiWeb Registration](https://ankiweb.net/account/register)
- [AnkiDroid (Google Play)](https://play.google.com/store/apps/details?id=com.ichi2.anki)
- [AnkiMobile (App Store)](https://apps.apple.com/us/app/ankimobile-flashcards/id373493387)
