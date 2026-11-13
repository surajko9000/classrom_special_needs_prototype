# Classroom Compass — "Add a Student" Flow (Standalone Context)

> **Scope:** this file covers **only** the student-addition process (drag-and-drop IEP + behavior log). It is self-contained so it can be handed to Lovable on its own. For the whole product, see `CONTEXT.md` (§6.6 is the same flow in brief).
> **This is a hackathon demo with fictional data only.** Do not build real document parsing, a backend, or real student data handling.

## 1. What to build

A **drag-and-drop "add a student" flow**. The teacher drops a student's **IEP** and **past behavior log**; the app finds the student's name in each file, matches the files to one student, lets the teacher **review and approve** what it found, then adds the student everywhere in the app.

- **One student at a time.** The demo adds a **new 4th student: Sam Taylor, shown as "Sam T."**
- **Assumption (agreed):** the IEP contains the student's name, and the behavior log contains the student's name. Match files by name.
- **Mostly dragging and dropping**, not typing. The only typing is correcting fields in the review step.
- Reading the IEP is **simulated** and must be labeled so. Reading the behavior-log CSV is **real**.

## 2. Where it starts (important)

- A **"▶ Demo: add a student"** button in the **header**, next to the Demo clock (with a small "↺ Reset" button that restores the demo data).
- The app **still opens on the Today screen.** The flow starts only when the button is pressed. Do **not** open the app on an add-student screen.
- A **drop zone at the top of the Students tab**: "Drop an IEP and a behavior log to add a student." Dropping real files there opens the same flow. The existing **＋ Add student** manual form stays for one-offs.

## 3. The flow (dialog with a 3-step stepper)

**Add files → Review → Done**

### Step 1 — Add files
- Large **drop zone** (dashed, teal tint). Also works by click, or Enter/Space, to browse. Accepts `.pdf` (IEP) and `.csv` (behavior log). Drag-over highlights it.
- **Sample chips** for the demo, fictional: `📄 IEP_Sam_Taylor.pdf` and `📊 BehaviorLog_Sam_Taylor.csv`. They can be **dragged into the drop zone or simply clicked**. Also a **"Use sample files"** button as the reliable on-stage fallback.
- Links to **download the two sample files** (served from `/samples/`) so presenters can also demo real dragging from the desktop.
- **"What the app found"** list: one selectable card per detected student, e.g.
  `Sam T. · [New student] · IEP ✓ · Behavior log ✓ (6 entries)`.
  - If the name matches someone on the roster: badge **"Already on your roster · will merge."**
  - If one file has no counterpart: say so, e.g. "No behavior log yet" (the flow can still continue).
  - If a file contains several students, show one card each and say "The demo adds one student at a time. Pick one."
- **Unusable files are listed with a plain reason**, never silently ignored. Examples: "This demo can only read the sample IEP. Try the sample files." / "Couldn't find a 'student' column." / "Not a file this demo reads. Use an IEP .pdf or a behavior log .csv."
- Primary button **Review →** (disabled until a student is detected).

### Step 2 — Review (the teacher approves everything)
Nothing is saved until the teacher clicks **Add student**.

1. **Student** card: first name, **last initial** (single letter, required), grade (9–12), and **classes with you** (checkboxes for the four teaching periods). Prefill from the files. Helper text: *"IEPs list a full name. The app shows first name and last initial."*
2. **From the IEP** card, labeled **"Simulated reading · check each item"** (amber):
   - Plan start date and next review date (date inputs).
   - Strengths (editable text).
   - **Supports and accommodations as individual checkboxes**, each with a source reference such as "IEP §4". **Only checked items are saved.** Label: "Supports and accommodations (only checked items are saved)".
3. **From the behavior log** card, labeled **"Imported · needs your OK"** (blue): each entry is a checkbox row showing date, class, `Before › Observed › After`, and an **Incident / Positive** pill. **Only checked entries are saved.** Helper: "They are marked 'Imported' and feed reminders once added."
4. If merging with an existing student, show a notice: "<Name> is already on your roster. The new plan, supports, and entries will be added to their profile."
5. Buttons: **← Back** and **Add student** (or **Add to profile** when merging). Validate first name and a single-letter initial; show an inline error otherwise.

### Step 3 — Done
Green check, "<Name> added" (or "updated"), then a confirmation checklist:
- ✓ On your roster and in the Log student picker (list their classes)
- ✓ IEP saved as a plan · next review <date>
- ✓ N approved supports on the profile
- ✓ N behavior log entries imported and marked "Imported"
- ✓ Reminders for their classes now include them, built only from the approved items

