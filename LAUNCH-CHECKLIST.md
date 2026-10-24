# JARVIS launch assessment

**Assessment date:** 23 October 2026  
**Target launch date:** 5 November 2026  
**Scope:** The deployable static site is the inner `JARVIS/` directory. The outer directory is a stale duplicate; some shared pages match, but the main page, chat, checkout, and JavaScript differ. Changes should target the inner directory unless deployment layout is clarified.

This is an evidence-based readiness assessment, not a claim that the site is production-ready. “Implemented” describes source code; “Tested” lists only checks actually run.

## Implemented (source present; end-to-end behavior not yet verified)

- Static HTML/CSS/vanilla-JavaScript pages exist for home, chat, dashboard, settings, profile, login, signup, checkout, about, and contact.
- Existing dark blue/glass visual styles, responsive CSS, logo, and favicon are present.
- Browser-local conversations and chat actions are implemented in `js/chat.js`: create/open, search, rename, pin, duplicate, archive, export, and delete. Storage is still `localStorage`, not IndexedDB.
- A separately deployable Express API exists in `backend/server.js`. It uses the Google GenAI SDK, reads `GEMINI_API_KEY` from the server environment, and defaults to the documented `gemini-3.8-flash` model.
- `POST /api/images/generate` uses the SDK Interactions API with configurable `GEMINI_IMAGE_MODEL` (default `gemini-nano-banana-2.1`) and returns generated image data only when the provider actually returns an image.
- The chat page includes a real image prompt/loading/error/preview/download UI and browser speech-recognition controls with permission/unsupported-browser feedback.
- The API source includes request-body limits, input validation, CORS origin allowlisting, Helmet headers, separate chat/image IP rate limits, and safe provider error mapping.
- `js/config.js` leaves the frontend API URL blank by default. Chat has no simulated assistant reply and displays an offline/demo state when no API is configured.
- `.gitignore` excludes `.env` and `node_modules` while allowing the placeholder `backend/.env.example`.
- Browser speech input and speech synthesis use native browser APIs when supported.
- Theme and preference controls, JSON import/export, dashboard summaries, and local profile editing have frontend implementations.

## Tested (verified during this assessment)

- Opened all 10 HTML pages in the browser and observed no JavaScript page errors after correcting the checkout page's implicit `selectedPlan` global.
- Checked local `href`/`src` references and script paths: no missing local targets were found.
- Checked local `#section` references against IDs: no broken section anchors were found.
- Ran `node --check` over the frontend JavaScript files and backend `server.js`; all passed.
- Ran `npm ci` using Node.js 22 and `npm ls --depth=0`; dependencies match `backend/package-lock.json`.
- Ran the API without a Gemini key: `/health` returned `200` with `ready: false`; chat and image requests returned `503` with `ai_not_configured`; invalid chat/image prompts returned `400`; six image requests from the same IP produced a `429` rate limit response.
- Confirmed the configured development origin receives `Access-Control-Allow-Origin`; an unapproved origin received no CORS allow header.
- Served the static site over localhost HTTP, verified the image stylesheet loaded, exercised the image form in offline mode, and confirmed no generated preview/download is exposed when the API is not configured.
- Checked chat layout at 390, 768, and 1280 pixels; no horizontal document overflow was observed.
- Image model documentation: Google currently lists `gemini-nano-banana-2.1` as a stable image model and documents `interactions.create()` returning `output_image.data`. This confirms the documented API/model path, not account access or successful image generation.
- Confirmed the frontend chat with no API URL adds the user's message locally and displays an offline notice rather than inventing an assistant response.
- Confirmed no real API key is present in the project files inspected. No replacement key was configured and no real Gemini request was made.

## Untested (must verify before launch)

- Full browser workflows for conversation create/open/search/rename/pin/archive/duplicate/export/delete, edit-and-resend, regenerate, and cancellation; storage quota failures and corrupted-data recovery.
- Migration and backup/restore for real legacy `jarvis.v2` data; no IndexedDB migration exists yet.
- Touch/keyboard accessibility and all responsive pages at representative mobile, tablet, and desktop sizes (only chat overflow was checked at 390/768/1280 pixels).
- Settings persistence, language changes, all page navigation in a hosted (rather than `file://`) environment, and every form's validation and error states.
- Real Gemini authentication, model availability, quota behavior, costs, provider response parsing, production CORS, and deployed HTTPS behavior.
- Real image-generation access, returned bytes/MIME type, preview, and download for the configured Google account. The UI/API are implemented but have not been exercised with a real provider key.
- Microphone permission-denied and successful speech-recognition flows on physical browsers/devices. The UI now reports unsupported browsers and recognition permission errors; microphone access was not requested during testing.
- Hosting compatibility, clean URLs, redirects, security headers, and HTTPS behavior on the actual InfinityFree account/domain.
- Complete privacy/terms wording and privacy review for the actual hosting and provider setup.

