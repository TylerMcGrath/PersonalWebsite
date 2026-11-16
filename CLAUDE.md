# CLAUDE.md: Tyler McGrath personal website

This file is the handoff for managing Tyler's portfolio site. Read it fully before changing anything.

## What this is

A single-page personal portfolio for Tyler McGrath, an embedded software engineer (C/C++, CAN, real-time systems, robotics) based in New York City. Its job is to make hiring managers and collaborators in embedded, robotics, and cyber-physical systems want to contact him.

- Repo: `TylerMcGrath/PersonalWebsite` on GitHub
- Hosting: Vercel, auto-deploys on every push to `main`
- Stack: one static HTML file with inline CSS and JS. No framework, no build step, no package.json.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire site: markup, styles, and the hero demo script |
| `assets/climbing.jpg` | The only photo, used in Off the bench. 900×1200, compressed |
| `TylerMcGrath_Resume2026.pdf` | Linked from the Contact section. The link is relative, so the filename must match exactly |
| `CLAUDE.md` | This file |

If the resume is replaced with a new file, either keep the same filename or update the `href` in the Contact section of `index.html`.

## Deployment

Vercel project settings (already configured, don't change them unless asked):

- Framework Preset: **Other**
- Root Directory: `./`
- Build Command: empty
- Output Directory: empty

Pushing to `main` deploys to production within about a minute. Pushes to other branches get preview URLs. Check status in the Vercel dashboard under Deployments, or with `vercel ls` if the Vercel CLI is installed and logged in.

If `index.html` ever stops being at the repo root, or a `package.json` with a build script is added, Vercel's behavior changes. Avoid both unless Tyler asks for a framework migration.

## Working rules

1. **Ask before pushing.** Show Tyler the diff and get approval before every `git push`. Commits on a local branch are fine without asking.
2. **No invented content.** Every claim, number, date, and project on the site must come from Tyler's resume or from Tyler directly. Never round, inflate, or add metrics. If something is missing, ask him.
3. **No em dashes** anywhere in site copy, commit messages, or docs. Use periods, commas, colons, or parentheses. En dashes in date ranges (`June 2021 – Sept 2024`) are fine.
4. **No phone number** on the site. Email, GitHub, LinkedIn, and the resume PDF are the only contact methods.
5. **No placeholder text** on the live site. If a detail isn't known, write general copy or leave the item out.
6. **The "Currently exploring" card stays general.** Don't name the company, product, or platform behind the SLAM work without Tyler's explicit OK.
7. **Keep it one file** unless Tyler asks otherwise. No external JS libraries. The only external resource is Google Fonts.
8. **Plain writing.** Short, direct sentences in the first person. No buzzwords, no "passionate about", and no self-ratings like "expert in".

## Site structure

Sections in page order, with their anchor ids:

| Section | id | Notes |
|---|---|---|
| Header strip | n/a | Sticky. Name links to `#top`, nav links to each section |
| Hero | `top` (on `<main>`) | Status pill, role line, headline, intro, 3-metric strip, tabbed demo panel (Navigation, Localization) |
| Projects | `projects` | Lead card (ArduPilot IMU detector) plus half-width cards |
| Experience | `experience` | Continental AG (now Aumovio), three roles, newest first |
| Skills and education | `skills` | Five skill groups from the resume, then Georgia Tech and Wayne State |
| Off the bench | `off` | Climbing photo plus three hobby cards. The hobby copy is Tyler's own wording, so don't rewrite it |
| Contact | `contact` | Email with copy button, GitHub, LinkedIn, resume PDF |
| Footer | n/a | Name, city, and "Last updated" month. Update the month whenever content changes |

## Current facts (source: Tyler's 2026 resume)

- **Education:** Georgia Tech MS in Computer Science, graduating Dec 2026, GPA 3.8. Wayne State B.S. Electrical Engineering, April 2020.
- **Continental AG (now Aumovio):** Software Test Engineer, Sept 2024 – May 2026, Auburn Hills, MI. Associate Test Engineer, June 2021 – Sept 2024, Auburn Hills, MI. Electrical Engineering Intern, June 2018 – July 2019, Troy, MI.
- **Hero metrics:** 400 Hz onboard detection, 0.74 cm navigation error, 4 days to 1 day test time.
- **Contact:** tylerjmcgrath@yahoo.com, github.com/TylerMcGrath, linkedin.com/in/tyler-mcgrath0

When the resume changes, update the site to match and check every number against the new version.

## Design system

The CSS opens with a token block on `:root`. Always use tokens and never hard-code colors.

- **Colors:** `--mat` (page ground), `--panel` (cards), `--ink` (text), `--muted`, `--line` (borders), `--accent` (amber, for highlights and alerts), `--trace` (teal, for data and positive results), `--ok`, `--planned`, `--fault-bg`.
- **Themes:** light values on bare `:root`, dark values in both `@media (prefers-color-scheme: dark) { :root:not([data-theme="light"]) }` and `:root[data-theme="dark"]`. Any new token needs a value in all three places.
- **Fonts:** `--display` (Barlow Condensed, headings and big numbers), `--body` (IBM Plex Sans), `--mono` (JetBrains Mono, labels, chips, data).
- **Look:** an embedded engineer's bench. Instrument-style labels in uppercase mono, datasheet-like panels, tabular numbers. Keep new elements in that style.
- **Status LEDs:** `.led` (green, complete), `.led.pending` (amber, in progress), `.led.planned` (grey, exploring or planned).

### Adding a project card

Copy this inside `<div class="builds">`. Use `half` for a half-width card, or `lead` for full width. Cards fill a 6-column grid, so keep half cards in pairs.

```html
<article class="panel build half">
  <div class="build-meta"><span class="state"><span class="led"></span>Complete</span><span>Tech · Stack · Here</span></div>
  <h3>Project name</h3>
  <p>One or two sentences on what it is and what it does.</p>
  <ul class="pts">
    <li>A concrete result, with a real number if there is one.</li>
  </ul>
  <ul class="chips"><li>Tool</li><li>Tool</li></ul>
</article>
```

For a headline comparison (like 0.74 cm vs 9.3 cm), add a `.pair` block after the `<h3>`. Give the better number `class="good"`.

## The hero demos

The hero panel has two tabs. Both are browser simulations, labeled as simulations, and must stay labeled that way. Both arenas are 160 × 100 cm. Only the visible tab runs, to save CPU. The code is in the `<script>` at the bottom, in two modules: `NAV` and `PF`.

### Navigation tab (`NAV`, based on the Robotarium project)

- Differential-drive robot with a state machine: `ROTATE`, then `DRIVE`, then `SETTLE` (waits for lag to flush and records the stop error), then `ARRIVED`, which is latched.
- Each leg gets random hardware error in `newHw()`: speed scale ±3%, turn scale ±5%, and wheel-mismatch curvature. This is what makes open loop drift.
- **Feedback** uses an overhead-tracker pose with 0.12 cm noise. **Open loop** plans timed turn-then-drive moves from where it believes it is, so errors pile up. It re-homes after each tour.
- **0.4 s command lag** delays every command. **Predictive stopping** projects the pose forward through the queued commands (`predicted()`) before deciding.
- Visitors can click to queue up to 6 waypoints. The preset tour resumes 4 s after they finish.
- Measured results, which should stay close to the resume: feedback about 0.7 cm, open loop about 9 cm average, lag without prediction about 1.7 cm, and lag with prediction about 0.7 cm. Re-check these if you touch the controller or `newHw()`.

### Localization tab (`PF`, based on the localization project)

- Monte Carlo localization: 300 particles, a range-only sensor with a 60 cm reach, and 7 landmarks placed asymmetrically. True range noise is σ 2.5 cm. The filter uses σ 5 (`SIG_F`) to avoid particle collapse.
- Updates at 5 Hz, with low-variance resampling when effective particles drop below half. Each resample adds jitter and replaces 2% of particles with random ones (`INJECT`), which is what lets it recover from a kidnapping.
- It also penalizes particles that expect to see a landmark the robot can't see.
- The robot auto-kidnaps every 22 s (off when the visitor has reduced motion turned on). Visitors can click to move it, or scatter the particles.
- Expected behavior: it converges within a few seconds, then holds an error of about 1 to 3 cm, recovering within about 5 s after a kidnap.

## Before every push

1. Open `index.html` in a browser and check for console errors.
2. Check at desktop width and at about 400px wide. Nothing should scroll sideways.
3. Check light and dark mode.
4. Confirm both demo tabs animate, clicking the arena works, and every toggle and button does something.
5. Confirm every link opens the right place and the resume PDF loads.
6. Search for em dashes. `grep -nP '\x{2014}' index.html` should print nothing.
7. Update the footer's "Last updated" month if content changed.
8. Show Tyler the diff and wait for approval.

## Git and environment notes

- Tyler works on Windows. Git Bash runs in MinTTY, which can't show interactive prompts. If a CLI needs interactive login, run it in PowerShell or prefix it with `winpty` (for example `winpty gh auth login`).
- GitHub auth goes through the GitHub CLI (`gh auth status` to check). If a push fails with "Password authentication is not supported", have Tyler run `gh auth login` in PowerShell, then `gh auth setup-git`.
- Commit messages: short, imperative, and specific (for example "Add Robotarium project card").

## Ideas Tyler may ask for later

- A custom domain (Vercel project Settings, then Domains).
- Linking specific GitHub repos from project cards once they're public.
- More photos or a demo video, if Tyler provides them. Store them in `assets/`, compress them (under about 300 KB per image), and give each one alt text.
- Updating the "Currently exploring" card as the SLAM work produces results.
