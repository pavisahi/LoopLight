# LoopLight

A personal attention-management agent, built for Week 3 of The Gen AI Academy's 
Mastering Agentic AI Certification (Track: No-code / n8n).
![LoopLight workflow illustration](docs/looplight-cover.png)

## What it does

You dump whatever's on your mind into a form — messy, unstructured, however it 
comes out. LoopLight extracts real, persistent items from it, asks a quick 
capacity check-in, and when you're ready, an agent reads everything still open and 
recommends up to three things worth doing right now — weighed against urgency, 
deadlines, and how much time/energy you actually have. Nothing gets acted on 
without your approval. The goal here is not capture and triage, but simplify 
decision making to reduce overwhelm when dealing with large to-do lists. 

**One-liner:** My agent helps me triage a running brain-dump into a short, ranked 
set of next-action recommendations inside an n8n form, replacing the mental 
overhead of manually re-reading and re-prioritizing scattered notes myself. It 
does the recommending on its own using tool calls against my own persistent task 
memory, hands off to me before anything is marked in progress, and I'll know it 
works when I can go from a raw brain dump to a chosen next action in under 2 
minutes, with a recommendation I'd actually act on most of the time.

## Architecture

Single ReAct agent embedded inside a deterministic n8n pipeline — not multi-agent 
orchestration. One component (the Decision Agent) calls a tool, reasons over the 
result, and makes a real judgment call. Everything else — extraction, sheet 
writes, branching — is fixed logic, even where an LLM is involved.

```
Form: What's on your mind?
        │
Information Extractor (LLM) ──▶ Write Session to Sheets
        │
Prepare Items ──▶ New Items? ──true──▶ Save Items to Sheets (status: open)
        │                                        │
        
└──────────────false────────────────────▶│
                                                   ▼
                                       Form: Got it! (capacity check-in)
                                                   │
                                       Normalize Follow-up Capacity
                                                   │
                                       Capacity Check ──false──▶ End (items 
saved, no rec.)
                                                   │
                                                  true
                                                   ▼
                                              Start Decision
                                                   │
                              Decision Agent (reads Open Items via tool, ranks up 
to 3)
                                                   │
                                       Validate Recommendation
                                                   │
                              Form: Review Recommendation (up to 3 options + None)
                                                   │
                                            Parse Selection
                                                   │
                              Item selected? ──true──▶ Update Sheet (status: 
In Progress)
                                        └──false──▶ End (no change)
```

## Stack

- **n8n** — workflow orchestration, forms, branching, Sheets I/O
- **Groq** (openai/gpt-oss models) — LLM calls for both the Information Extractor 
and the Decision Agent
- **Google Sheets** — persistent state, two tabs (`Sessions` for capacity 
check-ins, `Items` for the task backlog)
- **n8n Structured Output Parser** — enforces the Decision Agent's JSON output 
shape at the framework level, rather than relying on prompt instructions alone

## Files

- `looplight-workflow.json` — the full n8n workflow export. Import via n8n's 
canvas 
menu → Import from File.
- `LoopLight_Week3_Documentation_final.docx` — full project write-up: one-liner, 
agent 
framework table, architecture rationale, prompts, design iterations, testing log, 
and known limitations. (Also submitted separately as a Google Doc per the 
assignment's requirements.)

## Known limitations

Documented in full in the project write-up (Section 10), briefly:
- The capacity value collected on the follow-up screen gates whether a 
recommendation runs, but isn't yet threaded into the Decision Agent's own 
reasoning input.
- A second tool for the Decision Agent (flagging ambiguous items instead of 
guessing) was designed but not implemented in this version.
- Two nodes from an earlier design remain on the canvas, disconnected from the 
live execution path.

## Setup to run locally

1. Import `looplight-workflow.json` into your own n8n instance.
2. Connect your own Google Sheets OAuth credential and Groq API credential (the 
imported workflow references credential IDs from the original build, which won't 
resolve in a different account).
3. Create a Google Sheet with two tabs, `Sessions` and `Items`, matching the 
column names referenced in the workflow's Sheets nodes.
4. Activate the workflow and open the form trigger's Production URL.

