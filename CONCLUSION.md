# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

| run | correct | wrong | no answer | model calls | tokens | seconds |
|---|---|---|---|---|---|---|
| bare | 16 | 4 | 4 | 51 | 39536 | 100.2 |
| layer 1 | 18 | 6 | 0 | 62 | 48708 | 103.5 |
| layer 2 | 18 | 6 | 0 | 62 | 48708 | 103.6 |
| layer 3 | 18 | 4 | 2 | 74 | 60116 | 112.9 |
| layer 3 + notes | 17 | 4 | 3 | 71 | 83940 | 152.8 |

(All out of 24 questions.)

Layer 1 changed the results the most.
Before -> after (correct / wrong / no answer, out of 24): **16 / 4 / 4 -> 18 / 6 / 0**.

It changed what the agent can do, but only a little. Before, a tool error (for example a wrong region name) ended the
run, so there was no answer. With layer 1, the error goes back to the model as text and the model can try again.
That gave 2 more correct answers (16 -> 18). All 4 "no answer" cases became answers, but only 2 of them were correct;
the other 2 were wrong (wrong went from 4 to 6). So retrying helped, but it also made the agent give wrong answers where
before it gave none.

Layer 2 changed nothing: the numbers are the same as layer 1 (18 / 6 / 0, 62 model calls, 48708 tokens). The step limit
never fired in my run.

Layer 3 did not add correct answers (still 18). It only changed what the agent admits: wrong went from 6 to 4 and no
answer from 0 to 2, because it replaced two answers that were not in any tool result with "unverified". This cost 12 more
model calls (62 -> 74) and about 11400 more tokens (48708 -> 60116), and 9 more seconds.

## 2. One failure the full harness did not fix
Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

The question is **q18** (Jambyl, cpi_yoy, 2022 to 2025, "how many percentage points did it rise or fall"). With layer 3
the result is **no answer**: the final answer was UNVERIFIED. The right answer is 5.5 (the rate fell from 16.1 to 10.6).

In the trace the model went wrong at step 2. It read the data correctly (16.1 in 2022 and 10.6 in 2025), but then it
called `calculate` with `(16.1 - 10.6)/100`. A change in percentage points is only the difference, 16.1 - 10.6, so the
"/100" was not needed. It looks like the model mixed this up with the formula for percent change. The tool returned
0.055, and at step 3 the model made a second mistake: it wrote `ANSWER: 0.55 percentage points`, so it moved the decimal
point again. Layer 3 saw that 0.55 is not in any tool result and sent the model back. But the model repeated the same
`calculate` call with the same wrong expression, got 0.055 again, and after the "stop calling tools" message it wrote
0.55 again. So the harness did the right thing (it did not accept a made-up number), but it could not fix the answer: it
turned a wrong answer into "no answer".

One thing I would add: when the harness rejects an answer, it should show the model the numbers it is allowed to use,
for example "the last calculate result was 0.055, your answer 0.55 does not match", and say "do not repeat the same
call; if the expression is wrong, change it".
It would cost about one more model call for every rejected answer (around 800 to 1200 tokens and a few seconds), and a
new way to fail: if the hint points to the wrong number, the model may copy it and give a confident wrong answer that
passes the check.

## 3. Workflow or agent for the monthly ranking?
Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

I would use the **fixed workflow**.

In Part 1, on the 15 fixed questions:

| | correct | wrong | no answer | model calls | tokens | seconds |
|---|---|---|---|---|---|---|
| workflow | 13/15 | 0 | 2 | 15 | 3852 | 16.0 |
| agent | 9/15 | 2 | 4 | 26 | 19835 | 48.4 |

The workflow was better in every way. It had more correct answers and no wrong answers. It used about 5 times fewer
tokens, fewer model calls, and it was about 3 times faster.

The monthly task is always the same: the highest and the lowest region for one indicator in one year. This is exactly the
kind of task a workflow is good at. The tool already gives the regions in order, so plain code can take the first and the
last one. The workflow only failed on q21 and q22, where the question used old region names. In a monthly report the
region names are fixed, so this problem should not happen. One note: my 15 questions include lookups and rankings
together, so the numbers are an estimate for ranking, not a direct measurement.

I would choose the agent for an open question where the steps are not known in advance, for example "how much did the
rate change in percentage points" or a question where the data may not exist.

---
I wrote this file myself, without any AI tool.
