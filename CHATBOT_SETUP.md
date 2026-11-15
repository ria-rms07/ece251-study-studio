# Add the textbook tutor to your existing GitHub Pages site

## What is ready, and what you need to connect

This update includes the full study site plus an AI chat panel and backend code. It is **not an already-running AI service**. Your existing lessons and labs still work without any AI configuration. The tutor needs your OpenAI API project, the actual textbook PDF indexed in that project, and a deployed Cloudflare Worker. The API key belongs only on the backend, never in GitHub or the page.

The tutor is configured for **John D. Carpinelli, An Animated Introduction to Digital Logic Design (2023)**, the exact edition linked in your request:

https://digitalcommons.njit.edu/cgi/viewcontent.cgi?article=1000&context=oat

Listing and license: https://digitalcommons.njit.edu/oat/1/

Do not substitute the newer edition linked from the listing unless you intentionally update the tutor's attribution and source URL too. The PDF is not included in this ZIP because its download could not be retrieved in the build environment. Download it yourself using the link above. The uploader asks you to confirm the book; it does not automatically authenticate the PDF's identity.

## 1. Extract and upload the website files

Extract the ZIP. Upload these **nine files** to the top level of your existing GitHub repository, replacing the existing files with the same names:

- `index.html`
- `styles.css` (exactly this spelling)
- `app.js`
- `chat.css`
- `chat.js`
- `chat-config.js`
- `CHATBOT_SETUP.md`
- `README.md`
- `ATTRIBUTION.md`

Keep the `backend` folder on your computer for now. Do not upload any downloaded PDF, API key, access code, `.dev.vars`, or generated `textbook-index.json`. The backend source itself contains no secrets, but GitHub Pages does not run it.

The new **Ask the textbook** button will appear after the Pages deployment completes. It will ask for a connection until the steps below are done. This is expected; it does not pretend to answer without textbook access.

## 2. Create your API project and download the PDF

1. Open https://platform.openai.com/ and create/select an API project. Check its billing and usage controls. API usage is separate from this site's hosting; this code does not use your ChatGPT session as an API key.
2. Create an API key with access to Files, Vector Stores, and Responses in that project. Keep it private. You will paste it into hidden terminal prompts, not this chat or GitHub.
3. Download the linked **2023 textbook PDF** and note its location.
4. You need Python 3.10+ and Node.js 22+ with npm on your computer. If needed, install from https://www.python.org/downloads/ and https://nodejs.org/ .

## 3. Index the textbook (once)

Open the extracted `backend` folder in Windows File Explorer. Click its address bar, type `powershell`, and press Enter. This opens PowerShell in that folder.

Run, replacing the example PDF path with the location of your downloaded PDF:

```powershell
python upload_textbook.py "C:\Users\riasa\Downloads\YOUR-TEXTBOOK-FILENAME.pdf"
```

Confirm that the file is the correct book. Paste your OpenAI API key when prompted; the characters will not appear. The script uploads the PDF and waits for indexing. It prints two IDs:

- `VECTOR_STORE_ID`, beginning with `vs_`
- `TEXTBOOK_FILE_ID`, beginning with `file-` or `file_`

Keep these for the next step. The IDs and PDF checksum are saved locally to `textbook-index.json`, without your API key. If the connection breaks, rerun the same command to resume from saved IDs. A failed API attachment may need to be checked in the OpenAI dashboard before retrying.

## 4. Deploy the private-key backend

Create/sign in to a Cloudflare account at https://dash.cloudflare.com/ . From the same `backend` PowerShell window, run:

```powershell
npx wrangler@4 login
npx wrangler@4 deploy
```

The first command opens Cloudflare sign-in in your browser. The second deploys the included `worker.mjs` with the settings in `wrangler.toml`. Accept npm's prompt to install Wrangler if shown. The initial deployment will return a setup-required error until its secrets are added.

Now run each command separately and paste its value when prompted:

```powershell
npx wrangler@4 secret put OPENAI_API_KEY
npx wrangler@4 secret put VECTOR_STORE_ID
npx wrangler@4 secret put TEXTBOOK_FILE_ID
npx wrangler@4 secret put TUTOR_ACCESS_CODE
```

