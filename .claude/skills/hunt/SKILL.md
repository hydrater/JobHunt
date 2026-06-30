---
name: hunt
description: The main job-hunting loop. Crawls JobStreet, LinkedIn, and MyCareersFuture with the browser using scope.txt, and for each matching job creates an output folder with job_posting.pdf, a tailored resume.pdf and cover_letter.pdf, then tracks it in applications.csv (Found → Applied/Error).
---

# hunt

Crawl the job boards, find postings that fit the user's scope, and produce a
tailored application package per job while tracking everything in a CSV.

## 0. Preconditions (check, then proceed)

- **`info.txt`** exists at project root. If missing → tell the user to run
  `/build-info` first and stop.
- **`scope.txt`** exists at project root. If missing → copy
  `scope.example.txt` to `scope.txt`, tell the user to edit it, and stop.
- **`applications.csv`** exists at project root. If missing → create it by
  copying `applications.template.csv` (header row only).
- **Playwright MCP** tools are available and the user is logged in. If browsing
  a site lands on a login wall → tell the user to run `/setup-login` and stop.
- Read `info.txt` and `scope.txt` fully into context now.

Ask the user (or accept from their message): **how many matched jobs to apply
for this run** (default 10) and whether to **auto-submit applications** or
**stop at draft** (default: prepare everything but pause before final submit on
each one). Note their choice.

## The main loop — ONE job, start to finish, before the next

**This is the most important rule of the skill. Do NOT batch.**

Process exactly one job through its *entire* lifecycle before you look for
another. For each job, in this order:

1. **Find** the next matching job (§1 crawl + §2 filter) — stop as soon as you
   have ONE match.
2. **Build** its application package (§3) and seed/track the row (§4).
3. **Apply** to it, or stop at draft per the user's choice (§5), and update its
   state in the CSV (§4).
4. Only once that job has reached a final state (**Applied**, **Error**, or
   draft-ready **Found**) do you return to step 1 for the next job.

Never collect a list of matches and build them all, and never build several
packages and apply at the end. Find one → build one → **apply to that one** →
then find the next. Repeat until you reach the user's matched-job limit or run
out of matches.

The numbered sections below (§1–§5) are the *details* for each step of one loop
iteration — not sequential phases to run across all jobs.

## 1. Crawl (per site, guided by scope.txt)

While crawling, return to step 2 of the main loop the moment you identify a
single matching job — do not keep paging to amass a batch. Remember where you
were in the results so you can resume the crawl for the next iteration.

For each of: **JobStreet (sg.jobstreet.com)**, **LinkedIn (linkedin.com/jobs)**,
**MyCareersFuture (mycareersfuture.gov.sg)**:

1. Build a search from `scope.txt`: use the target roles as keywords and the
   location filter. Navigate to the site's job search with those terms.
2. Read the results list (use the browser snapshot, not screenshots, to get
   structured text + links). Page through a reasonable number of results.
3. For each listing, capture: company, role title, location, source site, and
   the canonical job URL.

## 2. Filter against scope

For each candidate posting, open it and judge it against `scope.txt`:
- **Skip** if it matches anything in the "Avoid" list (wrong role type,
  staffing agency repost, below salary floor, excluded tech, seniority
  mismatch, etc.).
- **Skip duplicates**: read `applications.csv`; if the same company+role or the
  same job_url already has a row, skip it.
- **Keep** if it fits the target roles/seniority/location. When genuinely
  unsure, keep it and record the doubt in the row's `notes`.

Respect: **one application per company per run** (unless distinct roles), and
stop once you reach the user's matched-job limit.

## 3. Build the application package (for the ONE current match)

> You arrive here with a single matched job. Build its package, then go straight
> to §5 and apply to it *before* returning to §1 for another. Do not build a
> second package until this job has been applied to (or drafted/errored).


Let `STAMP` = current date-time as `YYYY-MM-DD_HHMM` (get it via
`date "+%Y-%m-%d_%H%M"` in the Bash tool — do not guess the time).

1. **Make the folder**:
   `output/<STAMP>_<Company>_<Role>/`
   Sanitize company & role for the filesystem: keep letters/digits, replace
   spaces with `_`, strip `/ \ : * ? " < > |`, trim length to ~60 chars.

2. **Seed the tracking row immediately** with state **`Found`** (see §4) so the
   job is recorded even if a later step fails.

