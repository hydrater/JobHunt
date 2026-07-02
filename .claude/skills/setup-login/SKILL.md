---
name: setup-login
description: Open JobStreet, LinkedIn, MyCareersFuture, and Indeed in the persistent browser so the user can log in once. Sessions are saved to .browser-profile/ and reused by the hunt skill. Run this before the first hunt, or whenever a session expires.
---

# setup-login

The `/hunt` skill needs to browse job sites as a logged-in user (LinkedIn in
particular hides most jobs behind auth). This skill drives the **Playwright
MCP** browser to each site and lets the user log in manually. Cookies persist
in `.browser-profile/` (gitignored), so future runs stay logged in.

## Prerequisites
- The `playwright` MCP server must be connected (configured in `.mcp.json`).
  If its tools aren't available, tell the user to restart Claude Code so the
  MCP server loads, and to run `npx playwright install chromium` once.

## Steps

For each site below, in order:

1. **JobStreet** — navigate to `https://sg.jobstreet.com/oauth/login`
   (or `https://www.jobstreet.com.sg/`).
2. **LinkedIn** — navigate to `https://www.linkedin.com/login`.
3. **MyCareersFuture** — navigate to `https://www.mycareersfuture.gov.sg/`
   (login via Singpass when prompted).
4. **Indeed** — navigate to `https://sg.indeed.com/account/login`
   (or `https://sg.indeed.com/`).

For each one:
- Open the page with the browser, then **pause and tell the user**: *"Please
  log in to <site> in the browser window, then reply `done` here."*
- Wait for the user to confirm before moving to the next site. Do **not** try
  to type their credentials yourself — they log in manually (handles 2FA,
  Singpass, captchas).
- After they confirm, take a quick snapshot to verify a logged-in state (e.g.
  the page shows their feed / profile avatar) and note it.

## Finish
- Confirm all four sessions are active and saved to `.browser-profile/`.
- Remind the user that `.browser-profile/` is gitignored and contains live
  login cookies — they should never commit or share it.
- Tell them they're ready to run `/hunt`.

## Notes
- Region defaults assume **Singapore** (JobStreet SG, MyCareersFuture is
  SG-only). If the user is elsewhere, ask for their JobStreet/LinkedIn country
  and adjust the URLs.
- If a session later expires, just re-run `/setup-login`.
- **External ATS logins persist too.** `/hunt` follows postings that redirect to
  external application sites (Workday, Greenhouse, Lever, Ashby, iCIMS, company
  careers pages) and applies there. Those sites save cookies to the same
  `.browser-profile/`, so if the user expects to apply through a particular ATS
  a lot, they can ask to open and log into it here as well — future redirected
  applications will reuse that session instead of hitting an account wall.
