# Classroom Compass — Full Project Context for Lovable

> **How to use this file:** Drop it in the repo root and tell Lovable: *"Read CONTEXT.md and build the app it describes. Match the reference prototype (`index.html`) for layout, wording, and behavior."*
> It combines the original product brief with every design decision made while prototyping. Where this file and the original brief differ, **this file wins**.

- **Working name:** Classroom Compass (placeholder, easy to rename)
- **Stage:** Hackathon prototype / product validation. A **demo, not a production student-data system.**
- **Reference prototype:** `index.html` in this repo is a single-file, working HTML mock-up of the target look, feel, copy, data, and interactions. Treat it as the visual and behavioral spec. Rebuild it properly as a React app; do not just paste it in.

---

## 1. Problem and origin

A special education teacher described these pain points:

- Behavioral incident reports are fragmented and hard to consolidate.
- Before parent conferences, IEP meetings, and support-plan reviews, teachers need to see patterns and whether an intervention helped.
- Incoming teachers often don't get a clear, usable summary of a student's past supports and behavioral context.
- Teachers need **actionable information before a class starts**, not only retrospective reports after an incident.

**Key insight:** an incident-logging tool is useful; a **time-aware daily teaching assistant that helps prepare supports *before* an activity** is more compelling. Logging and analytics exist to power that daily workflow.

Example (fictional): "At 10:00 AM during reading, John threw a book toward Kate after he struggled with the assignment. The teacher offered a break and shortened the task." The app should structure this, keep its context, and make it available for later analysis. **Never claim the antecedent proves the student's intent or motivation.**

## 2. Product proposition

**Classroom Compass is an AI-assisted daily classroom support and behavioral-observation tool for special education teachers.** It turns teacher-approved observations and existing supports into relevant, time-sensitive **preparation reminders**, simplifies logging, and makes progress over time easier to understand.

- **Primary user:** high-school special education teacher (resource or self-contained classes, may coordinate paraprofessionals).
- **Future secondary users:** authorized aides, case managers, incoming teachers.

### Jobs to be done
1. **Right now:** see my current/next class and the supports to prepare.
2. **Before class:** review student-specific, evidence-linked strategies and make a checklist.
3. **During/after class:** record an observation in seconds, mostly by tapping, without a long form.
4. **Over time:** understand patterns by activity, time, antecedent, and response; see what happened after an intervention began.
5. **At meetings / year transition:** generate a grounded, reviewable summary with source links.

### Positioning and tone (important)
- Do **not** predict that a child "will crash out" or assign any risk score.
- Offer **contextual support reminders**, e.g. *"Three prior observations during independent science reading mention task difficulty. Shorter segments and written directions are documented supports. Review before this activity."*
- Supportive vocabulary: *Preparation reminder, Documented support, Observation, Progress, Review.*
- **No** red warning icons or "high risk" badges next to students. No danger color-coding of children.
- Link suggestions are **observational associations, not causal proof.** Say so in the UI.

## 3. Stack and build guidance (Lovable)

