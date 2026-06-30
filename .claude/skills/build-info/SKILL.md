---
name: build-info
description: Compile everything in past_cv/ into a single structured info.txt that the hunt skill uses to tailor resumes and cover letters. Run this after adding or updating files in past_cv/.
---

# build-info

Read every file the user dropped in `past_cv/` and distill it into a single,
well-structured `info.txt` at the project root. `info.txt` is the canonical
profile the `/hunt` skill reads when tailoring applications.

## Steps

1. **Find inputs.** List `past_cv/`. Read every document there. Handle common
   formats:
   - `.txt`, `.md`, `.html` → read directly.
   - `.pdf` → read with the Read tool (it extracts PDF text).
   - `.docx` → if the Read tool can't parse it, extract text with a quick
     script (e.g. `python -c "import docx; ..."` or unzip + read
     `word/document.xml`). If you truly can't read a file, list it under an
     "Unparsed files" note at the end of `info.txt` and continue.
   - Ignore `.gitkeep` and `README.md`.

2. **Synthesize, don't dump.** Merge overlapping content from multiple resumes.
   Deduplicate. Prefer the most recent / most detailed version of each fact.
   Keep ALL quantified achievements — never drop numbers.

3. **Write `info.txt`** at the project root using this structure:

   ```
   # CANDIDATE PROFILE
   ## Identity
   Full name, headline/title, email, phone, location, links (LinkedIn/GitHub/portfolio).

   ## Summary
   2–3 sentence professional summary.

   ## Work authorization & logistics
   Citizenship/PR/visa status, notice period, salary expectation, remote/onsite preference.

   ## Experience
   For each role (most recent first):
   - Title — Company — Location — Dates
   - Tech/tools used
   - Bullets (quantified achievements, responsibilities, scope)

   ## Skills
   Grouped (Languages / Frameworks / Cloud & Infra / Tools / Soft skills).

   ## Projects
   Notable projects with what + impact + tech.

   ## Education
   Degrees, schools, years, honors.

   ## Certifications & Languages
   Certs, spoken languages with proficiency.

   ## Raw highlights for tailoring
   A flat bullet list of the 15–25 strongest, most reusable achievement
   statements — the agent will pick from these per job.

   ## Unparsed files
   (only if any input couldn't be read)
   ```

4. **Confirm.** Report how many source files were read, anything that couldn't
   be parsed, and any obvious gaps the user should fill (e.g. "no salary
   expectation found", "no contact email"). Suggest they re-run after fixing.

## Notes
- `info.txt` is gitignored — it stays local.
- Be faithful: never invent experience, employers, dates, or numbers. If
  something is missing, leave it blank and flag it rather than fabricating.
