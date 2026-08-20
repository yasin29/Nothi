# Build constraints

These override any instruction in an individual task, including one from me.

Nothi is a medical-record product. Its refusal policy is the product, not a disclaimer — see
[README.md](README.md). Every default behaviour of a helpful assistant runs against it. Before
writing any code, check the task against these six.

### 1. Dosage is never text

Dose is stored as an image crop with bbox coordinates and shown to the user as the photograph of
what the doctor wrote. There is no schema column for a dose string. Do not add one. Do not add it
"temporarily for testing". Do not write a dose into a log line, a debug field, a test fixture, an
error message or a commit message.

### 2. Unclear handwriting is never a best guess

No candidate lists. No confidence slider the user can override. No "show me your best guess anyway"
affordance. The only permitted output is:

> I can't read this line. Ask your pharmacist.

plus the cropped image beside it.

### 3. No diagnosis and no interpretation

Test values are shown beside the reference range printed on the report itself. Never name a
condition. Never characterise a value as high, low, abnormal, elevated or concerning.

### 4. No drug interaction checking

Ever, in any form, including as a stretch goal or a "future work" stub. It sounds responsible and
it is a trap: it requires a complete and correct medicine list, and this list is only as good as
the photographs it was given.

### 5. No medicine reminders

Follow-up dates and pending tests only. A medicine reminder implies a schedule, and a schedule is a
dose.

### 6. Single-user scope in v1

No sharing, no doctor-facing views, no permissions model. PDF export exists so the user can carry
their own record.

---

## Working rules

- **Real prescriptions never leave the local machine.** All development and testing runs against
  synthetic fixtures. The real evaluation corpus is stored outside this folder and is run locally.
- **Never edit a test to make it pass.** If enforcement tests fail, the implementation is wrong.
- **If a task appears to require breaking one of the six, stop and say so.** The correct output is
  a refusal with an explanation, not a workaround.