- React + TypeScript + Vite, Tailwind CSS, shadcn/ui components, Recharts for charts (this is Lovable's default stack).
- **No backend, no auth, no database, no external API.** Keep all state client-side (in-memory + optional `localStorage`) with synthetic data. Add a "Reset demo" control that restores seed data.
- Put all AI-style behavior behind one module, `src/services/aiService.ts`, with deterministic functions (see §10). Label everything simulated.
- Suggested structure: `src/data/` (seed + types), `src/state/` (store: students, entries, clock), `src/services/aiService.ts`, `src/pages/{Today,Log,ClassPrep,Students,Insights}.tsx`, `src/components/` (shared UI).
- Accessibility: keyboard-operable everything (tiles/rows/pills/list items focusable and activatable with Enter/Space), visible focus rings, high contrast, labels on form fields, Esc closes dialogs.
- Responsive enough for a tablet/laptop in a classroom. Mobile is a stretch goal.

## 4. Visual design

Modern educator utility: calm, credible, **not** neon-gradient "AI SaaS".

- Light theme, off-white canvas, understated deep-blue/teal accents, warm neutral cards, generous whitespace, restrained typography.
- Reference palette (from the prototype): background `#f7f5f0`, card `#fffdf9`, border `#e7e2d8`, ink `#1f2a37`, muted `#6b7280`, blue `#1e4a6e`, teal `#2a8a8a`, amber `#b7791f` (drafts/notes only), green `#3b7d5a` (positive).
- Left sidebar with five tabs + a persistent **"＋ Log observation"** button. Sticky top header with date/time and the **Demo clock** (see §5).
- A small persistent **"Demo uses fictional data"** badge in the sidebar.
- Today's hero card is a blue→teal gradient; everything else is quiet cards. Today prioritizes the current class and prep **above** charts.

## 5. Demo clock (critical for the live demo)

The app must never depend on the viewer's real time. A **Demo clock** lives in the header:

- Fixed demo date: **Thursday, November 12, 2026.** Default time **9:00 AM → Period 2 Science.**
- Preset chips: **8:00, 9:00, 10:00, 11:00, 13:00**; a **"Next class ▸"** button (jumps to the start of the next class, wrapping to 8:00); a **▶ Play / ❚❚ Pause** that auto-steps to the next class every ~2.5 s so the countdown and "now" class change on their own.
- Changing the clock updates: header date line, Today (current/next class, countdown, status labels, reminders, greeting "Good morning/afternoon"), and Log (auto class + time, student choices).
- Anything saved while demoing is timestamped with the **simulated** clock, not real time.
- Future idea (not built): day picker for other weekdays/weeks.

## 6. Tabs / information architecture

Five tabs: **Today, Log, Class Prep, Students, Insights.**

### 6.1 Today (home, the centerpiece)
- Greeting + a **class strip** of the five periods (Period 1 English Support, Period 2 Science, Period 3 Math Support, Planning, Period 5 Life Skills). Each pill shows name and time; "now" has a dot; past classes are dimmed; the selected pill is highlighted. **Clicking a pill lets the teacher browse ahead/back** without changing the clock.
- **Hero card** for the selected class: status pill (*In session now / Up next / Later today / Earlier today*), course + period, time, room, number of students (computed from enrollment), planned activity, a countdown (*ends in X min / starts in X h Y min / Done*), "Next: …" line, and buttons **Open class prep** and **View roster**.
- **Preparation reminders** for the selected class (see §8). Each reminder card: student avatar + name + activity context, evidence sentence with a **"View N source observations"** link (opens that student's profile), a **Documented support** line, and buttons **Add to prep** (toggles to "✓ Added to prep") and **Dismiss for today**.
- **Planning period (11:00)** has no students: instead of reminders show "Good uses of planning time" cards: finish N drafts (→ Log), preview the next class, prepare for the next IEP review (→ student profile).
- **Needs your attention** card (right column, keep it; it fills the space and is meaningful), fully **live**:
  - "N draft observation(s)" with each draft listed (student · time · awaiting review) → opens Log. If none: "No drafts waiting."
  - The **two soonest upcoming plan reviews** computed from students' plans (IEP → "IEP review · Alex M. · Tue, Nov 17"; support plan → "Plan check-in · John D."). Click → student profile.
- **This week** card: observation count (**starts at 4, +1 per saved observation**), positives vs. incidents, a tiny sparkline, "Details" → Insights. Keep it small; no giant dashboard.
- Disclaimer line under reminders: reminders come from teacher-recorded observations and approved supports; associations not causes; not predictions.

### 6.2 Log (quick observation entry, tap-first)
Design goal: **logging must be faster than typing.** Modeled on a real tap-to-choose app: choose A / B / C / Proactive Strategy from lists, with quick-fill shortcuts for common incidents. The teacher can **also** write a note in their own words.

Layout: two columns — **form (left)**, **Recent log (right)**.

Form:
1. **Incident / Positive progress** toggle (top right). Positive mode swaps the Behavior list and quick fills. Positive logging is first-class (successful coping, break card use, smooth transitions).
2. **Class (auto)** and **Time (editable)** — auto-populated from the demo clock. **Student** pills show only students enrolled in the current class (fallback: all active students, e.g., during Planning).
3. **Quick fill** chips (shortcuts, not facts; editable after tapping):
   - Incident: *Long task → Work refusal → Break*; *Transition → Left area*; *Unclear directions → Property misuse*.
   - Positive: *Break card → Returned to task*; *Transition went smoothly*.
4. **Four compact rows** (not big squares — the teacher disliked large tiles): **A Before (Antecedent)**, **B Observed (Behavior)**, **C After (Consequence/response)**, **✦ Support (Proactive strategy)**. Each row shows chosen options as tags (or "Not set") and a **＋ Choose** button that opens a **right-side picker sheet**: title, search box, multi-select options with check circles, optional "Description" input, **Done**. Back arrow closes.
5. **Quick note — in your own words (optional)**: large textarea. A mic icon is shown **visual-only** (tooltip says voice input isn't active); do not imply it works.
6. **Intensity** (Low/Medium/High) and **Outcome** (Re-engaged/Partly/Ended activity) segmented controls (optional, toggleable).
7. **Draft bar** (dashed amber outline): "Draft — review required", one-line summary, and buttons **Clear**, **Save as draft**, **Save observation**. Both save buttons are disabled until at least a Behavior or a note exists.
8. Footnote: quick fills are shortcuts, not facts; unset fields are saved as **"Not documented"**; "Not finished? Save as draft and it will wait in your log and on Today."

**Save vs. draft behavior**
- **Save observation** → prepends to Recent log (highlighted briefly), adds to the student's profile timeline, increments the weekly count, shows a toast.
- **Save as draft** → saved with an amber **Draft** tag; appears in Recent log (with **Resume** / **Discard**) **and** in Today's *Needs your attention*. **Resume** reloads the full form (type, A/B/C/Support, note, intensity, outcome, student). Saving a resumed draft replaces it.
- Never invent missing facts; teacher verification is required before final save.

**Recent log (right column):** newest first. Filters: **All / Incidents / Positive / Drafts**, and **Everyone / each student**. Each entry: colored dot (blue incident, green positive, amber draft), student · class, time, `Before › Observed › After` chips ("Not documented" when empty), and the teacher's note in quotes. Link "View full history in Students →".

**Option lists** (neutral, non-diagnostic wording; derived from the reference app):
- **Antecedent:** Activity interrupted, Activity denied, Activity transition, Divided attention, Given corrective feedback, Given demand / task, Long independent task, Unclear directions, Group work change, Left alone, Presence of a specific person, Redirection, Told to wait, Unexpected change.
- **Behavior (incident):** Left designated area, Property misuse, Property destruction, Verbal outburst, Work refusal, Disengaged / sleeping, Threat, Physical contact with others. *(Deliberately softened from "Physical/Verbal aggression".)*
- **Behavior (positive):** Used a break card, Asked for help, Returned to task, Transitioned smoothly, Completed task, Joined group calmly.
- **Consequence / response:** Behavior ignored, Blocked, Given another activity, Given preferred item, Left alone, Offered a break, Peer remarks, Removed from activity, Reprimand, Task shortened / chunked, Verbal redirection, Positive interaction.
- **Support (proactive strategy):** Countdown before transitions, Offer choices, Positive interaction, Proactive brief break, Reduce unstructured time, Self-monitoring check-in, Teach expected behavior, Timed work intervals, Transition support, Use of clear expectations, Visual schedule, Chunked directions, None, Other.

*Optional AI feature:* a **"Structure my note"** action that turns the free-text note into a pre-filled editable draft (see §10). If included, it must flag uncertain/missing fields and never invent facts.

### 6.3 Class Prep
- Select class and planned activity (e.g., "Independent reading → group lab").
- **Prep checklist** (check/uncheck, add a custom note). Example items: Display the visual agenda; Prepare a shorter, chunked reading option; Set out break-option cards; Review transition routine into group lab. Each shows which student/support it came from and links to source observations.
- **What to watch for in this activity** (about the activity, not labeling the student), **What worked before** (with dated source links), and **Student supports in this class**.
- Banner: *these are observational associations, not proof of cause.*
- Should feel valuable even without AI: a clean consolidated prep briefing. Items added from Today's **Add to prep** should appear here.
- *Prototype status:* the reference page is static for Science only. In the real build, drive it from the selected class and the same data as Today.

### 6.4 Students
- **Left:** searchable roster (name, grade, class short names). **＋ Add student** button. **Archived (N)** list with **Restore**.
- **Right (selected student):**
  - Header: avatar, name, grade, classes; **Edit** and **Archive** (confirm dialog; archive hides them from rosters/pickers but **keeps observations**; restorable). *We chose Archive over Delete because school records have retention rules.*
  - **Strengths** and **Approved supports** (tags).
  - **IEP & support plans** card: list of plans (title, type, start date, next review, status pill Active / In review / Ended) with **＋ Add plan**, **Edit**, **Remove** (confirm). Helper text: *"A summary for classroom use. The official plan lives in your district system."*
  - **Timeline:** every saved observation for the student (and drafts, tagged), plus an **intervention/support-plan start marker** (amber) such as "Support plan started: Chunk reading instructions + planned break options — Mon Oct 5".
  - **Handoff briefing** (marked **Draft — review required**): *What has helped*, *Settings that need preparation*, *What has improved*, with "Based on N saved observations. Review before sharing." Students with no briefing data show "Not enough observations yet."
- **Add/Edit student form:** first name, **last initial (single letter)**, grade (9–12), classes (checkboxes for the four teaching periods), strengths (text), approved supports (comma-separated). Validate first name + single-letter initial. Note in form: *the app shows first name + last initial only.* (IEPs themselves may be uploaded; see §6.6.)
- **Add/Edit plan form:** type (IEP, 504 plan, Classroom support plan, Other), title (required), start date, next review date, status.
- Added students appear immediately in Log pickers, Today rosters/counts, and filters.
- **Never** reduce a student to a "problem score," diagnosis, or incident count.

### 6.5 Insights (keep it basic for the hackathon)
- **Observations per week** stacked bars (incidents vs. positive/"support worked") over ~8 weeks with a dashed **intervention marker at Oct 5**. Caption: *Recorded incidents were lower after Oct 5. This does not show that the intervention caused the change.* (Descriptive only.)
- **By classroom activity** horizontal bars (Independent reading, Transitions, Group work, Hands-on lab).
- Filters by student and class (visual in prototype; wire them in the real build if time permits).
- **Draft meeting summary** panel: *Draft — review required*, editable text, source links, **Edit** and **Print view** buttons, labeled simulated. Example: "Since the 'Chunk reading + break options' support began on Oct 5, recorded incidents during independent reading decreased from 9 to 3, and 6 notes describe re-engagement after a structured break."
- Every AI-style conclusion must link to source records. Compute chart numbers from the seed observations (don't hard-code unrelated values).

### 6.6 Adding a student: drag-and-drop IEP + behavior log (demo flow)

**Goal:** adding a student should be mostly **drag and drop**, not typing. **Build this as the demo's "add a student" flow; one student at a time.**

**Assumptions (agreed):** the IEP contains the student's name, and the behavior log contains the student's name. The app uses the name in each file to match the files to one student.

**Entry points**
- A **"▶ Demo: add a student"** button in the header, grouped with the Demo clock (and a "Reset demo" control). The app still opens on **Today**; the flow only starts when this button is pressed. Do **not** open the app on an add-student screen.
- A **drop zone at the top of the Students tab** ("Drop an IEP and a behavior log to add a student"), plus the existing **＋ Add student** form for manual one-offs.

**Three steps (dialog with a stepper): Add files → Review → Done**
1. **Add files.** Large drop zone (also click or Enter to browse). Two **sample chips** (fictional) can be dragged into the zone **or clicked**, plus a **"Use sample files"** button as a reliable fallback on stage. Below, show **"What the app found"**: a card per detected student, e.g. *"Sam T. · New student · IEP ✓ · Behavior log ✓ (6 entries)"*. If the name matches someone already on the roster, show *"Already on your roster · will merge."* Files the demo can't use are listed with a plain reason (never silently ignored). If a file contains several students, show one card each and let the teacher pick one ("the demo adds one student at a time").
2. **Review (the teacher approves everything).**
   - *Student:* first name, last initial (prefilled from the file's full name, e.g. "Sam Taylor" → Sam T.), grade, and the classes they are in (checkboxes).
   - *From the IEP* (labeled **"Simulated reading · check each item"**): plan start date and next review date, strengths, and the proposed **supports and accommodations as individual checkboxes**, each with a source reference (e.g. "IEP §4"). Only checked items are saved.
   - *From the behavior log* (labeled **"Imported · needs your OK"**): each entry as a checkbox row (date, class, Before › Observed › After, Incident/Positive). Only checked entries are saved.
3. **Done.** Confirmation checklist: on the roster and in the Log student picker; IEP saved as a plan with its review date; N supports approved; N log entries imported; reminders for their classes now include them. Buttons: **Open profile**, **See on Today** (moves the demo clock to their first class), **Done**.

**What "adding" changes everywhere (must be derived from the same store)**
- Roster, class student counts, Log student picker (by class), and Students filters include the new student.
- The student gets a **plan** (type IEP, title "Annual IEP (uploaded)", start, next review, Active). The upload date appears on **Needs your attention** automatically via the plan review date.
- The student's profile shows an **IEP document row**: filename, a **"Sensitive record"** label, "Added <date> · only you can see this," and an Open link for the sample file.
- Imported behavior entries use the **existing observation model** (§8): `type` incident/positive, `studentId`, class, `occurredAt`, `antecedent[]`, `behavior[]`, `consequence[]`, `support[]`, `note`, `draft=false`, plus an `imported=true` flag shown as an **"Imported"** tag in the Recent log and the student timeline. Entries are kept in date order.
- **Reminders for the new student** are generated only from **approved supports** and **saved observations** in that class (e.g., "6 saved observations in Math Support (Oct 8 – Nov 10); N describe a support working…" + "Documented supports: …"). If there are no observations in a class, say so ("New to this class. Supports were approved from the plan; no observations recorded here yet."). Never reminders from unapproved items.

**Behavior log file format (CSV; parsed for real in the demo).** Header row with these columns (extra columns ignored):
`student,date,time,class,type,antecedent,behavior,consequence,support,note`
- `student`: full name (required; if the column is missing, show "Couldn't find a 'student' column").
- `date` is `YYYY-MM-DD`; `time` is 24-hour `HH:MM`; `type` is `incident` or `positive`.
- `antecedent`, `behavior`, `consequence`, `support` use the **same option vocabulary as the Log pickers** (§6.2); multiple values are separated by `|`.
- `class` is a class name from the schedule (e.g., "Math Support", "Science").

**IEP reading is simulated.** In the demo only the fictional sample IEP (**IEP_Sam_Taylor.pdf**) is recognized, matched by the name in the filename; any other IEP shows "This demo can only read the sample IEP. Try the sample files." Label the reading step **simulated** in the UI and in code (`aiService.extractIep`). A real build would parse the document and still require teacher approval of each item.

**Sample data to create (all fictional).** Student **Sam Taylor → shown as "Sam T."**, grade 10, classes Science (P2) and Math Support (P3).
- *Sample IEP (PDF):* a one-page banner "SAMPLE DOCUMENT, fictional"; sections: student info (name, grade, IEP start Feb 9, 2026, annual review Feb 9, 2027), strengths ("strong with numbers, enjoys music, persistent once there is a clear plan"), services, **accommodations and supports** (§4: extra time on written tasks; break pass for self-regulation; preferential seating near the teacher), **classroom routines** (§5: check-in at the start of class; advance notice of schedule changes), progress reporting. **No diagnoses** in the sample.
- *Sample behavior log (CSV, 6 rows, about half positive):* e.g., Nov 10 Math Support positive (Given demand / task › Asked for help › Task shortened / chunked); Nov 5 Science incident (Unexpected change › Verbal outburst › Verbal redirection; "Lab groups were changed at the start of class."); Oct 29 Math Support positive (Long independent task › Returned to task › Offered a break); Oct 22 Science incident (Told to wait › Verbal outburst › Verbal redirection); Oct 15 Math Support positive (Group work change › Joined group calmly › Positive interaction); Oct 8 Science incident (Long independent task › Work refusal › Task shortened / chunked).
- Provide both files as downloads in the app (`/samples/`) so presenters can also demo real dragging from the desktop.

**Edge cases to handle**
- Re-adding the same student merges (adds the plan, new supports without duplicates, and entries); it does not create a duplicate.
- Last initial must be a single letter; first name required.
- Unreadable files are listed with a reason; the flow can still continue with whatever files were usable.
- Esc and a visible **Close** button exit the dialog; the drop zone and sample chips are keyboard-operable (Enter/Space).
- **Technical note:** make the draggable sample chips `span`s (not `button`s) and set `-webkit-user-drag: element` on them; some browsers won't start a drag from a reset (`all: unset`) button.

**Real-world context (why the flow looks like this).** In schools a student usually arrives like this: a concern is raised, the school evaluates with parental consent, the team decides eligibility, an **IEP meeting** writes the plan (goals, services, accommodations, placement, sometimes a behavior plan), the student is enrolled and a case manager is assigned, **teachers are told their responsibilities under the IEP**, and the team reviews annually and re-evaluates about every three years. Transfers arrive with records from the old school. So the **roster and the IEP usually arrive first**, and the teacher builds the day-to-day picture; our flow mirrors that: **Enrolled → IEP added → Supports approved → History imported and reviewed → Ready for class prep.**

**Past behavior records: decision and guardrails.** **Yes, import past behavior logs** as a **baseline** (gives "before" data for charts, context for handoffs, and evidence for reminders). Guardrails: tag as **Imported** with source and date; the teacher approves entries before they count; show how old records are and weight recent records more in reminders; never show a total or risk label, only a timeline and summary. Import teacher-observation logs and behavior-plan data; **leave out district discipline referrals** (sensitive, different framing).

**Open questions for Lovable/the team**
- Should a larger **multi-student import** (several IEPs and one shared log at once) be added later? Not for the demo.
- Should the review step show a per-student **setup checklist** (Enrolled → IEP → Supports approved → History reviewed → Ready) on the profile? Recommended if time permits.

## 7. Scope decisions made with the user

| Topic | Decision |
|---|---|
| Schedule | **Same five periods every day** (no per-day variation). |
| Demo date | **Fixed: Thu Nov 12, 2026**; seed history ends Wed Nov 11. |
| Roster size | Small and focused: **3 core students** (John D., Alex M., Maya R.), more can be added via the UI. (The original brief suggested 8–12; the user chose to keep it small.) |
| Insights | **Super basic** for now. |
| Meeting summary vs. handoff briefing | Two views of the same editable, source-linked panel. |
| PDF/export | **Out of scope** (no real PDF). A print-friendly view is optional. |
| Needs-attention panel | **Keep**, make it live (drafts + upcoming reviews). |
| Class switching on Today | **Yes** — pill strip to browse classes + demo clock. |
| Draft observations | **Yes** — "Save as draft" feeds Log and Today. |
| Student management | **Yes** — add / edit / archive students; add / edit / remove multiple plans per student. |
| Adding students | **Drag-and-drop IEP + behavior log**, started from a **"▶ Demo: add a student"** header button (and a Students-tab drop zone). **One student at a time**, adds a **4th student (Sam T.)**. See §6.6. |
| IEP documents | **OK to upload full IEPs**; marked Sensitive record; diagnoses kept off everyday screens; teacher approves each extracted support. |
| Past behavior logs | **Yes**, as an imported baseline in the **existing observation format**; teacher reviews before they feed reminders. |
| Log style | Tap-first with quick fills **plus** own-words note and a recent log; **no big square tiles**. |

## 8. Seed data (ALL FICTIONAL)

**Teacher:** Morgan Lee, high-school special education teacher. Demo date Thu Nov 12, 2026.

**Schedule (every day):**

| ID | Class | Time | Room | Planned activity |
|---|---|---|---|---|
| P1 | Period 1 · English Support | 8:00–8:50 | 108 | Paragraph writing workshop |
| P2 | Period 2 · Science | 9:00–9:50 | 214 | Independent reading, then group lab |
| P3 | Period 3 · Math Support | 10:00–10:50 | 112 | Fractions practice in small groups |
| PL | Planning | 11:00–11:50 | 214 | No students scheduled |
| P5 | Period 5 · Life Skills | 13:00–13:50 | 120 | Following a multi-step recipe card |

**Students**

| ID | Name | Grade | Classes | Strengths | Approved supports |
|---|---|---|---|---|---|
| s1 | John D. | 10 | P2, P3, P5 | Hands-on labs, helps peers with tools, strong verbal reasoning | Chunked reading; Clear written directions; Planned break options |
| s2 | Alex M. | 11 | P1, P2, P5 | Strong visual memory, takes pride in routines, kind with peers | Visual schedule; Transition warnings |
| s3 | Maya R. | 9 | P1, P2, P3, P5 | Creative, thoughtful writer, supports classmates in groups once settled | Quiet workspace; Advance notice of group-work changes |

Use first name + last initial only. No photos, no real identifiers, **no diagnoses**.

**Plans**
- John: IEP "Annual IEP" (start 2026-03-12, review 2027-03-12, Active); **Classroom support plan** "Chunk reading instructions + planned break options" (start **2026-10-05**, review 2026-11-19, Active).
- Alex: IEP "Annual IEP" (start 2025-11-17, review **2026-11-17**, In review).
- Maya: IEP "Annual IEP" (start 2026-01-20, review 2027-01-20, Active).

**Student stories (keep consistent)**
- **John:** historical observations cluster in lengthy independent reading; after the Oct 5 support plan, recorded incidents decline and notes show re-engagement after structured breaks (descriptive only; never say the plan caused it).
- **Alex:** benefits from visual schedules and transition warnings; include both smooth and challenging transitions.
- **Maya:** benefits from a quiet workspace and advance notice of group changes; include clear strengths and positive progress.

**Preparation reminders per class** (must be *derived from / consistent with* the observations; each cites a count and links to sources):
- **P1 English Support:** Alex — transition from writing to share-out (smooth in 4 of 6 notes with a visual cue, Oct 15–Nov 10; show visual schedule + 5-min warning). Maya — writing workshop (two notes: quiet corner helped, Oct 20 & Nov 3; offer quiet seat, preview partner changes).
- **P2 Science:** John — independent reading (three prior observations of task difficulty: Oct 8, Oct 22, Nov 3; two recent notes of re-engagement after a structured break; chunk reading, written directions, have breaks ready; 5 sources). Alex — transition to group lab (same transition evidence; 6 sources).
- **P3 Math Support:** John — fraction problem sets (chunked sets and break cards in 3 recent notes; on Nov 11 used a break card on his own, back at work in ~5 min; 3 sources). Maya — small-group practice (advance notice of group changes in 3 positive notes, Oct 27–Nov 11).
- **P5 Life Skills:** Maya — recipe-card groups (Nov 11 joined new group calmly after notice). John — multi-step recipes (numbered written steps alongside smoother starts, 3 notes). Alex — moving between stations (transition warnings helpful, 2 notes).
- **PL Planning:** no student reminders; planning-time suggestions (see §6.1).

**Observations.** The prototype seeds 10 entries so the log/timeline look real. **For the real build, seed ~40–100 coherent dated observations over 6–8 weeks (roughly Sep 21–Nov 11)** so Insights charts and timeline filters are meaningful, including many positive entries. Each observation: `id, studentId, occurredAt, classId, activity, note, antecedent[], behavior[], consequence[], support[], outcome, intensity, observationType (incident|positive), draft (bool), reviewed, source`. Seeded sample entries (newest first):
1. Wed Nov 11 1:20 PM — Maya, Life Skills, positive — Group work change › Joined group calmly › Positive interaction — "Had advance notice about the new groups, joined without prompting."
2. Wed Nov 11 10:20 AM — John, Math Support, positive — Long independent task › Used a break card, Returned to task › Offered a break — "Took the break card on his own, back at work in about 5 min."
3. Tue Nov 10 9:35 AM — Alex, Science, positive — Activity transition › Transitioned smoothly › Positive interaction.
4. Mon Nov 9 9:20 AM — John, Science, incident — Long independent task › Work refusal › Offered a break — "Pushed the reading away; break offered, finished a shorter section after."
5. Fri Nov 6 8:40 AM — Alex, English Support, incident — Activity transition › Left designated area › Verbal redirection.
6. Thu Nov 5 9:25 AM — Maya, Science, positive — Unclear directions › Asked for help › Verbal redirection.
7. Tue Nov 3 9:30 AM — John, Science, positive — Long independent task › Used a break card, Returned to task › Offered a break — "Re-engaged within 5 minutes."
8. Tue Oct 27 10:15 AM — John, Math Support, positive — Given demand / task › Asked for help › Task shortened / chunked — "Asked for a shorter segment before getting frustrated."
9. **Intervention marker** — Mon Oct 5 — support plan started (timeline only, not an observation).
10. Tue Sep 29 9:20 AM — John, Science, incident — Long independent task › Work refusal › Verbal redirection — "Pushed workbook away during a long reading task; task not finished."
11. **Seed draft** — Wed Nov 11 8:25 AM — Alex, English Support — Antecedent "Activity transition", note "Needed a second reminder to move to share-out. Add what helped." (so the Needs-attention panel has something to show on first load).

Make sure Today reminders, Class Prep, student timelines, and Insights are all **derived from the same store**, not separate hard-coded claims.

## 9. Privacy, fairness, and safety boundaries

- **Synthetic data only.** Show a small "Demo uses fictional data" indicator. In any real deployment, student records raise FERPA and district policy needs (role-based access, retention, transparency, approvals, vendor/security review).
- **No** prediction of an individual child's imminent behavior, "crash-out probability," violence score, diagnosis, or discipline/placement recommendation.
- **No** claim of automatically performing an FBA or producing an approved BIP.
- **No** speculative psychology stated as fact (never "John threw the book because he wanted to escape reading").
- Teachers validate all extracted observations and all generated summaries. Every summary says **"Draft — review required."**
- Highlight improvements, strengths, and effective supports alongside difficulties. Show evidence trails and uncertainty.
- **Decision: teachers may upload the full IEP** (see §6.6). Mitigations: label the file a **"Sensitive record"**, visible only to the teacher; keep **diagnosis text out of everyday screens** (Today, Log, Class Prep show supports and strengths only); show only first name + last initial on screen; only teacher-approved items drive reminders. All demo IEPs are fictional. A real deployment still needs district and FERPA approval.
- Students are archived, not deleted, to respect record retention.

## 10. AI behavior (mocked, behind `aiService`)

All AI is **simulated and deterministic**. Be explicit in code comments and UI ("Simulated for this demo"). A real LLM is optional and must not block the demo.

1. `structureNote(text, context) → draft` — keyword/regex extraction into A/B/C/support/outcome. Flags uncertain/missing fields as "Not documented". Never invents facts.
   *Sample input:* "During independent science reading around 9:15, John tossed his workbook after the instructions seemed confusing. I offered a break and then split the reading into smaller chunks. He returned to the activity." → A: independent science reading; instructions seemed confusing · B: tossed workbook · C: break offered; reading split into smaller chunks; returned to activity · Support: break + chunking · Outcome: re-engaged · Duration/Location: Not documented.
2. `getPrepReminders(classId, activity) → Reminder[]` — match class/activity to approved supports and past observations; return student, context, evidence sentence with dates and source IDs, and a documented support. Must be computed from the store.
3. `summarizeTrends(studentId?, classId?) → text` — descriptive counts before/after an intervention start date; never causal.
4. `draftSummary(studentId, kind: "meeting"|"handoff") → editable draft` — strengths-based, grounded in saved (non-draft) observations, with source links.

## 11. Terminology
**SPED** special education · **IEP** Individualized Education Program · **504** accommodation plan · **ABC** Antecedent–Behavior–Consequence · **FBA** Functional Behavioral Assessment · **BIP** Behavior Intervention Plan · **FERPA** student-records privacy law.

## 12. Competitor context (differentiate, don't clone)
ABC Tracker, CentralReach LiftEd, AbleSpace, RethinkEd, Branching Minds. **Our differentiator:** the teacher's **current period + planned activity** drives timely, grounded, individualized support *preparation*; logging and analytics power that daily workflow. Do not copy competitors' branding or UI. (The Log's tap-to-choose pattern was inspired by an existing app but is restyled and uses strengths-based, softened wording plus positive logging.)

## 13. Definition of done

**Must have**
- Navigation for Today, Log, Class Prep, Students, Insights; persistent Log button; Demo-data badge.
- Demo clock (presets, Next class, Play) that drives class context everywhere; default 9:00 Science.
- Today: class strip, hero, evidence-backed reminders (Add to prep / Dismiss), live Needs-attention, weekly snapshot.
- Log: tap-to-choose A/B/C/Support, quick fills, own-words note, Incident/Positive toggle, Save + Save as draft + Resume/Discard, Recent log with filters.
- Class Prep checklist with working check/uncheck and custom notes.
- Students: roster/search, add/edit/archive/restore, multiple plans (add/edit/remove), timeline that updates on save, handoff briefing.
- Insights: ≥2 charts from seed data with intervention marker (descriptive wording) and an editable "Draft — review required" summary panel.
- Consistent seed data; polished UI.

**Stretch:** printable summary page, advanced student filters, interventions overlay, responsive mobile layout. (Roster import in bulk is out for now: the demo adds **one student at a time**; real schools normally sync rosters from the student information system.)

**Out of scope:** production LLM inference, district/SIS integrations, real student data, multi-tenant permissions, automated placement recommendations, predictive scoring, real notifications, clinically validated recommendations, real voice input.

## 14. Demo narrative (2–4 minutes)
1. Open **Today** at 9:00 → Science · Period 2 hero.
2. Read John's reminder (evidence + documented support + source link); click **Add to prep**.
3. Click **Open class prep** → check off an item.
4. Tap **＋ Log observation** → use a quick fill, add a note in your own words → **Save observation** (or **Save as draft** and show it appear in Needs attention, then resume it).
5. Open **John's profile** → new entry on the timeline; show his plans.
6. Open **Insights** → patterns and the Oct 5 intervention marker → draft meeting summary with source links.
7. Show **Handoff briefing** on a student profile.
8. Press **▶ Demo: add a student** in the header, drag in a sample IEP and behavior log, review, and add the 4th student (see §6.6); then see them in the roster and in Today's reminders.
9. Switch demo clock to **10:00** (or tap **Next class** / **Play**) → Math Support with different reminders; also tap the class strip to look ahead; at **11:00** show Planning suggestions.

## 15. Known gaps in the reference prototype (`index.html`)
- Class Prep and Insights are still static (Science/John only); they don't react to the clock, new students, or saved observations.
- Insights filters, "Print view," and the meeting-summary editor are visual only.
- State is in memory only (refresh resets).
- The "Structure note" AI action was removed from Log in favor of tap-first entry; add it back as an optional helper if time permits.
- Accessibility pass is partial (focus rings and keyboard activation exist on key controls, not audited).

## 16. Open questions for the team
- Should Class Prep also use the demo clock to pick its class automatically? (Recommended: yes, with a class/activity selector override.)
- Do we want a "day" picker on the demo clock for multi-day storytelling?
- Keep the roster at 3, or seed 8–12 for a fuller Science class? (UI already supports adding students.)
- Should the Recent log also live inside each student's profile with the same filters? (Timeline already covers most of this.)