Use your API key for the first prompt, the two IDs printed above for the next two, and a **new private access code of at least 16 characters** for the final prompt. This last code is for unlocking your tutor; it is NOT your API key. Use a password manager to generate it, and keep it out of your source files.

Deploy once more after adding the secrets:

```powershell
npx wrangler@4 deploy
```

Copy the Worker URL printed in the terminal. It will look like `https://signal-textbook-tutor.YOUR-SUBDOMAIN.workers.dev`. This is a public endpoint address, not a secret. The Worker requires your private tutor code before it can spend API credits.

The included `ALLOWED_ORIGIN` is `https://ria-rms07.github.io` (no repository path and no trailing slash). If you change your GitHub username or domain, edit that value in `wrangler.toml` and redeploy. Local `file://` origins are intentionally not allowed for real API calls; use the published GitHub Pages site for connected chat.

## 5. Connect the study site

1. Open https://ria-rms07.github.io/ece251-study-studio/ .
2. Click **Ask the textbook → Connect**.
3. Paste the Worker URL and enter your private **tutor access code**. Never enter your API key in this panel.
4. Click **Use this connection**.
5. Ask “What is the difference between a half adder and a full adder?”
6. Expand a supporting passage beneath the answer to inspect the retrieved textbook text. Then ask a follow-up such as “Why is the carry 1 when both inputs are 1?”

For an endpoint prefilled for you on any browser, edit `chat-config.js`, paste the Worker URL into `endpoint: ""`, and upload that file again. The access code is never stored there. The code is kept only in the open page's memory and must be reentered after a reload. Conversation history is also memory-only; New clears it. Recent turns are sent with each question so follow-ups work.

## What grounding does—and does not—mean

- Each question first searches the uploaded PDF. Only results matching your configured textbook file ID are used. No other files in the project are sent as evidence.
- The model receives retrieved passages, the current topic title, your question, and up to eight recent messages. The existing Savir lecture notes are not presented as Carpinelli textbook evidence.
- Answers use [1], [2], etc. to reference expandable passages. **These are passage IDs, not page numbers.** The PDF retrieval service does not supply verified page locations, so the app does not invent them.
- Empty retrieval or invalid citations produce a clarification/unsupported response instead of a fake sourced answer. The model is also instructed to decline explanations the passages cannot support. Valid IDs do not guarantee every generated claim is correct; inspect the excerpts when in doubt.
- Text retrieval does not reliably inspect circuit images. Ask about a named concept or transcribe a diagram's connections if its picture is essential. Follow-up tutoring is supported, but diagram image upload is not included.
- Chat is a private-code-protected study tool, not a multi-user authentication system. Do not publish the access code. Rate limiting is approximately 10 requests per minute per Cloudflare location, not a global hard spending limit. Monitor API billing and configure your own project usage controls.
- The backend requests `store: false` for generated Responses and does not deliberately log chats. Your PDF remains in your OpenAI project until deleted, and API/provider data policies still apply. Questions and recent messages travel through Cloudflare to OpenAI.

## Quick fixes

| Message | Check |
|---|---|
| No chat button | All three new JS/CSS files and updated index uploaded; hard-refresh Ctrl+Shift+R. |
| Cannot reach backend | Worker URL, deployment, and `ALLOWED_ORIGIN`; use your published HTTPS site. |
| Incorrect access code | Enter the value of `TUTOR_ACCESS_CODE`, not the API key. |
| Tutor not configured | All four secrets and the rate-limiter binding from `wrangler.toml`. |
| No supporting passage | Indexing completed; correct `TEXTBOOK_FILE_ID`; ask a specific digital-logic question. |
| Usage/billing limit | API project's billing and usage limits. |
| Model/index not found | Model availability and vector store/file IDs in the same API project as the key. |

## Validation status

Backend behavior is tested with mocked retrieval/model responses, including authentication, CORS, rate limiting, file filtering, empty results, and citation validation. The original 13 labs have script regression tests. No live AI answer is claimed: an API key, indexed PDF, and deployed backend were not available during packaging. Browser rendering could not be inspected in this environment.

## References

- https://developers.openai.com/api/docs/guides/retrieval
- https://developers.openai.com/api/reference/resources/responses/methods/create
- https://developers.cloudflare.com/workers/configuration/secrets/
- https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/
