# Nothi — your medical record, from the prescriptions you already have

নথি (nothi) means "record" or "file."

Photograph a prescription. Nothi reads what it can, files it on a timeline, remembers the
tests you were told to do, and reminds you about the follow-up. Over years it becomes the
health history you were never given — because in Bangladesh nobody hands you one.

It explains what a medicine is generally for. It never tells you what to take.

---

## Refusal policy

This is the entire product. Read this before anything else.

### 1. Nothi never states, adjusts, confirms, or infers a dose

Not "take one twice daily." Not "the usual dose is 500mg." Not "that looks like the
standard dose." Not even reading a dose back to you as a confirmation, because a
confirmation of a misread dose is a misread dose delivered with confidence.

Dosage text on a prescription is stored as an **image crop**, shown to the user as the
photograph of what the doctor wrote, and never transcribed into text by the model. The
user reads their own doctor's handwriting; Nothi does not stand between them and it.

This is the single hardest constraint in the product and it is not negotiable for
convenience. A transcription feature for dosage would be the most requested feature and
the one that turns a helpful product into a dangerous one.

### 2. Unclear handwriting is never a best guess

When a line cannot be read with confidence, Nothi says exactly this and nothing else:

> "I can't read this line. Ask your pharmacist."

It does not offer candidates. It does not say "this might be Napa or Napa Extra." A list
of two guesses is still a guess, and the user will pick one. The line is shown as a
cropped image beside the message so the user can take it to a pharmacist directly.

Confidence is not a slider the user can override. There is no "show me your best guess
anyway" button. That button is the whole risk.

### 3. What Nothi will say about a medicine

Only this, and only for medicines it read with high confidence:

- **What it is generally for** — "Omeprazole is generally prescribed for acid reflux and
  stomach ulcers."
- **What to watch for** — common side effects, and the specific signs that mean call
  someone.
- **The close, every time** — "Confirm with your pharmacist or the doctor who prescribed
  this."

It will not say whether the medicine is right for you, whether it interacts with your
other medicines, whether you should stop taking it, or whether the doctor made a mistake.
Interaction checking in particular sounds responsible and is a trap: it requires a
complete and correct medicine list, and Nothi's list is only as good as the photographs
it was given.

### 4. Nothi never diagnoses, and never reads a test result as a verdict

Test reports are stored, their values are extracted and charted over time, and the
reference range printed on the report is shown beside each value. That is the boundary.

Nothi will say "your haemoglobin was 11.2 on 3 March; the report's reference range is
12.0–15.0." It will not say "you are anaemic." Showing a value against the range the lab
itself printed is data presentation. Naming the condition is diagnosis, and it belongs to
the doctor at the follow-up appointment Nothi is reminding you about.

### 5. Reminders are logistics, never medical instruction

Nothi reminds you that a follow-up date was written on a prescription, and that a test was
requested. It does not remind you to take medicine, because a medicine reminder implies a
schedule, and a schedule is a dose.

### 6. Your record is yours

Medical history is the most sensitive data any of these three products touch. Everything
is scoped to one authenticated account. No sharing in v1 — not with family, not with
doctors — because a sharing feature is a permissions model, and a permissions model built
in a hurry on medical data is a breach waiting for a date. Export to PDF exists so the
user can carry their own record to a doctor themselves.

---

## What it does

**Capture.** Photograph a prescription. Nothi extracts what it can read confidently:
doctor name, hospital or chamber, date, medicine names, tests requested, follow-up date.
Dosage stays as an image crop.

**Timeline.** Every visit in order. What you went in with, what was prescribed, what tests
were ordered, what came back, when the follow-up was. The thing you cannot reconstruct
from a shoebox of paper two years later.

**The Bangladeshi visit pattern, modelled properly.** The system understands that a first
visit often produces tests and no medicine, that the follow-up is where the results are
read and treatment starts, and that the follow-up can produce more tests instead of an
answer. An episode in Nothi is a chain of visits, not a row in a table.

**Test reports.** Photograph the report; values are extracted into a chart over time,
always beside the lab's own printed reference range.

**Reminders.** Follow-up dates and pending tests. Nothing else.

**Directory.** Search for a specialist, hospital, or diagnostic centre, with published
consultation fees, hospital charges, and test prices where they are publicly available and
dated.

## What it does not do

- Does not transcribe, state, or confirm dosage
- Does not guess at unclear handwriting
- Does not diagnose, or interpret a test result beyond the printed reference range
- Does not check drug interactions
- Does not remind you to take medicine
- Does not share your record with anyone, including doctors, in v1
- Does not recommend a doctor — the directory sorts by fee and location, never by
  suitability for your condition

## Status

Specified, not built. See [PRD.md](PRD.md). This is the third build in the queue, and the
first one talked about in interviews — the constraint reasoning is worth more than the
code.
