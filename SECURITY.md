# Security Policy

## Reporting a vulnerability

Please do **not** open a public issue for security problems. Use GitHub's private reporting: **Security → Report a vulnerability** on this repository. Include steps to reproduce and the potential impact.

## Security model

### Secrets and identifiers
- The workflow export contains **no secrets**: every credential reference is the placeholder `REPLACE_ME`.
- Identifiers that point at private resources were replaced with placeholders: the Google Sheet ID (`REPLACE_WITH_GOOGLE_SHEET_ID`), the Drive folder ID and URL (`REPLACE_WITH_DRIVE_FOLDER_ID`) and the Telegram chat id (`REPLACE_WITH_TELEGRAM_CHAT_ID`). Even though these are not passwords, they identify where applicant data lives.
- Real credentials (Google OAuth, Gemini API key, Telegram bot token) belong only in n8n's credential store.
- Never commit `.env` files, credential exports (`n8n export:credentials`), real sheet or folder links, or applicant data. The `.gitignore` and the CI secret scan help catch mistakes.
- If a secret is ever pushed, **rotate it immediately** (revoke the Telegram token via @BotFather, regenerate the Gemini key, revoke the Google OAuth grant). Deleting the commit is not enough.

### Applicant data (personal data)
The workflow processes names, emails, phone numbers, CV contents and CV files.
- **Storage:** Google Sheets (applicant log and result tabs) and a Google Drive folder. Keep the sheet and folder **private**, shared only with the people who need them, and never "anyone with the link". The sheet stores the Drive link to each CV, so Drive permissions are the real access control.
- **Third-party processing:** the full CV text, name, email, phone and stated experience are sent to Google Gemini for evaluation. Review the provider's data-processing terms and choose a plan or setting that matches your privacy requirements.
- **Telegram:** the daily digest and error alerts go to one Telegram chat and include names, phone numbers and candidate links. Use a private chat or a small group.
- **Consent and transparency:** the form should tell applicants who processes their data, for what purpose, how long it is kept and that an automated tool assists the screening. Add this notice to the form description before using it with real applicants.
- **Retention and deletion:** define how long applications are kept, and delete a candidate on request from the applicant log, the three result tabs and the Drive folder.

### Application and AI risks
- **Prompt injection:** the CV text is untrusted input passed to the model. A malicious CV may try to force a high score. The model's output only fills spreadsheets and never triggers messages to applicants, and the workflow stores its reasoning, but a human must review decisions.
- **Automated decisions and bias:** AI scoring can be inaccurate or biased. Treat the score as advice, review rejected and borderline candidates manually, and comply with the hiring and data-protection laws that apply to you.
- **Public form abuse:** the application form is reachable by anyone who knows its URL. Anyone can submit PDFs, which consume storage and model usage. Keep the production URL controlled, consider a CAPTCHA or a reverse-proxy rate limit, and set quotas and spending limits on the Gemini account.
- **Input validation:** the form accepts only PDF files by type filter, which is client-side. Do not open uploaded CVs on a machine that holds sensitive data without scanning them.
- **Duplicate check:** it relies on the email field, which is not verified. It prevents accidental duplicates, not deliberate spam.

## Hardening checklist before going live

- [ ] Replace all `REPLACE_*` placeholders only inside n8n, never in this repository.
- [ ] Restrict the Google Sheet and Drive folder to named users.
- [ ] Add a privacy notice to the application form.
- [ ] Set a spending limit and quota on the Gemini API key.
- [ ] Make sure a person reviews every rejected and review-tier candidate.
- [ ] Enable GitHub **Secret scanning**, **Push protection** and **Private vulnerability reporting** (Settings → Code security).
