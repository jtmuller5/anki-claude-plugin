---
name: anki-review
description: Start an interactive Anki flashcard review session
args:
  - name: deck
    description: Optional deck name to review (if not provided, reviews all due cards)
    required: false
  - name: limit
    description: Optional maximum number of cards to review in this session
    required: false
---

Start an interactive review session where you'll review Anki flashcards one at a time, marking each with an ease rating (Again/Hard/Good/Easy).

**What This Does:**
- Opens Anki in review mode
- Shows you cards one at a time (question first, then answer)
- Asks you to rate how well you knew each card
- Submits your ratings to update Anki's spaced repetition schedule
- Continues until you want to stop or run out of cards

**Requirements:**
- Anki must be running
- AnkiConnect plugin must be installed
- Anki window should be visible (not minimized)

**How It Works:**
1. Finds cards that are due for review
2. For each card:
   - Shows the question/front
   - Waits for you to think about it
   - Shows the answer/back
   - Asks you to rate: Again(1), Hard(2), Good(3), or Easy(4)
   - Submits your rating to Anki
3. Continues until you choose to stop or complete all cards

**Example Usage:**
```bash
# Review all due cards
/review

# Review only cards from the Vocab deck
/review --deck "Vocab"

# Review up to 10 cards
/review --limit 10

# Review up to 5 cards from Software Development deck
/review --deck "Software Development" --limit 5
```

**Ease Rating Guide:**
- **Again (1):** Couldn't remember → Review very soon
- **Hard (2):** Barely remembered → Shorter interval
- **Good (3):** Remembered correctly → Normal interval (recommended)
- **Easy (4):** Instantly knew it → Longer interval

This command uses the anki-flashcard-reviewer agent to provide an interactive, one-card-at-a-time review experience.