## Blocked or not implemented

- **Gemini production service:** no replacement API key, deployed API, or tested provider request. The frontend API base URL remains blank because no Render URL has been created. Keep chat and image generation visibly offline until these exist.
- **InfinityFree backend:** no Node.js support evidence or hosting configuration was found. Do not deploy the Node service there unless the actual plan explicitly supports it. Use a separate Node.js 20+ host if necessary.
- **Discord:** no real Discord invite URL exists anywhere in the inspected project. Do not invent one; obtain the intended invite before linking it.
- **Clean URLs:** no `.htaccess` or other rewrite configuration exists. Preserve the working `.html` URLs until the actual server's rewrite support and document root are confirmed.
- **Privacy and Terms:** `privacy.html` and `terms.html` do not exist. Homepage legal links currently point to `#`.
- **Authentication:** login and signup wait briefly, save profile data in the browser, and display success without authenticating or creating an account. Google/GitHub controls are placeholders.
- **Payments:** checkout is not a real payment flow. It currently collects card number, expiry, CVV, and billing details even though it does not process payment; do not publish or use it until those fields are removed or a legitimate PCI-compliant provider flow is implemented.
- **Contact:** the contact form has no sending backend or submit handler; it cannot deliver a message.
- **Memory and storage:** there is no user-editable saved-memory feature or IndexedDB. The existing `memory` preference does not implement AI memory. `app.js` silently falls back to defaults on malformed localStorage and writes state during initialization, risking replacement of recoverable user data. Storage failures are not surfaced consistently.
- **Streaming, document analysis, cross-device sync, account-backed persistence, and server-side authentication are not implemented.**
- **Placeholder links/actions:** homepage and contact social links (including Discord) use `href="#"`; homepage Privacy/Terms/Cookies links are placeholders. `app.js` routes literal `#` links to a generic backend-pending modal. The “Watch Demo” and “Compare Plans” buttons also have no real destination or demo.
- **Duplicate/stale copies:** the outer project tree duplicates the inner tree but diverges in critical files. Do not assume fixes in one copy update the deployable copy.

## Deployment configuration currently present

The static frontend may be served from the site's document root using the existing `.html` pages. No production host configuration, `.htaccess`, or rewrite rules are present.

For a separate Node.js 20+ backend, set these variables in the backend host's environment/secret manager:

- `GEMINI_API_KEY` — a newly issued server-side key; never reuse an exposed key or commit a real value.
- `GEMINI_MODEL` — optional; defaults to `gemini-3.8-flash`.
- `GEMINI_IMAGE_MODEL` — optional; defaults to `gemini-nano-banana-2.1`.
- `PORT` — optional; defaults to `3000`.
- `ALLOWED_ORIGINS` — exact comma-separated frontend origins. Keep the production origin and remove local development origins from the production value.

Only after deployment and testing, set `API_BASE_URL` in `js/config.js` to the backend's HTTPS origin. Leave it blank while no backend is deployed.

## Local backend setup

From the inner project directory:

```powershell
Set-Location .\backend
npm ci
Copy-Item .env.example .env
# Edit .env and set a new GEMINI_API_KEY; keep the file private.
npm run dev
```

Serve the project directory separately on `http://localhost:5500`, configure `js/config.js` with `http://localhost:3000` for local-only development, then check `http://localhost:3000/health`. For production, follow `backend/README.md`, set `API_BASE_URL` to the actual Render HTTPS origin only after deployment, and use exact allowed origins.

## Next stages

1. Remove the checkout card-data fields and replace fake login/signup success and contact submission with clearly unavailable, non-collecting states.
2. Replace dead/placeholder links; ask for the real Discord invite rather than fabricating one. Add accurate privacy and terms draft pages.
3. Protect and migrate existing local chat data; add tested IndexedDB persistence, import/export, and explicit storage-error recovery.
4. Implement opt-in local memory controls only after the storage design and migration are tested.
5. Verify InfinityFree's actual plan capabilities, deploy the backend to Render or another compatible Node host, then test with a newly issued Gemini key. Do not call this live AI before real chat and image requests succeed.
6. Check the Google AI account's image model access/quota and verify a real returned image can be previewed and downloaded.
7. Run full interaction, responsive, accessibility, security, and deployment tests; resolve launch blockers before 5 November 2026.
