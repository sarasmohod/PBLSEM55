# HealthBridge — full-stack connected healthcare prototype

A hospital-themed website with patient, doctor and administrator portals, a shared SQL database, protected report uploads, hospital-branded PDF exports, and a report-aware assistant integration.

## Run locally in VS Code

Requirements: Node.js 22.13 or newer, npm, and a modern browser. Node 24 LTS is suitable. No Python is required.

1. Extract the ZIP and open the HealthBridge-Fullstack folder in VS Code.
2. Open Terminal > New Terminal in that folder.
3. Install dependencies:

   npm install

4. Create the local database and default local configuration:

   npm run local:setup

5. Start the full-stack development app:

   npm run dev

6. Open http://localhost:5173/portal.html.
7. Choose Sign in with ChatGPT. On localhost, the included development sign-in simulator uses seedy@sites.test. Complete the HealthBridge registration form; this configured local owner receives administrator access and fictional sample data.
8. Admin can switch between the three workspace previews. These are administrator previews, not separate patient/doctor logins. On the hosted site, each real ChatGPT identity has its own server-checked membership.

Keep the terminal open. Ctrl+C stops the app. If port 5173 is occupied, run `npm run dev -- --port 5174` and use the printed URL.

Do not open the HTML file directly or use Live Server for this version: its database, identity and API require the full-stack server.

## Hosted identity and permissions

- Hosted authentication uses Sites' Sign in with ChatGPT integration. The server trusts identity headers supplied by the hosting boundary, not an identity chosen in browser JavaScript.
- BOOTSTRAP_ADMIN_EMAIL identifies the intended hospital owner. Do not allow an arbitrary first visitor to become administrator.
- Other registrations create patient accounts or pending doctor accounts. Doctor accounts must be approved by an administrator.
- Patients can access only their own records, appointments, bills and uploads.
- Doctors can access only patients assigned to them. Uploaded reports additionally require sharing by the uploader before another doctor can read them.
- Admin can assign doctors, approve/disable accounts, manage records, invoices, hospital profile and beds, and inspect audit events.
- Every mutation and file download is authorized on the server. Same-origin checks protect write endpoints. Database uniqueness constraints prevent conflicting appointment slots and double bed assignment. Record versions reject stale edits.
- This source is designed for the Sites trusted identity boundary. Do not place the API behind a generic public proxy that forwards user-supplied identity headers. Integrate a verified identity provider before deploying elsewhere.

## Shared records and the four value propositions

1. Unified care journey: appointments, clinical notes, prescriptions, reports, billing and admissions refer to the same patient ID.
2. Living health timeline: completed consultations create timeline records, while uploaded reports, invoices and current bed assignments are visible together.
3. Connected care teams: doctor approval, patient assignment, protected sharing and server authorization connect the three workspaces.
4. Integrated operations: the same SQL database holds care records, appointment slots, bed occupancy and invoice status. Dashboards refresh every 15 seconds while idle, with a manual refresh control.

Old browser-local records are not silently uploaded. This full-stack app starts with its own shared database; fictional sample records are loaded for the configured owner. The previous localStorage dataset remains in the old browser but is no longer used for healthcare data.

## Upload and discuss a lab report

1. Open Reports > Upload report.
2. Select the patient, title, report date and a PDF, PNG, JPG or plain-text file (maximum 5 MB).
3. The original file is stored in private R2 storage. Use Download to retrieve it.
4. Choose Share with care team to allow the assigned doctor to access the upload. Sharing can be revoked. Admin retains administrative access.
5. Choose Discuss report. Select the exact report, add any relevant context, and ask a question.
6. AI mode requires a server API key and explicit per-message consent to send the report, context, question and recent conversation to OpenAI.
7. Without an API key, the assistant explicitly uses a reference-range guide. It compares manually entered values with the exact lab ranges provided by the user. It does not claim to read uploaded files.

Deleting an upload removes its stored file and metadata. Previous assistant exchanges remain in the protected conversation history; deletion of a file does not erase past chat text.

