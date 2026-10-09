# HealthBridge username/email + password login

The app now uses `/api/hb/signup`, `/api/hb/login`, `/api/hb/logout` and a server-side session stored in Cloudflare D1. It does not require ChatGPT sign-in.

From the **HealthBridge-Fullstack** folder (the folder containing package.json):

```powershell
pnpm install
pnpm run local:setup
pnpm build
pnpm start
```

Open the localhost URL displayed by the server and choose **Sign in** or **Register**. New patient accounts can enter immediately; doctor registrations await approval. Hospital staff/admin accounts must be provisioned by an administrator (the sign-up form cannot grant these roles).

Passwords are stored as salted PBKDF2 hashes, with HttpOnly SameSite cookies and random database-backed session tokens. Use fictional health data for this prototype; production deployment requires a full security assessment, brute-force controls, account recovery, credential onboarding for staff, and HTTPS. Existing ChatGPT-based accounts will not automatically have passwords; register a new local account.
