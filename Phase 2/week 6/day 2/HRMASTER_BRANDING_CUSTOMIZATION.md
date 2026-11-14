# HRMaster Application Branding Customization

## Purpose

Customize the Resume RAG application as **HRMaster**, a professional, people-first talent search and resume management workspace.

## Brand identity

- **Organization name:** HRMaster
- **Logo:** A simple human/person symbol paired with the HRMaster wordmark. Use the same mark in the application header, assistant avatar, browser favicon, and upload workspace.
- **Suggested tagline:** People-first talent intelligence
- **Voice:** Clear, respectful, concise, and professional. Avoid technical error details in user-facing messages; keep diagnostic details in server logs.

## Brand colors

Use a consistent blue-led palette and keep status colors distinct from brand colors.

| Token | Suggested value | Use |
| --- | --- | --- |
| Primary blue | `#1D4ED8` | Primary actions, active navigation, links |
| Deep blue | `#1E3A8A` | Hover states, emphasis |
| Sky blue | `#0EA5E9` | Secondary accents and information |
| Page background | `#F1F5F9` | Application canvas |
| Surface | `#FFFFFF` | Sidebar, cards, dialogs, forms |
| Main text | `#0F172A` | Headings and primary content |
| Muted text | `#64748B` | Supporting labels and descriptions |
| Success | `#15803D` | Completed uploads and successful actions |
| Warning | `#B45309` | Partial/degraded search results |
| Error | `#B91C1C` | Validation and request failures |

Maintain readable contrast for text, controls, and status messages. Do not rely on color alone to communicate state.

## Layout and responsive behavior

- Use a clean, light enterprise interface with white surfaces, subtle borders, restrained shadows, and consistent spacing.
- On wide screens, keep primary navigation in a stable sidebar and place search or ingestion content in the main panel.
- On tablets and phones, stack navigation and content, allow controls to wrap, and keep forms and cards within the viewport.
- Support viewport widths down to 320px without horizontal page scrolling.
- Keep primary controls comfortable to tap and preserve visible keyboard focus.
- Candidate cards should wrap long names, skills, summaries, and identifiers instead of overflowing.

## User feedback requirements

### Candidate search

- **Validation:** Do not submit a blank query. Provide a clear prompt near the input when a query is required.
- **Loading:** Disable duplicate submission and announce that resume search is in progress using an accessible status region.
- **Success:** Show the result count, selected search mode, elapsed time when available, and ranked candidate cards.
- **Empty:** Explain that no candidates matched and invite the user to adjust the query.
- **Degraded:** Clearly identify fallback ranking or unavailable summaries without implying that those stages succeeded.
- **Error:** Show a concise, actionable message and a retry path when appropriate. Keep provider diagnostics out of the browser UI.

### Resume ingestion

- **Validation:** Accept PDF files only and enforce the existing 5 MB limit before upload. Identify the specific validation issue and how to correct it.
- **Loading:** Show upload progress when available and distinguish file transfer from backend processing.
- **Success:** Confirm ingestion completion and show the available resume details and identifier.
- **Error:** Explain that ingestion did not complete, retain the selected file when possible, and provide a retry action.

### Candidate profile

- Show a loading skeleton while profile data is fetched, an accessible error with retry when the request fails, and only the profile fields actually returned by the API.

## Accessibility and content

- Use semantic landmarks and headings, descriptive control labels, keyboard-operable actions, and `role="status"`/`role="alert"` where appropriate.
- Keep focus indicators visible and ensure status text meets contrast requirements.
- Use respectful, neutral hiring language. Do not infer or display protected characteristics.
- Avoid displaying candidate contact details in search cards; show them only in the existing profile view and according to product permissions.

## Suggested implementation locations

- `recruitbot-web/src/config/api.config.ts` — application name default
- `recruitbot-web/tailwind.config.cjs` and `recruitbot-web/src/index.css` — brand palette and global theme
- `recruitbot-web/src/components/layout/AppShell.tsx` — responsive shell and navigation
- `recruitbot-web/src/features/search/components/` — search header, assistant, results, candidate profile, and search controls
- `recruitbot-web/src/features/ingestion/` — ingestion branding and upload feedback
- `recruitbot-web/index.html` and `recruitbot-web/public/hrmaster-human.svg` — browser title, theme color, and human logo mark

## Acceptance checklist

- [ ] HRMaster name and human/person logo are consistent throughout the application and browser tab.
- [ ] Search and ingestion screens use the blue brand palette consistently.
- [ ] Desktop, tablet, and 320px mobile layouts remain usable without horizontal overflow.
- [ ] Search and upload validation, loading, success, empty, degraded, and error states are understandable and accessible.
- [ ] Existing search, candidate profile, resume upload, and retry flows continue to work.
- [ ] Frontend build and lint checks pass after implementation.