## Enable AI report explanations locally

`npm run local:setup` creates an ignored `.dev.vars` file. Add these server-side values locally:

   OPENAI_API_KEY="your-own-api-key"
   OPENAI_MODEL="gpt-4.1"

Restart the development server. Never put the key in public files, browser code, screenshots or chat messages. API usage is billed by the provider. For the hosted site, set these values as server runtime secrets through Sites, then redeploy.

The integration uses OpenAI's Responses API and accepts a selected PDF/image/text file. It sends `store: false`; this does not imply zero provider retention. Only the chosen report, user context, confirmed measurements and up to four recent exchanges for that report are included. The server limits assistant requests to 30 per user per hour.

AI mode can explain findings and discuss possible associations with uncertainty. It is not a clinically validated diagnosis or disease-prediction model, does not prescribe medicines or doses, and should not be used to change treatment. Uploaded report text is treated as untrusted content; the assistant is instructed not to follow instructions embedded in it. Live AI behavior must be tested with the configured provider before relying on it.

Sources used for the implementation:
- https://developers.openai.com/api/docs/guides/file-inputs
- https://medlineplus.gov/lab-tests/how-to-understand-your-lab-results/

## Technical stack

- Frontend: HTML5, CSS3, vanilla JavaScript; existing responsive HealthBridge design retained.
- Server: TypeScript API routes using Vinext on Cloudflare Workers.
- Database: Cloudflare D1 (SQLite), Drizzle schema and versioned migrations.
- Files: private Cloudflare R2, authorized downloads.
- Identity: hosted ChatGPT sign-in plus server-side memberships and role permissions; loopback-only development sign-in simulator locally.
- PDFs: bundled pdf-lib and HTML Canvas. Exported pages are raster pages; source uploads retain their original format.
- Assistant: server-side OpenAI Responses API integration, with an explicitly labeled reference-range fallback.

## Source map

- public/portal.html, style.css, app.js, enhancements.js: landing page and portal UI.
- public/fullstack.js: authenticated API calls, shared state, upload/sharing UI and assistant workspace.
- public/vendor/: bundled PDF library and its license.
- server/service.ts: identity, authorization, CRUD, upload/download, sharing and chat endpoints.
- server/analysis.ts: reference-range comparison and AI system instructions.
- server/seed.ts: fictional sample dataset.
- app/api/hb/[...path]/route.ts: Worker API entry point.
- db/schema.ts, drizzle/: SQL schema and migrations.
- scripts/local-setup.mjs: local build and migration setup.
- tests/backend.mjs: integration tests running the server in Miniflare with actual local D1/R2 bindings.
- .env.example: configuration names only, no real credentials.

## Verification and scope

Run `npm run typecheck` and `npm run test:backend`. The automated suite checks authorization, role escalation rejection, patient isolation, doctor approval and assignment, appointment conflicts, stale updates, consultation timeline records, uploads, sharing, downloads, deletion, invalid file rejection, same-origin enforcement, chat isolation and measurement validation.

The deployment build is checked. Browser-based visual QA and live AI-provider analysis were not available in the authoring environment. This is a full-stack demonstration, not a certified electronic medical-record system or a clinically validated diagnostic product. Hospital contact details in seeded reports are clearly illustrative and editable under Admin > Settings.

## Database and configuration troubleshooting

- Run local:setup from the project root. It tracks applied local migrations in .wrangler/healthbridge-applied.json; do not replay old migrations manually.
- Local D1/R2 data lives under .wrangler/state. Hosted data is separate.
- .dev.vars contains local runtime values and must stay private.
- Doctor cannot see patients: approve the doctor under Care team & access, then edit a patient and assign that doctor.
- Report unavailable to doctor: verify both patient assignment and the upload's sharing status.
- Assistant remains in basic mode: check OPENAI_API_KEY in .dev.vars and restart. On hosting, apply the runtime secret and redeploy.
- These files exclude runtime data, credentials, node_modules and the original Site identity, so the ZIP can be used as a standalone local checkout.
