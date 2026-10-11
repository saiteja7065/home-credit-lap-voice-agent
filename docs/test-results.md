# Test Results – Home Credit LAP Voice Agent

- Platform: Retell AI (single prompt agent, browser web calls)
- Model: GPT 5.6 Terra | Voice: ElevenLabs "Monika" (Indian English, female)
- Test values used: agent_name = Priya, agent_gender = Female, company_name = Home Credit, customer_name = Saiteja, language_to_speak = English
- Date of testing: 11 October 2026

I made every call myself, speaking as the customer the way a real person would. I didn't read out clean scripted lines. I used fillers, interrupted the agent, changed answers and asked side questions. The recordings and transcripts for each call are in folders 2 and 3.

## Summary

| # | Scenario | What I said as the customer | Expected | What the agent did | Result |
|---|---|---|---|---|---|
| T01 | Fully eligible customer | Residential 2BHK, sole owner, originals at home, 60 lakh, salaried with bank salary, value 1 crore, 10 years | Collect all 7, then hand off | Asked all 7 in order. When "original papers" came through as "virtual papers", it asked again to confirm. Handed off and ended the call. | Pass |
| T02 | Agricultural property | "It's agricultural land in my village" | Stop immediately | Politely said the offer is only for residential, commercial or industrial property and ended the call. No further questions. | Pass |
| T03 | Cash income | Shop, sole owner, originals, 30 lakh, then "my income is mostly cash" | Stop immediately | Stopped at the income answer, gave the reason, ended the call. Market value and tenure were never asked. | Pass |
| T07 | Loan above 75 lakh | Asked for 90 lakh, accepted 75. Also said "16 years" and corrected to 15 straight away. | Explain the cap, offer 75 lakh, continue | Explained the 75 lakh limit, asked if I wanted to go ahead with 75, recorded 75 and continued. Accepted my immediate correction from 16 to 15 years. | Pass (see note 2) |
| T09 | Existing loan on property | "I already have a home loan running on this house with SBI" | Loan-transfer specialist, end call | Said a loan-transfer specialist will contact me and ended the call without any eligibility questions. Worked even though speech-to-text heard "home loan" as "hold on". | Pass |
| T10 | EMI reduction (mid-flow) | Answered property type, then said "I want to reduce my current EMI" | Loan-transfer specialist, end call | Dropped the checklist, gave the transfer message, ended the call. | Pass |
| T11 | Out-of-order answers | In one sentence: "residential house, jointly owned with my wife, worth 1.2 crore, need 60 lakhs" | Note everything, ask only what is missing | Went straight to the documents question. Did not ask ownership, amount or market value again. Jumped from income straight to tenure. | Pass |
| T12a | Busy customer (first version of the prompt) | "I'm in a meeting", then a callback time that speech-to-text garbled | Ask for callback time, end call | Did not pitch the offer, which was correct. But it gave up on the garbled time and never triggered end_call. | Fail (bug found, see below) |
| T12b | Busy customer (after the fix) | Same as T12a | Same | No pitch. Asked again when the audio was unclear, confirmed "tomorrow, Monday, around 6 PM" and ended the call itself. | Pass |
| T13 | Diversions and interruptions | Interrupted to ask the interest rate, asked "what documents do you need?", asked "are you a robot?" | Answer briefly, don't invent anything, go back to the checklist | Said the senior loan expert will share the exact rate (no number made up). Explained documents briefly and asked the documents question again. Said honestly it is a virtual assistant and went back to the loan amount. All 7 collected, then handed off. | Pass (see note 2) |
| T16 | Joint ownership | "Jointly owned with my wife" | Continue, joint is allowed | Continued normally and qualified. Also clarified "unsalaried" by asking whether I was self-employed. | Pass |

Required scenarios covered: verification, busy and callback, offer pitch, all 7 data points, out-of-order capture, both disqualifiers named in the brief (agricultural and cash), the above-75-lakh rule, both transfer triggers (existing loan and EMI reduction), diversions, and the handoff gate.

## Bugs I found during testing and what I changed

| # | Found in | Problem | Fix |
|---|---|---|---|
| 1 | Text test before the voice calls | Customer said "worth about 1.2 crore" early on, but the agent still asked for market value later | Added a rule to re-check the whole conversation before every question, with a worked example where market value is skipped. Confirmed fixed in T11. |
| 2 | T12a | When the callback time came through garbled, the agent said "we'll try you later" instead of asking again | Added a general rule: if an answer is unclear or doesn't make sense, ask the customer to repeat. Never guess, never give up. Confirmed in T12b, T13 and others. |
| 3 | T12a | Agent said goodbye but did not call end_call, so the customer had to hang up | Made end_call mandatory in the same turn as every closing line. Confirmed in T12b, T10, T11. |
| 4 | T07, T13 | Background noise on my side kept cutting off the agent's goodbye, so end_call never fired | Platform settings: interruption sensitivity 0.9 → 0.49, noise removal now also filters background speech, transcription set to accuracy mode, boosted keywords (lakh, crore, EMI, salaried, agricultural, kirana, home loan), and end call on silence after 25 seconds as a safety net. |

## Notes

1. Speech-to-text was the biggest source of trouble, not the prompt logic. The transcripts show things like "salad" for salaried, "kid runners" for kirana and "hold on" for home loan. I've left these as they are so you can see how the agent coped with them.
2. In T07 and T13 the business logic finished correctly (all items collected, handoff said), but noise during the goodbye stopped the agent from ending the call. That's why the post-call tag for T07 says INCOMPLETE. Bug 4 above is the fix.
3. The T12a recording is included on purpose, to show the bug before the fix.
4. Hindi is handled in the prompt (language_to_speak, including female verb forms like "bol rahi hoon"), but I only voice-tested in English.
