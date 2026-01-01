---
name: review-due
description: Show count of Anki flashcards due for review by deck
allowed-tools: Bash(curl:*)
---

# Anki Review Summary

## Due Cards Data

All due card IDs: !`curl -s -X POST http://localhost:8765 -H "Content-Type: application/json" -d '{"action": "findCards", "version": 6, "params": {"query": "is:due"}}'`

Card details: !`curl -s -X POST http://localhost:8765 -H "Content-Type: application/json" -d '{"action": "cardsInfo", "version": 6, "params": {"cards": '$(curl -s -X POST http://localhost:8765 -H "Content-Type: application/json" -d '{"action": "findCards", "version": 6, "params": {"query": "is:due"}}' | grep -o '\[.*\]')'}}' | jq -r '.result[] | "\(.deckName)"' | sort | uniq -c`

## Your Task

Based on the card details above, create a clear summary showing:
1. Total number of cards due for review
2. Breakdown by deck with count for each deck
3. Format it as a clean, easy-to-read list
