# Nothi — Product Requirements

**Owner:** Yasin Billah
**Status:** Specified, not built — third in the build queue
**Version:** 1.0
**Refusal policy:** [README.md](README.md) — it is the product spec, not a disclaimer

---

## 1. Problem

Bangladesh has no patient-held medical record. What a patient has instead is a drawer.

**The prescription is the only record, and it is handwritten.** A doctor writes on a pad,
the patient carries it to a pharmacy, and that piece of paper is the sole artefact of the
visit. There is no copy at the chamber, nothing in a system, nothing the patient can look
up. Lose it and the visit did not happen.

**Nobody knows what they are taking or why.** A patient can name their medicines by the
shape of the strip. They frequently cannot say what condition it treats, how long they are
meant to take it, or which doctor started it. When a second doctor asks "what are you
currently on," the honest answer is often a photograph of a strip.

**The visit pattern makes history essential and impossible.** The standard sequence is:
first visit produces a diagnosis attempt and a list of tests, often with no medicine at
all; a follow-up date is written on the prescription; the patient returns with test
reports; the doctor reads them and either starts treatment or orders more tests. The
information that matters is spread across three pieces of paper and two lab envelopes,
weeks apart, and the patient is the only integration layer.

**So the same investigations get repeated.** Tests are re-ordered because the earlier
report is lost. Medicines are re-prescribed because nobody knows what was already tried
and failed. Every specialist starts from zero.

### The insight

The record already exists — it is in the patient's drawer and their phone's camera roll.
The product is not a health platform. It is a filing system with a camera, and its most
valuable output is the two-year-old detail nobody remembers at the moment a doctor asks
for it.

## 2. Why this product is dangerous, and why that is the point

Every feature that would make Nothi more convenient makes it more dangerous:

| Tempting feature | Why it is refused |
|---|---|
| Transcribe the dosage | A misread dose delivered as clean text is worse than illegible handwriting, because handwriting makes the reader cautious |
| Offer best-guess candidates for unclear medicine names | Two guesses is still a guess; users pick one |
| Check drug interactions | Requires a complete, correct medicine list; Nothi's list is only as good as the photos it was given |
| Flag "your result is abnormal" | That is diagnosis with extra steps |
| Remind you to take your medicine | A reminder schedule is a dose |
| Share with your doctor | A permissions model on medical data, built fast, is a breach with a date on it |

Building this product well means shipping a shorter feature list than the market asks for
and being able to explain, in one sentence each, why every removed feature was removed.
That explanation is the deliverable.

## 3. Users

**Primary — the household health manager.** Usually one person in a Bangladeshi family
who holds everyone's medical paperwork: an adult managing a parent's diabetes alongside
their own visits. Already photographs prescriptions; has no system beyond a phone gallery.

**Secondary — the chronic patient.** Diabetes, hypertension, thyroid. Regular tests, a
medicine list that changes, several doctors who do not talk to each other. The person for
whom a two-year value chart is genuinely new information.

**Not a user — the doctor.** Nothi produces no clinician-facing surface in v1. A PDF
export the patient carries themselves is the only doctor-facing artefact.

## 4. Scope

### In scope, v1

| Area | Included |
|---|---|
| Prescription capture | Photo → extraction of doctor, chamber, date, medicine names, tests requested, follow-up date |
| Dosage | Stored and displayed as an image crop only. Never transcribed. |
| Unclear text | Reported as unreadable with the crop shown. No candidates offered. |
| Timeline | Visits grouped into episodes; the test → follow-up → treatment chain modelled explicitly |
| Test reports | Photo → values with the lab's printed reference range; charted over time |
| Medicine info | General purpose and what-to-watch-for, high-confidence reads only, always closing with "confirm with your pharmacist" |
| Reminders | Follow-up dates and pending tests only |
| Directory | Doctors, hospitals, diagnostic centres with publicly published fees, each dated and sourced |
| Export | Full record as PDF |
| Accounts | Single user, phone-number auth. Multiple profiles under one account (self, parent, child). |

### Out of scope, v1

Sharing of any kind, doctor-facing views, appointment booking, drug interaction checking,
diagnosis or result interpretation, medicine reminders, insurance, telemedicine.

## 5. The data model — and why the visit pattern is the hard part

Most health-record schemas assume a visit is a self-contained event. In Bangladesh it is
not; it is a link in a chain that may stay open for months. Model the chain or the product
is a photo album.

