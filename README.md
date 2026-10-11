# Home Credit LAP Qualification – AI Voice Agent

My submission for the SalesAgents AI "AI Intern (Prompt Engineer)" assignment.

The agent calls existing Home Credit customers, tells them about a pre-approved Loan Against Property offer of up to ₹75 lakh, and runs a short eligibility check before handing qualified leads to a senior loan expert. It's built as a single system prompt and tested on Retell AI.

**Call recordings (audio):** [Google Drive folder](https://drive.google.com/drive/folders/1nql4YlgJXw9nnks9ZOxXhYRA26VH5osw)

## What's in this repo

| Path | What it is |
|---|---|
| [`prompt/system_prompt.md`](prompt/system_prompt.md) | The full system prompt, exactly as used on the agent |
| [`docs/test-results.md`](docs/test-results.md) | Every test call: what I said, what was expected, what happened, and the bugs I fixed |
| [`docs/approach-note.md`](docs/approach-note.md) | How I approached the problem and the decisions I made where the brief was open |
| [`transcripts/`](transcripts/) | One transcript per test call, with the post-call outcome and captured details at the top |

## What the agent does

1. Confirms it's speaking to the right customer. If they're busy, it takes a callback time and ends the call.
2. Presents the loyalty offer (up to ₹75 lakh).
3. Collects 7 details in order: property type, ownership, original documents, loan amount, occupation and income mode, market value, tenure. If the customer gives answers early or out of order, it notes them and doesn't ask again.
4. Stops politely the moment a rule fails: agricultural property, no original documents, cash income, or tenure outside 3–15 years.
5. If the customer asks for more than ₹75 lakh, it explains the cap and offers ₹75 lakh.
6. If the customer mentions an existing loan on the property or wants to reduce their EMI, it says a loan-transfer specialist will call, and ends the call.
7. Only after all 7 details pass does it say a senior loan expert will call back with the exact interest rate.

It also handles interruptions, side questions (it never makes up an interest rate), unclear audio and self-corrections. It supports English and Hindi through the `language_to_speak` variable, with gendered Hindi verb forms based on `agent_gender`.

## Platform setup (Retell AI)

- Single prompt agent, model GPT 5.6 Terra, voice ElevenLabs "Monika" (Indian English, female)
- Agent speaks first, `end_call` function enabled
- Dynamic variables: `company_name`, `customer_name`, `agent_name`, `agent_gender`, `current_date`, `current_day`, `current_time`, `additional_context_from_rag`, `language_to_speak`, `conversation_history`, `customer_utterance`
- Interruption sensitivity 0.49, noise and background-speech removal, accuracy transcription mode, boosted keywords (lakh, crore, EMI, salaried, agricultural, kirana, home loan), end call on 25s silence
- Post-call extraction: `call_outcome` (QUALIFIED / DISQUALIFIED / TRANSFER_TO_SPECIALIST / CALLBACK / NOT_INTERESTED / WRONG_PERSON / INCOMPLETE) and `eligibility_details`

## Test calls

| Call | Scenario | Result |
|---|---|---|
| T01 | Fully eligible customer | Qualified, handed off |
| T02 | Agricultural property | Stopped immediately |
| T03 | Cash income | Stopped immediately |
| T07 | Asked for 90 lakh | Explained the cap, continued with 75 lakh |
| T09 | Existing home loan on the property | Transfer specialist |
| T10 | Wants to reduce EMI (mid-call) | Transfer specialist |
| T11 | Several answers in one sentence | Asked only what was still missing |
| T12a / T12b | Busy customer, before and after a fix | Bug found, then fixed |
| T13 | Interest rate question, "what documents?", "are you a robot?" | Answered briefly, went back to the checklist |
| T16 | Joint ownership | Continued, qualified |

Details and the fixes are in [`docs/test-results.md`](docs/test-results.md).

— Saiteja Garlapati
