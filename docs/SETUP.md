# Setup guide

Estimated time: 30–45 minutes. You need an n8n instance, a Google account, a Telegram bot and a Google Gemini API key.

## 1. Create the Google Sheet

Create one spreadsheet with **five tabs**. Tab names and header names must match exactly (they are Arabic because the form and the sheets are used in Arabic).

### Tab `سجل المتقدمين` (Applicants log)

Written by the workflow for every new application and used for the duplicate check.

| Header | Content |
|--------|---------|
| `الاسم الكامل` | Full name |
| `البريد الإلكتروني` | Email |
| ` رقم الهاتف` | Phone number |
| `سنوات الخبرة` | Years of experience |
| `الوظيفة المتقدم لها` | Applied position |
| `تاريخ التقديم` | Application date, `yyyy-MM-dd` |
| `رابط CV` | Link to the CV in Drive |

> **Note:** in the current export the *Append* node writes the phone number to a column named with a **leading space** (` رقم الهاتف`). Either create the header exactly like that, or edit the column name in the node mapping and in its schema to `رقم الهاتف`.

> **Phone numbers as text:** phone values are written with a leading apostrophe (`'0912...`) so Google Sheets does not drop the leading zero. Check on a test submission that the cell shows the number without a visible apostrophe; if the apostrophe is visible, set the node option *Cell Format* to *Let Google Sheets format* or remove the apostrophe in the three places that add it (the applicants-log mapping and the *parse AI result* code node).

### Tab `الوظائف الشاغرة` (Open positions)

One row per job. It must contain a column named `اسم الوظيفة` whose values equal the options of the form's dropdown (`مطور Backend`, `مطور Frontend`, `مصمم UI/UX` in the export). Every other column is free-form criteria that is passed to the AI as JSON, for example required skills, minimum years of experience, preferred education. Change the criteria in the sheet without touching the workflow.

### Tabs `المقبولون`, `قيد المراجعة`, `المرفوضون` (Accepted, Under review, Rejected)

All three use the same headers:

`الاسم` · `البريد الإلكتروني` · `رقم الهاتف` · `الوظيفة المتقدم لها` · `سنوات الخبرة` · `المهارات` · `المؤهل العلمي` · `درجة المطابقة` · `الحالة` · `سبب القرار` · `تاريخ التقديم` · `GitHub` · `LinkedIn` · `روابط أخرى` · `رابط CV`

## 2. Create the Drive folder

Create a **private** folder for the uploaded CVs. Do not enable link sharing.

## 3. Create credentials in n8n

| Credential | Used by |
|------------|---------|
| Google Sheets OAuth2 | All Google Sheets nodes |
| Google Drive OAuth2 | CV upload |
| Google Gemini (PaLM) API | AI evaluation |
| Telegram API (bot token from @BotFather) | Daily summary and failure alerts |

## 4. Import and configure

1. In n8n choose **Import from file** and select `workflow/cv-screening-workflow.json`.
2. Attach the credentials to each node that shows a missing credential.
3. Replace the placeholders:

| Placeholder | Where | Replace with |
|-------------|-------|--------------|
| `REPLACE_WITH_GOOGLE_SHEET_ID` | Every Google Sheets node | Your spreadsheet ID (the long string in its URL) |
| `REPLACE_WITH_DRIVE_FOLDER_ID` | *Upload CV to Drive* node | Your private folder ID |
| `REPLACE_WITH_TELEGRAM_CHAT_ID` | Both Telegram nodes | The chat id that should receive reports |

4. Open the workflow **Settings** and set your **Timezone**, so the 23:55 schedule and the date stamps match your local time (the export deliberately contains no timezone).
5. Edit the dropdown options in the application form node to match the rows of `الوظائف الشاغرة`.
6. Add a privacy notice to the form description (see [SECURITY.md](../SECURITY.md)).
7. Activate the workflow and open the production form URL (path `cv-application`).

## 5. Test

1. Submit the form with a test PDF; check the new row in the applicants log and the file in Drive.
2. Confirm a row appears in one of the three result tabs with a score and reasoning.
3. Submit the same email for the same job again; the workflow must stop as a duplicate.
4. Run the 23:55 schedule branch manually and check the Telegram summary.

## Troubleshooting

- **Sheet node errors about columns:** header names differ from the list above, including the leading-space quirk.
- **No criteria reach the AI:** the form option does not match any value in the `اسم الوظيفة` column exactly.
- **Empty CV text:** the PDF is a scan (image only). Text extraction needs a PDF with selectable text.
- **Candidates land in *Under review* with a note about an unknown decision:** the AI returned a value other than the three allowed ones; this is intentional so nobody is rejected by mistake.
- **Telegram alert arrives instead of a result:** a Sheets or Drive step failed; read the message and check the credential and sharing permissions.
