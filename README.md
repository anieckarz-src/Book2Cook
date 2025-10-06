## Book2Cook

A web application that converts recipe e-books (PDF with text layer) into structured, editable recipes, enables weekly meal planning, and generates aggregated, editable shopping lists. The MVP focuses on the core flow: PDF import → automatic recipe extraction → inline editing → weekly plan → shopping list. The app targets Polish language users and includes a simple user account system.

---

### Table of contents
- [Project name](#book2cook)
- [Project description](#project-description)
- [Tech stack](#tech-stack)
- [Getting started locally](#getting-started-locally)
- [Available scripts](#available-scripts)
- [Project scope](#project-scope)
- [Project status](#project-status)
- [License](#license)

---

## Project description
Book2Cook helps users transform PDF cookbooks with a text layer into structured, searchable recipes that can be edited inline, planned into a weekly schedule, and turned into an aggregated shopping list. Key MVP constraints: no OCR, asynchronous processing with queue and retries, target latency < 2 minutes for ~20–30 MB files, single-user data isolation, and UI-only status notifications.

- Key assumptions (MVP):
  - Supported files: PDF with a text layer; files without a text layer are rejected (no OCR in MVP).
  - Asynchronous processing pipeline with queuing, retry, and UI notifications.
  - Target latency: < 2 minutes for ~20–30 MB files.
  - Limits: per-file ≤ 100 MB, per-user storage ≤ 500 MB, original PDFs retained for 6 months.
  - No exports in MVP; UI notifications only.
  - TOS requires user to confirm rights to the uploaded e-book.

For full product scope and acceptance criteria, see the PRD: `.ai/prd.md`.

## Tech stack
- Astro 5 (static-first web framework)
- React 19 (interactive UI components)
- TypeScript 5 (static typing)
- Tailwind CSS 4 (utility-first styling)
- Shadcn/ui (accessible React UI components)
- Supabase (PostgreSQL, auth, and BaaS)
- OpenRouter.ai (access to multiple AI models)
- GitHub Actions (CI/CD)
- DigitalOcean (container hosting)

Additional details: `.ai/tech-stack.md`.

## Getting started locally
- Prerequisites:
  - Node.js 22.14.0 (see `.nvmrc`)
  - npm (bundled with Node.js)

- Setup:
```bash
# Clone
git clone <your-repo-url>
cd Book2Cook

# Install dependencies
npm install

# Start dev server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

- Project structure (conventions):
```md
./src              # source code
./src/layouts      # Astro layouts
./src/pages        # Astro pages
./src/pages/api    # API endpoints
./src/middleware   # Astro middleware
./src/db           # Supabase clients and types
./src/types.ts     # Shared types (entities, DTOs)
./src/components   # UI components (Astro & React)
./src/components/ui# Shadcn/ui components
./src/lib          # Services and helpers
./src/assets       # Internal static assets
./public           # Public assets
```

## Available scripts
- `dev`: start the Astro dev server
- `build`: production build
- `preview`: preview the production build
- `astro`: run the Astro CLI directly
- `lint`: run ESLint
- `lint:fix`: fix ESLint issues
- `format`: run Prettier on the repo

Package highlights:
- Runtime deps: `astro@^5`, `@astrojs/react`, `react@^19`, `react-dom@^19`, `tailwindcss@^4`, `@tailwindcss/vite`, `lucide-react`, `clsx`, `class-variance-authority`, `tailwind-merge`, `tw-animate-css`.
- Dev deps: ESLint 9 + TypeScript ESLint, `eslint-plugin-react-compiler@19 beta`, `eslint-plugin-astro`, Prettier + `prettier-plugin-astro`, Husky and lint-staged.

## Project scope
- Core user flows (MVP):
  - Upload PDF with a text layer (≤ 100 MB) after accepting TOS; rejects files without text layer and beyond limits.
  - Asynchronous extraction of: recipe title, ingredients (with amounts if detected), preparation steps, and macronutrients if present; each field with a confidence score.
  - Store results tied to the user along with the original PDF; user sees processing status in UI (queued, processing, failed, completed) with timestamps and retry (bounded).
  - Manage recipes: list with confidence labels, inline editing (title, macros, ingredients, steps), AI-assisted categorization (with confidence), ingredient categorization.
  - Weekly meal planning UI (days × meals), save/edit plans.
  - Generate shopping lists from plans: aggregate and sum ingredient quantities, group by category; allow manual edits before use.
  - Accounts & security: email+password auth, per-user data isolation, encrypted storage for PDFs and extracted content.
  - Operations & telemetry: accuracy metrics per field, processing latency, retries, user corrections.

- Out of scope (MVP boundaries):
  - No OCR for image-only PDFs.
  - No external recipe imports (web scraping).
  - No data export (CSV/JSON/Sync).
  - No native mobile app (responsive web only).
  - No social features (sharing, ratings, comments).
  - No advanced dietary personalization.

Refer to `.ai/prd.md` for detailed user stories and acceptance criteria.

## Project status
- Status: MVP in development.
- Targets (examples):
  - Extraction quality goals on a control set: title 70%, ingredients 60%, macros 50%.
  - Latency target: < 2 minutes for ~20–30 MB PDFs.
  - Operational KPIs: successful retries > 80% (max 3), number of weekly plans and shopping lists created, user opt-in rate for anonymous corrections.

Badges (examples):
- Node: 22.14.0
- License: MIT

## License
MIT. See `LICENSE` if present in the repository.
