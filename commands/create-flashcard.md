---
name: create-flashcard
description: Create a new Anki flashcard - provide both sides or just one and let Claude generate the other
args:
  - name: content
    description: The content for the flashcard (Claude will generate the other side)
    required: false
  - name: front
    description: The front content of the flashcard (the question or prompt)
    required: false
  - name: back
    description: The back content of the flashcard (the answer)
    required: false
  - name: deck
    description: The deck name to add the card to (defaults to "Default")
    required: false
  - name: tags
    description: Optional comma-separated tags for the card
    required: false
---

Create a new Anki flashcard using the AnkiConnect API. You can provide both sides of the card, or just one side and Claude will intelligently generate the other.

**Smart Content Generation:**
- If you provide `--content` alone, Claude will determine if it's a question or answer and generate the complementary side
- If you provide only `--front`, Claude will generate an appropriate answer for the `--back`
- If you provide only `--back`, Claude will generate an appropriate question for the `--front`
- If you provide both `--front` and `--back`, they will be used as-is without generation

**Process:**
1. Check which parameters are provided (content, front, back, deck)
2. If deck is not provided:
   - Fetch available decks from AnkiConnect API using the `deckNames` action
   - Use AskUserQuestion to prompt the user with a single-select list of available decks
   - Use the selected deck for the flashcard
3. If only partial content is provided:
   - Analyze the content to understand its context
   - Generate the missing side intelligently based on the content type
   - For facts/definitions: generate a question
   - For questions: generate an answer
   - Use the deck and tags to understand the subject matter context
4. Parse tags if provided (split by comma)
5. Construct the AnkiConnect API request to add a note
6. Make the API call using curl
7. Handle the response and report success or errors

**API Request Formats:**

Get available decks (if deck not provided):
```bash
curl -X POST http://localhost:8765 \
  -H "Content-Type: application/json" \
  -d '{
    "action": "deckNames",
    "version": 6
  }'
```

Then use AskUserQuestion to present the decks to the user:
```
Use the AskUserQuestion tool with:
- question: "Which deck should this flashcard be added to?"
- header: "Deck"
- options: Array of deck names returned from the API (convert to label/description format)
- multiSelect: false
```

Add the note with the selected deck:
```bash
curl -X POST http://localhost:8765 \
  -H "Content-Type: application/json" \
  -d '{
    "action": "addNote",
    "version": 6,
    "params": {
      "note": {
        "deckName": "{{deck}}",
        "modelName": "Basic",
        "fields": {
          "Front": "{{front}}",
          "Back": "{{back}}"
        },
        "tags": {{tags_array}}
      }
    }
  }'
```

**Error Handling:**
- If Anki is not running, provide a clear message asking the user to start Anki
- If no decks exist when fetching deck list, suggest creating a deck first
- If the specified deck doesn't exist (when deck is provided as argument), offer to create it or list available decks
- If AnkiConnect returns an error, parse and display it clearly
- Limit deck selection options to a maximum of 4 decks in the prompt (if more than 4, show the 4 most recently used or alphabetically first 4)

**Success Response:**
Report the card ID that was created, confirm which deck it was added to, and show both the front and back content (including any generated content).

**Example Usage:**

Full specification (both sides provided):
```
/create-flashcard --front "What is the capital of France?" --back "Paris" --deck "Geography" --tags "europe,capitals"
```

Quick mode with just content (Claude generates the other side):
```
/create-flashcard --content "Paris is the capital of France"
# Will prompt: "Which deck should this flashcard be added to?" with options from your Anki decks

/create-flashcard --content "What is the capital of France?" --deck "Geography"
# Skips prompt because deck is specified
```

Provide one side only:
```
/create-flashcard --front "What is photosynthesis?" --deck "Biology"
/create-flashcard --back "The process by which plants convert sunlight into energy"
# Will prompt for deck selection
```

Minimal usage (will prompt for deck):
```
/create-flashcard --content "mitochondria is the powerhouse of the cell"
# You'll be asked to select a deck from your available decks
```
