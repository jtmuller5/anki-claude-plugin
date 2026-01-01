---
name: list-decks
description: List all existing Anki decks
args: []
---

List all Anki decks using the AnkiConnect API.

**Process:**
1. Make an API call to get all deck names
2. Optionally get deck statistics (card counts, due counts)
3. Format and display the results in a clear, organized way

**API Request Format:**

Get deck names:
```bash
curl -X POST http://localhost:8765 \
  -H "Content-Type: application/json" \
  -d '{
    "action": "deckNames",
    "version": 6
  }'
```

Get deck statistics:
```bash
curl -X POST http://localhost:8765 \
  -H "Content-Type: application/json" \
  -d '{
    "action": "getDeckStats",
    "version": 6,
    "params": {
      "decks": ["deck_name_here"]
    }
  }'
```

**Output Format:**
Display decks in a formatted list showing:
- Deck name
- Total cards (if stats available)
- Cards due for review (if stats available)
- New cards (if stats available)

Example output:
```
Available Anki Decks:
1. Spanish Vocabulary
   - Total: 150 cards
   - Due: 12 cards
   - New: 5 cards

2. Computer Science
   - Total: 89 cards
   - Due: 3 cards
   - New: 0 cards

3. Geography::Europe
   - Total: 45 cards
   - Due: 8 cards
   - New: 2 cards
```

**Error Handling:**
- Check if Anki is running
- Handle cases where no decks exist
- Parse and display API errors

**Example Usage:**
```
/list-decks
```
