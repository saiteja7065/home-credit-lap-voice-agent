# Approach Note – Home Credit LAP Qualification Voice Agent

Saiteja Garlapati · AI Intern (Prompt Engineer) assignment, SalesAgents AI

## What's in the folder

1. System Prompt: the exact prompt running on the Retell agent
2. Call Recordings: 11 test calls (audio)
3. Call Transcripts: one per call, with the post-call outcome and the captured eligibility details at the top
4. Test Results: what I tested, what happened, and the bugs I fixed
5. This note

## How I started

Before writing anything, I turned the brief into a list of rules and a simple flow. The brief reads easily, but the hard part is that the agent has to do several things at once. It has to follow a fixed order of 7 questions, remember answers the customer gives early, stop the moment something fails, and still sound like a person on a call.

So I first listed:
- the 7 checklist items and the exact pass or fail rule for each
- the special branches: busy customer, existing loan or EMI reduction, loan above 75 lakh
- what the agent must never do: invent interest rates, ask a question that's already been answered, or say the customer qualifies before all 7 are done

## How the prompt is organised

The prompt is long on purpose, since the brief says length doesn't matter and accuracy does. It's split into clear sections so the model can find the rule it needs:

- Identity, language and gender: all the variables (company, customer, agent name, gender, date, day, time, language, RAG context, history, latest utterance) are used, with a fallback if a value is missing. In Hindi, the verb forms change with gender, so I wrote those out.
- Speaking style: short turns, one question at a time, amounts spoken the Indian way ("seventy-five lakh rupees" and not "75,00,000").
- Internal state: a silent checklist the model keeps updating, so it always knows what's answered.
- A fixed order of work for every customer turn: understand the full sentence, pull out every fact, check for a transfer trigger, check for a disqualifier, answer any side question, then ask the first item that's still missing. This order is the core of the design.
- Eligibility rules written out item by item, with common ways people say things ("khet" for farm land, months instead of years, crore to lakh conversion).
- Separate flows for disqualification, transfer, busy customer and the final handoff, each with a sample line.
- A few short example exchanges for the tricky cases.

## Decisions I had to make where the brief was silent

The brief doesn't cover everything, so these were my calls:

- **Original papers held by a bank because of a loan.** I treat this as an existing-loan case (transfer), not as "no original documents" (disqualify). The real situation is a loan transfer.
- **Market value.** No minimum is given, so the agent only records it. It never rejects anyone on value and never compares it with the loan amount.
- **Tenure range.** I read "between 3 and 15 years" as including 3 and 15.
- **Customer refuses 75 lakh.** The brief only says to offer the maximum. If they say no, the agent closes politely.
- **Unclear answers.** "I think the papers are somewhere" leaves the item unanswered, and the agent asks once to confirm. I'd rather ask one extra question than qualify or reject someone on a guess.
- **Immediate correction.** If the customer corrects themselves straight away ("sixteen… okay, fifteen"), the agent takes the corrected answer. This is a judgment call. A strict reading would reject at "sixteen", but a person correcting a slip on a live call shouldn't lose the offer for it. The prompt also tells the agent not to coach people or invite changes.
- **Retired, homemaker, or not an owner at all.** The brief only names salaried and self-employed, and sole or joint owner. The agent checks once and treats anything else as not meeting the criteria.
- **Wrong person or family member.** The agent doesn't share any offer details. It asks for a good time to call back.
- **Customer not interested.** At most one gentle line, then a polite close.
- Never asks for OTP, PAN, Aadhaar or bank details, and is honest if asked whether it's a bot.

## Testing

I tested on Retell using browser web calls and spoke to the agent myself as different kinds of customers, with natural speech: fillers, interruptions, changed answers and side questions. Details are in the Test Results document.

The useful part was that testing found real problems:
1. It re-asked market value after the customer had already given it. Fixed with a "re-check the whole conversation before every question" rule.
2. It gave up when a callback time came through garbled. Fixed with a general "if unclear, ask again" rule.
3. It sometimes said goodbye without ending the call. Made end_call mandatory with every closing line.
4. Background noise kept cutting off the agent. Adjusted Retell's interruption sensitivity, noise filtering, transcription mode and keywords, and added an end-on-silence fallback.

I kept the recording of the busy-customer call from before the fix (T12a) so the change is visible.

## Limitations I'm aware of

- Most of the remaining errors come from speech-to-text on a noisy line, not from the prompt logic. With a quieter line or a phone number in place of a browser mic, this improves.
- Two calls (T07 and T13) had noise during the goodbye, so the agent didn't get to end the call itself. The logic in both was correct up to the handoff.
- Hindi is supported in the prompt but I only tested in English.
- In a real deployment, I'd pass the callback time and the eligibility answers to the CRM through a function call rather than relying on post-call extraction.
