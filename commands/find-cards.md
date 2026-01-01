---
name: find-cards
description: Search for Anki flashcards using keywords or Anki query syntax
args:
  - name: query
    description: Search query (simple keyword or Anki syntax like 'deck:Name tag:foo is:due')
    required: true
  - name: max
    description: Maximum number of results to display (default: 20)
    required: false
  - name: sort
    description: Sort results by 'interval', 'ease', 'deck', or 'created' (default: none)
    required: false
allowed-tools: Bash(curl:*)
---

Search for Anki flashcards using either simple keyword search or Anki's advanced query syntax.

## Process

1. **Validate arguments**
   - Ensure query is provided
   - Set max to 20 if not specified
   - Validate sort option if provided (must be: interval, ease, deck, or created)

2. **Process query string**
   - Detect if simple keyword or Anki syntax
   - If query contains special operators (deck:, tag:, is:, prop:, rated:, -, OR, AND), use as-is
   - Otherwise, wrap in Front field search: `Front:*${query}*`

3. **Find matching cards**
   ```bash
   curl -X POST http://localhost:8765 \
     -H "Content-Type: application/json" \
     -d '{
       "action": "findCards",
       "version": 6,
       "params": {
         "query": "${processed_query}"
       }
     }'
   ```

4. **Process results**
   - Extract card IDs array from response
   - Get total count of matches
   - Limit to first ${max} card IDs
   - Handle case when no cards found

5. **Get card details**
   ```bash
   curl -X POST http://localhost:8765 \
     -H "Content-Type: application/json" \
     -d '{
       "action": "cardsInfo",
       "version": 6,
       "params": {
         "cards": [limited_card_ids_array]
       }
     }'
   ```

6. **Sort results (if --sort specified)**
   - **interval**: Sort by interval descending (longest intervals first = most mature cards)
   - **ease**: Sort by ease ascending (lowest ease first = struggling cards)
   - **deck**: Sort by deckName alphabetically
   - **created**: Sort by cardId ascending (older cards first)

   Use jq for sorting:
   ```bash
   if [ "$sort" = "interval" ]; then
     sorted=$(echo "$cards_info" | jq '.result | sort_by(-.interval)')
   elif [ "$sort" = "ease" ]; then
     sorted=$(echo "$cards_info" | jq '.result | sort_by(.ease // 0)')
   elif [ "$sort" = "deck" ]; then
     sorted=$(echo "$cards_info" | jq '.result | sort_by(.deckName)')
   elif [ "$sort" = "created" ]; then
     sorted=$(echo "$cards_info" | jq '.result | sort_by(.cardId)')
   else
     sorted=$(echo "$cards_info" | jq '.result')
   fi
   ```

7. **Format and display results**

   Display format:
   ```
   Found 47 cards matching "Ebbinghaus" (showing first 20):

   1. What is the Ebbinghaus Forgetting Curve?
      Deck: Memory | Tags: psychology, learning
      Interval: 14 days | Ease: 250% | Card ID: 1767311298310

   2. Who discovered the spacing effect?
      Deck: Memory | Tags: psychology, history
      Interval: 7 days | Ease: 210% | Card ID: 1767311298311

   [... more cards ...]

   Total matches: 47 | Showing: 20 | Use --max to see more
   ```

   Formatting logic:
   ```bash
   echo "Found $total_count cards matching \"$query\" (showing first $shown_count):"
   echo ""

   card_num=1
   echo "$sorted" | jq -c '.[]' | while read -r card; do
     front=$(echo "$card" | jq -r '.fields.Front.value // .question')
     deck=$(echo "$card" | jq -r '.deckName')
     tags=$(echo "$card" | jq -r 'if .tags | length > 0 then .tags | join(", ") else "none" end')
     interval=$(echo "$card" | jq -r '.interval')
     ease=$(echo "$card" | jq -r '.ease // 0')
     card_id=$(echo "$card" | jq -r '.cardId')

     # Truncate front if too long
     if [ ${#front} -gt 80 ]; then
       front="${front:0:77}..."
     fi

     # Format ease as percentage (Anki stores as 2500 = 250%)
     ease_pct=$((ease / 10))

     # Format interval
     if [ "$interval" -lt 0 ]; then
       interval_text="learning"
     elif [ "$interval" -eq 0 ]; then
       interval_text="new"
     else
       interval_text="${interval} days"
     fi

     echo "${card_num}. ${front}"
     echo "   Deck: ${deck} | Tags: ${tags}"
     echo "   Interval: ${interval_text} | Ease: ${ease_pct}% | Card ID: ${card_id}"
     echo ""

     card_num=$((card_num + 1))
   done

   if [ $total_count -gt $shown_count ]; then
     echo "Total matches: $total_count | Showing: $shown_count | Use --max to see more"
   else
     echo "Total matches: $total_count"
   fi
   ```

## Error Handling

- **Anki not running**: "Cannot connect to Anki. Make sure Anki Desktop is running with AnkiConnect installed."
- **No results**: "No cards found matching '${query}'. Try a different search term or use Anki query syntax (deck:Name, tag:foo, is:due)."
- **Invalid query syntax**: If AnkiConnect returns error, display: "Invalid query: ${error_message}"
- **Invalid --max value**: If max is not a number or < 1, default to 20
- **Invalid --sort value**: If sort is not one of (interval, ease, deck, created), show error and list valid options

## Example Usage

### Simple keyword search
```bash
/find-cards --query "photosynthesis"
# Searches Front field for "photosynthesis"

/find-cards --query "capital" --max 5
# Show only first 5 results
```

### Anki query syntax
```bash
/find-cards --query "deck:Memory"
# All cards in Memory deck

/find-cards --query "tag:important is:due"
# Important cards due for review

/find-cards --query "deck:current -is:suspended"
# Non-suspended cards in current deck

/find-cards --query "is:new deck:Memory"
# New cards in Memory deck
```

### With sorting
```bash
/find-cards --query "deck:Memory" --sort interval
# Show Memory cards sorted by maturity (longest interval first)

/find-cards --query "is:due" --sort ease --max 10
# Show 10 most difficult due cards (lowest ease first)

/find-cards --query "tag:vocabulary" --sort deck
# Show vocabulary cards sorted alphabetically by deck

/find-cards --query "deck:Spanish" --sort created --max 15
# Show 15 oldest Spanish cards
```

### Advanced queries
```bash
/find-cards --query "deck:Biology tag:cell -tag:plant"
# Biology cards tagged 'cell' but not 'plant'

/find-cards --query "is:due prop:ivl>21"
# Due cards with interval > 21 days

/find-cards --query "rated:1 -is:suspended"
# Cards studied today that aren't suspended
```

## Anki Query Syntax Reference

Common query operators:
- `deck:Name` - Cards in specific deck
- `tag:foo` - Cards with specific tag
- `is:due` - Cards due for review
- `is:new` - New cards not yet studied
- `is:suspended` - Suspended cards
- `is:buried` - Buried cards
- `-deck:Name` - NOT in deck (minus prefix negates)
- `prop:ivl>21` - Cards with interval > 21 days
- `prop:ease<2` - Cards with ease < 200%
- `rated:1` - Reviewed in last 1 day
- `added:7` - Added in last 7 days
- `"deck:My Deck"` - Use quotes for names with spaces
- `OR` / `AND` - Combine queries: `tag:a OR tag:b`

For full query syntax, see: https://docs.ankiweb.net/searching.html
