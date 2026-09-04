# Sample Data

Example inputs and outputs from a real LoopLight run, included so a reviewer can see what the workflow actually produces without needing to spin it up and authenticate their own Google Sheets / Groq credentials first.

---

## 1. Sample brain dump (form input)

> "need to follow up with the recruiter about the offer timeline before friday. also should send mom's birthday gift, keep forgetting. team retro notes from tuesday still need to be written up sometime. thinking about trying that new sourdough recipe this weekend. dentist appointment reminder — need to actually call and book one, haven't been in a while."

## 2. Information Extractor output

```json
{
  "capacity_signal": "unknown",
  "capacity_evidence": null,
  "available_minutes": null,
  "items": [
    {
      "source_text": "need to follow up with the recruiter about the offer timeline before friday",
      "description": "Follow up with recruiter about offer timeline",
      "item_type": "task",
      "status": "open",
      "deadline_text": "before friday",
      "effort_band": "quick",
      "needs_clarification": false
    },
    {
      "source_text": "should send mom's birthday gift, keep forgetting",
      "description": "Send mom's birthday gift",
      "item_type": "task",
      "status": "open",
      "deadline_text": null,
      "effort_band": "low",
      "needs_clarification": false
    },
    {
      "source_text": "team retro notes from tuesday still need to be written up sometime",
      "description": "Write up team retro notes",
      "item_type": "task",
      "status": "open",
      "deadline_text": null,
      "effort_band": "medium",
      "needs_clarification": false
    },
    {
      "source_text": "thinking about trying that new sourdough recipe this weekend",
      "description": "Try new sourdough recipe",
      "item_type": "idea",
      "status": "open",
      "deadline_text": "this weekend",
      "effort_band": "high",
      "needs_clarification": false
    },
    {
      "source_text": "dentist appointment reminder — need to actually call and book one, haven't been in a while",
      "description": "Call to book a dentist appointment",
      "item_type": "task",
      "status": "open",
      "deadline_text": null,
      "effort_band": "quick",
      "needs_clarification": false
    }
  ]
}
```

## 3. Capacity follow-up ("Got it!" screen)

Choice selected: **"Ok, but a quickie"**

Normalized: `capacity_signal: "Ok, but a quickie"`, `has_capacity: true`

## 4. Decision Agent output

```json
{
  "candidates": [
    {
      "item_id": "2026-09-03T09:14:02-1",
      "description": "Follow up with recruiter about offer timeline",
      "why": "Explicit deadline before Friday — time-sensitive and outweighs the low-capacity signal"
    },
    {
      "item_id": "2026-09-03T09:14:02-5",
      "description": "Call to book a dentist appointment",
      "why": "Quick, no real deadline pressure, but fits a short window well"
    },
    {
      "item_id": "2026-09-03T09:14:02-2",
      "description": "Send mom's birthday gift",
      "why": "Lower urgency than the others, included as the weakest fit — no explicit deadline given"
    }
  ],
  "no_fit_message": ""
}
```

Note what's *not* recommended: the retro notes (medium effort, doesn't fit "a quickie") and the sourdough idea (high effort, and not an actionable task in the same sense) are correctly left out — this is the agent weighing effort against stated capacity, not just returning the 3 most recent items.

## 5. Resulting Items sheet (after picking candidate #1)

| item_id | description | item_type | status | deadline_text | effort_band |
|---|---|---|---|---|---|
| ...-1 | Follow up with recruiter about offer timeline | task | **In Progress** | before friday | quick |
| ...-2 | Send mom's birthday gift | task | open | — | low |
| ...-3 | Write up team retro notes | task | open | — | medium |
| ...-4 | Try new sourdough recipe | idea | open | this weekend | high |
| ...-5 | Call to book a dentist appointment | task | open | — | quick |

Only the picked item's status changes. Everything else stays `open` and remains eligible for a future recommendation.