Buttons: **Open profile**, **See on Today** (moves the demo clock to the student's first class and goes to Today), **Done**.

## 4. What "adding" must change everywhere

All of this must come from the **same shared data store**, not separate hard-coded values.

- **Roster / Students tab:** new student appears; the student count on Today's class hero card updates.
- **Log tab:** the student appears in the **student picker** for each class they are enrolled in.
- **Plans:** add a plan of type **IEP**, title "Annual IEP (uploaded)", start date, next review date, status **Active**. The review date automatically feeds **Today → Needs your attention** (soonest two plan reviews).
- **IEP document row on the profile** (inside the IEP & support plans card): 📄 filename · **"Sensitive record"** pill · "Added <date> · only you can see this" · an **Open** link for the sample file.
- **Imported observations** use the existing observation model (§6). They appear in the **Recent log** and the student's **Timeline** with an **"Imported"** tag, kept in **date order** with existing entries.
- **Today reminders for the new student** are **generated** (no hand-written text): from **approved supports** + **saved observations in that class**.
  - With observations: `"6 saved observations in Math Support (Oct 8 – Nov 10); 3 describe a support working or a positive moment."` + `"Documented supports: Extra time on written tasks; Break pass for self-regulation; …"` (first three).
  - Without observations in that class: `"New to this class. Supports were approved from the plan; no observations are recorded in <class> yet."`
  - Source link text: "View N source observations" (or "Open profile" when N = 0).
  - **Never** build a reminder from an unchecked/unapproved item.
- **Merge, don't duplicate:** re-adding the same name (same first name + last initial) adds the new plan, new supports (no duplicates), and entries to the existing student.

## 5. Data shapes

```ts
Student      { id, first, li /* last initial */, grade, classes: ClassId[], strengths, supports: string[],
               plans: Plan[], doc?: { name, added, sample }, archived }
Plan         { id, type: "IEP"|"504 plan"|"Classroom support plan"|"Other", title, start, review, status: "Active"|"In review"|"Ended" }
Observation  { id, kind:"obs", type:"incident"|"positive", sid, cls /* class name */, when /* display + sortable */,
               A: string[], B: string[], C: string[], P: string[] /* support used */, note, draft:false, imported: true }
```
Class IDs / names: `P1 English Support (8:00)`, `P2 Science (9:00)`, `P3 Math Support (10:00)`, `PL Planning (11:00, no students)`, `P5 Life Skills (13:00)`.

## 6. Behavior log file format (CSV, parsed for real)

Header row (extra columns ignored; column order flexible):

```
student,date,time,class,type,antecedent,behavior,consequence,support,note
```

- `student`: full name, **required**. Convert to first name + last initial (last word's first letter): "Sam Taylor" → **Sam T.**
- `date` `YYYY-MM-DD`; `time` 24-hour `HH:MM`; `type` is `incident` or `positive`.
- `antecedent`, `behavior`, `consequence`, `support`: use the **same option wording as the Log pickers** (below). Multiple values separated by `|`.
- `class`: a class name from the schedule ("Math Support", "Science", …).
- Quoted fields (with commas) must work. Handle `\r\n` and `\n`.

**Log picker vocabulary (use these exact strings):**
- *Antecedent:* Activity interrupted, Activity denied, Activity transition, Divided attention, Given corrective feedback, Given demand / task, Long independent task, Unclear directions, Group work change, Left alone, Presence of a specific person, Redirection, Told to wait, Unexpected change.
- *Behavior (incident):* Left designated area, Property misuse, Property destruction, Verbal outburst, Work refusal, Disengaged / sleeping, Threat, Physical contact with others.
- *Behavior (positive):* Used a break card, Asked for help, Returned to task, Transitioned smoothly, Completed task, Joined group calmly.
- *Consequence / response:* Behavior ignored, Blocked, Given another activity, Given preferred item, Left alone, Offered a break, Peer remarks, Removed from activity, Reprimand, Task shortened / chunked, Verbal redirection, Positive interaction.
- *Support:* Countdown before transitions, Offer choices, Positive interaction, Proactive brief break, Reduce unstructured time, Self-monitoring check-in, Teach expected behavior, Timed work intervals, Transition support, Use of clear expectations, Visual schedule, Chunked directions, None, Other.

## 7. Simulated IEP reading

- In the demo, **only the fictional sample IEP is recognized**, matched by the name in its **filename** (`IEP_Sam_Taylor.pdf` → Sam T.). Any other IEP file shows: *"This demo can only read the sample IEP. Try the sample files."*
- Put this behind one function, e.g. `aiService.extractIep(file)`, with a code comment and UI label saying it is **simulated**. A real build would parse the document and **still require the teacher to approve each extracted item**.
- Simulated extraction result for the sample:
  - grade 10; classes Science (P2) and Math Support (P3)
  - strengths: "Strong with numbers, enjoys music, persistent once there is a clear plan."
  - plan: start **2026-02-09**, next review **2027-02-09**
  - supports: Extra time on written tasks (IEP §4); Break pass for self-regulation (IEP §4); Preferential seating near the teacher (IEP §4); Check-in at the start of class (IEP §5); Advance notice of schedule changes (IEP §5)

## 8. Sample files to create (all fictional)

**`IEP_Sam_Taylor.pdf`** — one page, banner **"SAMPLE DOCUMENT — fictional student and data, created for the Classroom Compass demo."** Sections: 1 Student information (name Sam Taylor, grade 10, IEP start Feb 9, 2026, annual review due Feb 9, 2027, case manager Morgan Lee); 2 Strengths and interests; 3 Services (Math Support Period 3, Science Period 2); **4 Accommodations and supports** (extra time on written tasks; break pass for self-regulation; preferential seating near the teacher); **5 Classroom routines** (check-in at the start of class; advance notice of schedule changes); 6 Progress reporting. **No diagnoses.**

**`BehaviorLog_Sam_Taylor.csv`** — 6 rows, about half positive:

```csv
student,date,time,class,type,antecedent,behavior,consequence,support,note
Sam Taylor,2026-11-10,10:20,Math Support,positive,Given demand / task,Asked for help,Task shortened / chunked,Proactive brief break,Asked for help on the second problem set.
Sam Taylor,2026-11-05,09:30,Science,incident,Unexpected change,Verbal outburst,Verbal redirection,Countdown before transitions,Lab groups were changed at the start of class.
Sam Taylor,2026-10-29,10:05,Math Support,positive,Long independent task,Returned to task,Offered a break,Proactive brief break,Used the break pass and came back on their own.
Sam Taylor,2026-10-22,09:15,Science,incident,Told to wait,Verbal outburst,Verbal redirection,Use of clear expectations,Waited for materials at the lab station.
Sam Taylor,2026-10-15,10:10,Math Support,positive,Group work change,Joined group calmly,Positive interaction,Offer choices,Chose a partner and started right away.
Sam Taylor,2026-10-08,09:40,Science,incident,Long independent task,Work refusal,Task shortened / chunked,Chunked directions,Reading passage was two pages long.
```

Serve both from `/samples/`. The CSV text in the app should be the same as this file.

## 9. Real-world context and decisions (why it looks like this)

**How a student normally joins a special education class:** a concern is raised; the school evaluates with parental consent; the team decides eligibility; an **IEP meeting** writes the plan (goals, services, accommodations, placement, sometimes a behavior plan); the student is enrolled and a case manager is assigned; **teachers are informed of their responsibilities under the IEP**; the plan is reviewed annually and the student re-evaluated about every three years. Transfers arrive with records from the previous school. So the **roster and IEP usually arrive first**, and the teacher builds the day-to-day picture. The app's per-student progression mirrors this: **Enrolled → IEP added → Supports approved → History imported and reviewed → Ready for class prep.** (Optional: show this as a small checklist on the profile.)

**Decisions made with the user:**
- **Full IEPs may be uploaded.** Safeguards: label as **"Sensitive record"**, visible only to the teacher; keep diagnosis text **off everyday screens** (Today, Log, Class Prep show supports and strengths only); show first name + last initial on screen; only **teacher-approved** items drive reminders. All demo IEPs are fictional. A real deployment needs district and FERPA approval.
- **Past behavior logs: yes**, as an imported **baseline** (gives "before" data, handoff context, and evidence for reminders). Guardrails: tag "Imported" with source and date; teacher approves entries first; show record age and weight recent records more in reminders; never show a total or risk label, only a timeline and summary. Import teacher observation logs and behavior-plan data; **leave out district discipline referrals** (sensitive, different framing).
- **One student at a time** for the demo. Bulk/multi-student import is a later idea, not built now.
- Students are **archived, not deleted** elsewhere in the app (record retention).

## 10. Tone, style, and accessibility

- Calm educator-utility look: off-white canvas, deep blue/teal accents, warm neutral cards, generous whitespace. Teal dashed drop zone. Amber only for "simulated / draft / sensitive" labels. No red warning treatment next to students (errors only as small inline text).
- Supportive wording: "Review," "Approved," "Documented support." Never "risk," "flag," or "problem."
- Dialog: centered, wide (~760px), scrolls internally, stepper at the top, visible **Close** button; **Esc** closes it.
- Keyboard: drop zone, sample chips, and cards are focusable and activate with Enter/Space; visible focus rings; form fields have labels.
- **Technical gotcha:** make the draggable sample chips `span`s with `role="button"` and `tabindex="0"`, and set `-webkit-user-drag: element`. Draggable `<button>` elements (especially with a CSS `all: unset` reset) may not start a drag in some browsers.
- Add a visible **"Simulated"** label wherever IEP reading appears, and keep the small "Demo uses fictional data" badge in the sidebar.

## 11. Out of scope for this flow
Real IEP/PDF parsing or OCR, multi-student bulk import, student information system (SIS) sync, persisting data beyond the page session, permissions/roles, and any prediction or risk scoring about a student.

## 12. Done when
- Header **"▶ Demo: add a student"** opens the flow from any screen; app still loads on Today.
- Dragging the two sample chips (or clicking them, or "Use sample files") into the drop zone shows the **Sam T.** card with IEP ✓ and log ✓ (6 entries).
- Review step lets the teacher uncheck supports and log entries; only checked items are saved.
- After **Add student**: Sam T. is on the roster, in the Log picker for Science and Math Support, has an IEP plan and document row, shows "Imported" entries on the timeline and Recent log, and **Today** (at 9:00 Science and 10:00 Math Support) shows a generated reminder for Sam built from approved supports and saved observations.
- Dropping a real copy of the CSV through the file picker also works; an unknown file shows a clear reason.
- Adding the same student again merges instead of duplicating.