```
profile          -- self, parent, child, under one account
  id, account_id, name, dob, sex

episode          -- one health problem, spanning visits
  id, profile_id, title, opened_at, closed_at, status

visit            -- one consultation
  id, episode_id, doctor_id, chamber, visited_at,
  visit_type  ENUM(first, follow_up, emergency)
  outcome     ENUM(tests_only, medicine_only, tests_and_medicine,
                   results_reviewed_more_tests, results_reviewed_treatment_started)
  follow_up_date

prescription     -- the artefact
  id, visit_id, image_url, extraction_confidence

prescribed_item  -- one line on the prescription
  id, prescription_id,
  medicine_name        text NULL     -- NULL when unread
  name_confidence      ENUM(high, low, unread)
  dosage_crop_url      text          -- IMAGE ONLY. There is no dosage text column.
  duration_crop_url    text

requested_test
  id, visit_id, test_name, status ENUM(pending, done, reported)

test_report
  id, requested_test_id, profile_id, lab_name, reported_at, image_url

test_value
  id, test_report_id, analyte, value, unit,
  ref_low, ref_high            -- copied from the report, never from a database
  extraction_confidence
```

**The schema enforces the refusal policy.** There is no `dosage` text column. Not "we
don't populate it" — it does not exist, so no future feature can quietly start filling it.
Reference ranges come from the report image, never from a reference database, so Nothi
cannot apply a range the patient's lab did not use.

## 6. AI architecture

### 6.1 Prescription extraction

`claude-opus-5`, vision input, structured output. Confidence is per-field, not per-document.

```jsonc
{
  "doctor_name":   { "value": "Dr. ...", "confidence": "high | low | unread" },
  "chamber":       { "value": "...",     "confidence": "..." },
  "visit_date":    { "value": "2026-03-04", "confidence": "..." },
  "follow_up_date":{ "value": "2026-03-18", "confidence": "..." },
  "items": [
    {
      "medicine_name": "Omeprazole 20",
      "name_confidence": "high",
      "dosage_region":   { "page": 1, "bbox": [x, y, w, h] },
      "duration_region": { "page": 1, "bbox": [x, y, w, h] }
    },
    {
      "medicine_name": null,
      "name_confidence": "unread",
      "unread_region":   { "page": 1, "bbox": [x, y, w, h] }
    }
  ],
  "tests_requested": [{ "name": "CBC", "confidence": "high" }]
}
```

The model returns **bounding boxes for dosage, never dosage text**. The application crops
the image at those coordinates and displays the crop. The dosage never becomes a string
anywhere in the system, which means it cannot be logged, cached, summarised, or read aloud
by a future feature.

### 6.2 System prompt constraints

- Never output dosage, frequency, or duration as text. Return the region only.
- If a medicine name cannot be read with high confidence, return `null` and
  `"unread"`. Never return a partial name, a phonetic guess, or a list of candidates.
- Never correct or complete a medicine name to the nearest known drug. A name that looks
  like a known drug but is not clearly written is `unread`.
- Never infer a date that is not written.

### 6.3 Server-side enforcement

| Check | Action |
|---|---|
| Any field matches a dose pattern (`\d+\s?(mg|ml|mcg)`, `1\+0\+1`, `bd`, `tds`, `od`) outside a bbox | Record rejected, logged |
| `medicine_name` non-null with confidence `unread` | Name discarded |
| Medicine info requested for a `low`/`unread` item | Refused before reaching the API |
| Medicine info response lacks the pharmacist close | Response rejected |

The dose-pattern check is a text scan over everything the extractor returned. If a dose
string appears anywhere in a structured field, the whole extraction is thrown away rather
than sanitised — sanitising means something got through.

### 6.4 Medicine information

A second, separate call, only for `high`-confidence names, with a fixed three-part
response shape: what it is generally for, what to watch for, and the pharmacist close. The
call does not receive the patient's other medicines, their test results, or their history
— it cannot personalise, because personalisation is advice.

## 7. Directory and pricing — the honest version

The user asked for a public API of Bangladeshi doctors. **There is no reliable public API
for this.** Building the directory on a fictional integration would be the exact failure
mode these products are designed to avoid, so the plan is stated as it actually is:

| Source | What it gives | Reliability |
|---|---|---|
| BMDC registration lookup | Verification that a doctor is registered | Public but not an API; lookup by number |
| Hospital and diagnostic-centre websites | Published consultation fees, test price lists | Public, structured enough to parse, changes without notice |
| Existing directory sites | Specialist listings by area | Public, unofficial, quality varies |
| Manual seed | 200 doctors and 30 diagnostic centres in Dhaka | Slow, accurate, sufficient for v1 |

**v1 ships the manual seed.** Every fee carries a `source_url` and a `verified_on` date,
displayed with the price:

> Consultation ৳1,500 · from hospital website · checked 12 Feb 2026

