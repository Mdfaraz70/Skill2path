 Skill2Path™

Your skills. Your opportunities. Your path.

Created by Mohammad Faraz

---

#What this repo contains

This repo ships two things:

1. **A working MVP** (`index.html`) — a self-contained, functional Skill2Path application you can open in any browser right now. It implements real user flows: assessment → skill analysis → opportunity matching → personalized roadmap → dashboard → CV builder → application tracker → scam checker → AI mentor. No fake data screens — the matching and analysis are computed live by a deterministic rule-based engine (see `index.html` → `scoreOpportunity()` / `runAnalysis()`).
2. **This README**, documenting the full product, the recommended production architecture (Next.js/Supabase), and the roadmap for turning the MVP into that production system.

## The problem

People with real, usable skills often don't know how those skills translate into income. Generic "learn to code" or "start freelancing" advice ignores a person's actual education, time, location and constraints — and the online spaces where they go looking for opportunities are full of scams promising guaranteed income.

## The solution

Skill2Path takes a short, honest assessment and turns it into:
- A clear-eyed **skill analysis** (strengths, level, gaps, confidence score — labeled as a recommendation, not a certified test)
- A ranked list of **matched opportunities**, scored against the user's real skills, time, and preferences
- A **7/30/60/90-day roadmap** with concrete, buildable **portfolio projects**
- Tools to act on it: a **CV builder**, an **application tracker**, a **scam checker**, and an **AI mentor** for follow-up questions

Every surface is built around one rule: **never guarantee a job or an income.**

## Features (MVP, implemented in `index.html`)

| Feature | Status |
|---|---|
| Landing page (hero, how it works, features, safety, founder, FAQ) | ✅ |
| Multi-step skill assessment | ✅ |
| Skill analysis dashboard (level, gaps, confidence score) | ✅ |
| Opportunity matching engine (rule-based scoring, not hard-coded per-user text) | ✅ |
| Personal roadmap (7/30/60/90 day) | ✅ |
| Project/portfolio builder (per-opportunity project ideas) | ✅ |
| CV / resume builder with live preview and print/download | ✅ |
| Application tracker (Saved/Applied/Interview/Rejected/Accepted) | ✅ |
| Scam protection checker (rule-based pattern detection + optional AI second opinion) | ✅ |
| AI career mentor (chat, Claude API, server-side key handling) | ✅ |
| Multilingual architecture (English + Hindi string tables, expandable) | ✅ |
| Professional dashboard | ✅ |

### Deliberately out of scope for the MVP (see "Future features")

Accounts/auth, a real database, payments, employer partnerships, verified job listings, native mobile apps.

## How the recommendation engine works

The matching engine is intentionally **not** an AI hallucinating plausible-sounding advice. It's a transparent scoring function:

```
score = (required-skill overlap × 70%)
      + (nice-to-have overlap × 15%)
      + (work-type fit bonus)
      + (time-availability fit bonus)
      + (experience-level bonus)
```

This runs against a curated catalog of opportunity categories (`OPPORTUNITIES` in `index.html`), each with required skills, difficulty, learning time, a broad estimated earning range (explicitly labeled as an estimate), safety notes, and legitimate ways to find that kind of work. AI (Claude) is used only where judgment/conversation genuinely helps — the mentor chat and an optional "AI second opinion" on a pasted job offer — never to fabricate the core numbers.

## Architecture (production target)

The MVP is a single static HTML file so it's runnable anywhere with zero setup. The recommended production architecture:

```
Frontend:        Next.js / React
Styling:         Tailwind CSS
Backend:         Next.js API routes
Database:        PostgreSQL (Supabase)
Authentication:  Supabase Auth (email/password to start)
AI:              Anthropic API, called only from server-side routes
                 (API key lives in server environment variables, never
                 shipped to the browser)
```

### Proposed file structure

```
skill2path/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
├── .env.example
├── documentation/
│   ├── architecture.md
│   ├── data-model.md
│   └── design-system.md
├── src/
│   ├── app/                 # Next.js app router pages
│   │   ├── page.tsx                 # landing
│   │   ├── assessment/page.tsx
│   │   ├── dashboard/page.tsx
│   │   ├── opportunities/[id]/page.tsx
│   │   ├── cv-builder/page.tsx
│   │   ├── tracker/page.tsx
│   │   ├── scam-check/page.tsx
│   │   └── mentor/page.tsx
│   ├── api/
│   │   ├── analyze/route.ts         # skill analysis
│   │   ├── match/route.ts           # opportunity matching
│   │   ├── mentor/route.ts          # AI mentor (server-side Claude call)
│   │   └── scam-check/route.ts      # AI second-opinion (server-side Claude call)
│   ├── components/          # reusable UI components
│   ├── lib/
│   │   ├── engine/                  # deterministic scoring engine
│   │   ├── db/                      # Supabase client + queries
│   │   └── ai/                      # Claude service layer
│   └── i18n/                        # language string tables (en, hi, ...)
├── public/
└── tests/
```

