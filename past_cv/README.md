# `past_cv/` — drop your career material here

Put **anything about your professional history** in this folder. The
`/build-info` skill reads everything here and distills it into the
project-root `info.txt`, which the agent uses to tailor every resume and
cover letter.

### What to add
- Your latest resume / CV (`.pdf`, `.docx`, `.md`, `.txt` — all fine)
- Older resumes (different angles/roles are useful raw material)
- A LinkedIn profile export
- Bullet lists of achievements, metrics, projects
- A list of skills, certifications, languages
- Salary expectations, notice period, work-authorization status
- Anything else you'd want a recruiter to know

### Tips
- More signal = better-tailored applications. Don't worry about formatting.
- Include **quantified** wins ("cut p99 latency 40%", "led team of 5") — the
  agent will surface these in tailored resumes.
- Everything in this folder is **gitignored** — it never leaves your machine.

After adding files, run `/build-info` in Claude Code from the project root.
