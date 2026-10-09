# HealthBridge – No ChatGPT sign-in

The app uses **only HealthBridge username/email and password**. The old ChatGPT mock-auth plugin is disabled, and old auth URLs now redirect to the HealthBridge form.

1. Open the `HealthBridge-Fullstack` folder containing `package.json`.
2. Run `pnpm install`
3. Run `pnpm run local:setup` for local D1/R2 setup.
4. Run `pnpm build`
5. Run `pnpm start`
6. Use the exact local URL shown in your terminal, adding `/portal.html` if needed.
7. Select **Register** to create a patient account, then **Sign in** with username/email and password.

Doctor accounts require approval. Staff accounts need administrator provisioning. The ChatGPT-related URL is no longer a sign-in flow.

If you continue seeing an old page, stop old servers (`Ctrl+C`), use the updated extracted directory, restart the application and hard refresh (`Ctrl+Shift+R`). Do not use a hosted ChatGPT-preview URL for testing local credentials.