### Data model (proposed tables)

- **users** — id, email, created_at
- **profiles** — user_id, age, location, education, hours_per_week, income_goal, online_pref
- **skills** — id, name (normalized skill taxonomy)
- **profile_skills** — profile_id, skill_id
- **opportunities** — id, name, category, difficulty, learning_weeks, earning_range, description
- **opportunity_required_skills** — opportunity_id, skill_id, weight
- **assessments** — id, profile_id, submitted_at, raw_answers (jsonb)
- **recommendations** — id, profile_id, opportunity_id, match_score, generated_at
- **roadmaps** — id, profile_id, opportunity_id, horizon (7/30/60/90), steps (jsonb)
- **projects** — id, opportunity_id, title, description, difficulty
- **applications** — id, profile_id, company, position, date_applied, status, notes, follow_up_date
- **saved_opportunities** — profile_id, opportunity_id
- **learning_progress** — profile_id, skill_id, status

#Environment variables

```
# .env.example
DATABASE_URL=
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
ANTHROPIC_API_KEY=
```

`ANTHROPIC_API_KEY` and `SUPABASE_SERVICE_ROLE_KEY` must only ever be read on the server (API routes / server components) — never exposed to client bundles.

# Running the MVP locally

The MVP has no build step.

1. Download `index.html`
2. Open it directly in any modern browser, **or** serve it locally:
   ```bash
   npx serve .
   ```
3. Use the app: Start Assessment → see Analysis → see matched Opportunities → open a Roadmap → try the CV Builder, Tracker, Scam Checker and Mentor.

Note: the AI Mentor and "AI second opinion" scam-check feature call the Anthropic API. When run as a Claude.ai artifact, this is handled by the host platform's built-in proxy. If you deploy `index.html` standalone outside that environment, those two features will need to be pointed at your own server-side proxy endpoint instead of calling `api.anthropic.com` directly — do not put a real API key in client-side code.

## Development (production Next.js version)

```bash
npm install
npm run dev        # start local dev server
npm run build       # production build
npm run test         # run test suite
```

#Testing

- **Engine tests**: unit tests for `scoreOpportunity()` and `runAnalysis()` against fixed profiles, asserting deterministic, explainable output.
- **Component tests**: assessment form validation, roadmap tab switching, tracker CRUD.
- **E2E**: full flow from landing → assessment → analysis → opportunity detail → roadmap.
- **AI-output validation**: any AI-generated JSON (mentor structured suggestions, future features) is schema-validated before being rendered — never trusted blindly.

# Deployment

1. Push to GitHub
2. Connect the repo to Vercel (or similar) for the Next.js app
3. Provision a Supabase project; run migrations from `documentation/data-model.md`
4. Set environment variables in the hosting platform (never commit `.env`)
5. Deploy `main` → production; feature branches → preview deployments

#Publishing to GitHub

```bash
git init
git add .
git commit -m "Initial commit: Skill2Path MVP"
git branch -M main
git remote add origin https://github.com/<your-username>/skill2path.git
git push -u origin main
```

Include `LICENSE`, `SECURITY.md` (how to report vulnerabilities), and `CONTRIBUTING.md` (how to propose changes) at the repo root before making it public.

# Security

- No API keys in client code, ever.
- No collection of passwords, OTPs, or financial credentials — the product explicitly warns users never to share these with anyone.
- Minimal data collection: only what's needed to generate an assessment.
- AI-generated content is validated (schema/shape checked) before being shown to users, and is clearly distinguished from the deterministic recommendation engine's output.
- Report vulnerabilities per `SECURITY.md` (to be added) rather than public GitHub issues.

#Ethical rules (non-negotiable, enforced throughout the product)

Skill2Path never: guarantees employment or income, invents job openings or salary statistics, claims an opportunity is definitely legitimate, encourages illegal activity, or asks for passwords/OTPs/financial credentials. AI outputs are treated as suggestions, not verified facts, and are labeled as such in the UI.

# Monetization (future)

The architecture supports, without being required to launch:
- Premium personalized roadmaps (deeper, longer-horizon plans)
- Premium CV templates
- Advanced/verified skill assessments
- 1:1 career coaching marketplace
- Educational partnerships (course providers)
- Employer/recruiter partnerships (verified job postings)

The free tier (assessment, matching, basic roadmap, CV builder, tracker, scam checker) remains fully usable on its own.

#Roadmap — future features

- Real user accounts and saved history (Supabase Auth)
- Persistent database-backed profiles, applications and progress
- Verified/employer-partnered opportunity listings, clearly separated from general recommendations
- Expanded language support beyond English/Hindi
- Richer learning-progress tracking with completion checkmarks
- Community/mentor marketplace
- Mobile app (React Native) reusing the same engine and API routes
- More sophisticated matching (embeddings-based skill similarity instead of substring matching)

#Credits

Product concept, direction and founder: Mohammad Faraz