3. **`job_posting.pdf`** — with the job posting open in the browser, use the
   Playwright **`browser_pdf_save`** tool to save the page as
   `output/<folder>/job_posting.pdf`. Also save the raw posting text to
   `output/<folder>/job_posting.txt` for your own reference when tailoring.

4. **`resume.pdf`** — copy `templates/resume_template.html` into the folder as
   `resume.html`. Fill every `{{PLACEHOLDER}}` using `info.txt`, reordering and
   rewording bullets/skills to match THIS posting's requirements. Never
   fabricate experience. **Preserve the Harvard format**: single column, black
   on white, serif, no colors/graphics/chips; bullets start with strong
   past-tense action verbs and quantify impact; reverse-chronological. For a
   senior candidate you may move Experience above Education. Then open
   `file://<abs path>/resume.html` in the browser and `browser_pdf_save` →
   `resume.pdf`.

5. **`cover_letter.pdf`** — same flow with
   `templates/cover_letter_template.html` → `cover_letter.html` →
   `cover_letter.pdf`. Reference something concrete from the posting; map the
   candidate's strongest quantified wins to the role's needs; keep to one page.

6. **`match_report.md`** — write a short audit so the user can verify the
   resume is genuinely specialized for THIS role. Format:

   ```
   # Match report — <Company> · <Role>

   ## Key requirements (from the posting)
   1. <requirement>  → addressed by: <resume bullet / skill used>  [strong | partial | gap]
   2. ...

   ## What this version emphasizes
   - <bullets/skills moved up or reworded for this role, and why>

   ## De-emphasized / omitted
   - <experience from info.txt left out as less relevant here>

   ## Gaps (requirements you don't fully meet)
   - <honest list — these are NOT invented in the resume>

   ## Tailoring score: <X>/10  — <one-line justification>
   ```
   Be honest about gaps; never close a gap by fabricating experience.

7. Leave the `.html` files in the folder — they make it easy for the user to
   tweak and re-export.

## 4. Track in applications.csv

Columns: `id,date_found,source,company,role,location,job_url,state,date_applied,folder,notes`

- `id`: next integer (max existing id + 1).
- `state`: one of **Found, Applied, Error, Interview, Offer**.
- Append a row with state `Found` as soon as the folder is made (§3.2).
- Append rows by writing the CSV carefully: quote any field containing a comma,
  quote, or newline per RFC 4180. Read-modify-write the whole file or append a
  correctly-formatted line — never corrupt existing rows.

## 5. Apply (do this now, before searching for the next job)

This runs immediately after building the package for the current job — it is the
last step of the loop iteration, not a phase deferred to the end of the run.

If the user chose auto-submit (or after they approve a prepared draft):
- Go to the posting's apply flow in the browser. For "Easy Apply" style forms,
  fill fields from `info.txt`, attach/point to `resume.pdf`, and submit.
- On confirmed submission → update that job's row: `state` = **Applied**,
  `date_applied` = today.
- If the apply flow is external, requires an account you don't have, hits a
  captcha you can't clear, demands info not in `info.txt`, or otherwise fails →
  set `state` = **Error** and put the reason in `notes` (e.g. "redirects to
  Workday — needs manual apply", "missing required field: years with Kafka").
- If the user chose draft-only → leave state as **Found** and note "package
  ready, awaiting manual submit".

Once this job's row reaches its final state, **return to §1 to find the next
matching job**. Stop when you hit the user's matched-job limit or run out of
matches.

Never invent answers to application questions (work auth, salary, years of
experience). If a required answer isn't in `info.txt`, stop that application,
mark **Error**, and tell the user what's needed.

`Interview` and `Offer` states are set by the **user** manually — don't touch
those.

## 6. Summary

When the run ends (limit reached or no more matches), print a table:
company | role | source | state | folder. Report counts (Found / Applied /
Error), list anything marked Error with its reason, and remind the user they
can update rows to Interview/Offer in `applications.csv` themselves.

## Guardrails
- Be honest in the CSV: only mark **Applied** when you actually confirmed a
  submission. A prepared-but-unsubmitted package is **Found**, not Applied.
- Don't hammer the sites — browse at a human pace; if a site shows a captcha or
  rate-limit, stop that site and note it.
- All output (`output/`, `applications.csv`, `info.txt`) is gitignored and
  stays local.
