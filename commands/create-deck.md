---
name: create-deck
description: Create a new Anki deck
args:
  - name: name
    description: The name of the deck to create
    required: true
---

Create a new Anki deck using the AnkiConnect API.

**Process:**
1. Validate that the deck name is provided
2. Check if the deck already exists
3. If it doesn't exist, create it using the createDeck action
4. Report success or error

**API Request Format:**
```bash
curl -X POST http://localhost:8765 \
  -H "Content-Type: application/json" \
  -d '{
    "action": "createDeck",
    "version": 6,
    "params": {
      "deck": "{{name}}"
    }
  }'
```

**Validation:**
Before creating, check if deck exists:
```bash
curl -X POST http://localhost:8765 \
  -H "Content-Type: application/json" \
  -d '{
    "action": "deckNames",
    "version": 6
  }'
```

**Error Handling:**
- If Anki is not running, provide clear error message
- If deck already exists, inform the user and ask if they want to use it
- Handle API errors gracefully

**Success Response:**
Confirm that the deck was created and provide the deck ID if available.

**Example Usage:**
```
/create-deck --name "Spanish Vocabulary"
/create-deck --name "Computer Science::Algorithms"
```

**Note:** You can create nested decks using "::" separator (e.g., "Parent::Child").