A price with no date is not shown. Stale prices are marked stale rather than hidden — an
old price with a visible date is useful; an old price presented as current is a lie. The
directory sorts by fee and distance only. It never ranks by suitability for a condition,
because that is a referral, and a referral is advice.

## 8. User flows

### 8.1 Capture

Photograph → extraction (10–20s) → review screen. The review screen shows what was read
and, prominently, what was not:

```
Read
  Dr. Rahman · Popular Diagnostic, Dhanmondi · 4 March 2026
  Follow-up: 18 March 2026
  Tests: CBC, Serum Creatinine

Medicines
  Omeprazole 20        [dosage: image crop]
  Napa Extra           [dosage: image crop]
  ▲ Could not read this line — ask your pharmacist
    [image crop of the line]

Nothing on this prescription was interpreted for you. The dosage images
above are your doctor's handwriting, shown as written.
```

The user can correct any field. They cannot type a dosage — there is no field for it.

### 8.2 Timeline

Grouped by episode, not by date, so the chain reads as one story:

```
Stomach pain — opened 4 March 2026 · ongoing

  4 Mar   Dr. Rahman           tests ordered, no medicine
          → CBC, Serum Creatinine
  9 Mar   Reports uploaded     Haemoglobin 11.2 (range 12.0–15.0)
  18 Mar  Dr. Rahman           results reviewed, treatment started
          → Omeprazole 20, Napa Extra
  2 Apr   Follow-up due                                    [reminder set]
```

### 8.3 Reminders

Two kinds only: a follow-up date is approaching, and a requested test has not been marked
done. Both are logistics.

## 9. Evaluation

**Test set.** 60 real prescriptions from family and friends, consented and de-identified,
deliberately weighted toward bad handwriting. Plus 20 test reports from four different
labs with different layouts.

| Metric | Target | Why |
|---|---|---|
| Dose text leakage into any field | 0 | Single most important number in the product |
| Fabricated medicine name rate | 0 | A confident wrong name is the failure that hurts someone |
| `unread` rate on genuinely illegible lines | ≥ 95% | Under-reporting illegibility is the dangerous direction |
| False `unread` on legible lines | ≤ 20% acceptable | Over-caution is the correct direction to fail in |
| Follow-up date extraction accuracy | ≥ 90% | The reminder is the retention feature |
| Reference range copied from report, never from elsewhere | 100% | Enforced by schema |

The asymmetry in rows three and four is the whole design philosophy: a product that says
"I can't read this" too often is annoying; one that says it too rarely is harmful. Nothi
is tuned to be annoying.

## 10. Build plan

Four weekends, sequenced so the dangerous parts are proven before the pleasant parts are
built.

1. **Extraction and refusal.** Capture, extraction with bbox-only dosage, the server-side
   dose-leak check, the review screen. Run the 60-prescription eval. If dose leakage is
   not zero, the product does not proceed.
2. **The record.** Episodes, visits, timeline, the visit-outcome model.
3. **Tests and reminders.** Report extraction, value charts with printed ranges,
   follow-up and pending-test reminders.
4. **Directory and export.** Manual seed with sourced, dated prices. PDF export.

## 11. Risks

| Risk | Mitigation |
|---|---|
| A user treats "generally prescribed for" as "this is what you have" | Fixed response shape ending in the pharmacist close, every time, no exceptions |
| Bbox coordinates drift and the crop shows the wrong line | Crops are rendered back over the original in the review screen so the user sees the crop in context and can correct it |
| Someone requests dose transcription hard enough that it gets built | The schema has no column for it. Adding the feature requires a migration, which is a decision someone has to make deliberately. That is the point. |
| Directory prices go stale and mislead | Every price carries a checked-on date; undated prices are not displayed |
| Medical data breach | Single-user scope, no sharing, no third-party analytics on record screens, encrypted at rest |
| Regulatory exposure | No diagnosis, no dosing, no treatment recommendation — the three things that would make this a medical device rather than a filing cabinet |

## 12. What this proves in an interview

*"Bangladeshi prescriptions are handwritten and patients have no medical record. This
builds one from photographs — timeline, tests, follow-ups. The interesting part is what it
refuses to do: it never transcribes a dose. Dosage is stored as an image crop of the
doctor's handwriting, because a misread dose in clean text is worse than illegible
handwriting. There's no dosage text column in the schema at all, so nobody can add that
feature by accident. And when it can't read a line it says 'ask your pharmacist' — it
never offers a best guess, because a list of two guesses is still a guess and the user
will pick one."*

That answer is the same instinct as the JTBS money rules and the LifeArk mastery criteria:
identifying the point where the system must not be trusted, and designing the constraint
into the structure rather than the copy. It is a pattern across the work, not a one-off.
