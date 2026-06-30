# 🎯 JobHunt — an autonomous job-hunting agent for Claude Code

JobHunt turns [Claude Code](https://claude.com/claude-code) into a personal job
hunter. It reads your past resumes, learns what you're looking for, browses
**JobStreet**, **LinkedIn**, and **MyCareersFuture** in a real browser, and for
every posting that fits your scope it produces a tailored application package
and tracks the whole pipeline in a spreadsheet.

```
You add resumes ──► /build-info ──► info.txt
You write scope ──► scope.txt
                         │
                    /hunt loop  ──► browses job boards (logged in)
                         │
                         ▼
   output/2026-06-30_1432_Acme_Backend-Engineer/
        ├── job_posting.pdf      (the listing, saved as PDF)
        ├── resume.pdf           (tailored to THIS role)
        ├── cover_letter.pdf     (tailored to THIS role)
        ├── match_report.md      (requirement-by-requirement audit + gaps)
        └── …                    (+ editable .html sources)
                         │
                         ▼
   applications.csv   Found → Applied / Error → (you: Interview → Offer)
```

Everything personal stays on your machine — your resumes, your scope, your
generated applications, and your browser login are all **gitignored**.

---

## How it works

It's built entirely as **Claude Code skills** plus the **Playwright MCP** server
(a real browser the agent controls). There's no separate app to run — you just
open Claude Code in this folder and invoke skills.

| Skill | What it does |
|-------|--------------|
| `/build-info`  | Reads everything in `past_cv/` and compiles a structured `info.txt`. |
| `/setup-login` | Opens the job sites so you can log in once; the session persists. |
| `/hunt`        | The main loop: crawl → match against scope → build packages → track. |

---

## Setup

### 1. Prerequisites
- [Claude Code](https://docs.claude.com/en/docs/claude-code) installed.
- [Node.js](https://nodejs.org/) (for the Playwright MCP server, run via `npx`).
- Install the browser once:
  ```bash
  npx playwright install chromium
  ```

### 2. Get the project
```bash
git clone <this-repo-url> JobHunt
cd JobHunt
```

### 3. Add your career material
Drop your resumes and any career notes into **`past_cv/`**. See
[`past_cv/README.md`](past_cv/README.md) for what helps. Anything goes —
`.pdf`, `.docx`, `.md`, `.txt`.

### 4. Define your scope
```bash
cp scope.example.txt scope.txt   # Windows: copy scope.example.txt scope.txt
```
Edit **`scope.txt`** — the roles you want, locations, salary floor, and an
**Avoid** list. This is the agent's filter; be specific.

### 5. Open Claude Code and build your profile
```bash
claude
```
Then, inside Claude Code:
```
/build-info
```
This produces `info.txt`. Review it and re-run if you add more to `past_cv/`.

> If the `playwright` MCP tools aren't available, restart Claude Code so it
> picks up `.mcp.json`.

### 6. Log in to the job sites (once)
```
/setup-login
```
A browser opens to each site — log in manually (handles 2FA / Singpass). Your
session is saved to `.browser-profile/` and reused on every hunt.

### 7. Hunt
```
/hunt
```
Tell it how many jobs to target and whether to auto-submit or just prepare
drafts. It will crawl, build packages under `output/`, and fill
`applications.csv`.

---

## The tracking spreadsheet

`applications.csv` (created from `applications.template.csv` on first hunt):

| column | meaning |
|--------|---------|
| `id` | row number |
| `date_found` | when the agent found it |
| `source` | jobstreet / linkedin / mycareersfuture |
| `company`, `role`, `location` | the job |
| `job_url` | direct link |
| `state` | **Found · Applied · Error · Interview · Offer** |
| `date_applied` | set when an application is submitted |
| `folder` | the `output/…` folder with the PDFs |
| `notes` | doubts, errors, manual-apply reasons |

**State flow:** the agent sets `Found` when it discovers a fit, then `Applied`
on a confirmed submission or `Error` if it couldn't apply (reason in `notes`).
**You** set `Interview` and `Offer` manually as your search progresses. Open the
CSV in Excel or Google Sheets anytime.

---

## What's committed vs. local

| Committed to git (the "template") | Stays local (gitignored) |
|-----------------------------------|--------------------------|
| skills, templates, README | `past_cv/*` (your resumes) |
| `scope.example.txt` | `scope.txt` (your search) |
| `applications.template.csv` | `info.txt`, `applications.csv` |
| `.mcp.json` | `output/*` (your applications) |
| | `.browser-profile/` (your logins) |

---

## Notes, limits & etiquette
- **Region:** defaults assume **Singapore** (JobStreet SG, MyCareersFuture is
  SG-only). Tell `/setup-login` and `/hunt` your country to adjust.
- **Honesty:** the agent never fabricates experience and only marks `Applied`
  when a submission is actually confirmed. Anything it can't answer from
  `info.txt` (work auth, exact years, salary) becomes an `Error` for you to
  handle, not a guess.
- **Be a good citizen:** it browses at a human pace and backs off on captchas /
  rate limits. Respect each site's Terms of Service — you are responsible for
  how you use it.
- **Review before sending.** Tailored resumes and cover letters are a strong
  first draft; skim them before anything goes out.
- **Verify the tailoring.** Each job folder has a `match_report.md` mapping the
  posting's requirements to the exact bullets/skills used, what was
  emphasized/dropped, honest gaps, and a tailoring score — so you can confirm
  every resume is specialized rather than generic.

---

## Troubleshooting
- *Playwright tools missing* → restart Claude Code; ensure `npx` works and run
  `npx playwright install chromium`.
- *Lands on a login page mid-hunt* → session expired; run `/setup-login` again.
- *Resume/cover letter PDFs look off* → edit the `.html` in the output folder
  and re-export, or tweak `templates/*.html` for all future runs.
- *Bad matches* → tighten `scope.txt`, especially the **Avoid** section.
