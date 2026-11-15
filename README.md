# Signal — ECE 251 Study Studio

A standalone digital design study website with 13 concise lessons, worked examples, 13 interactive labs, revealable explanations, dark mode, and browser-local progress.

## New: textbook-grounded AI tutor

Click **Ask the textbook** for follow-up tutoring about the current topic. The AI service must be configured first: follow **CHATBOT_SETUP.md**. This update includes the chat interface and a Cloudflare Worker backend; it does not include API credentials or an already-indexed textbook. The tutor uses the 2023 John D. Carpinelli textbook requested by the user, separate from the existing lecture-note summaries. The original lessons remain usable without the backend.

When replacing the existing site, also upload `chat.js`, `chat.css`, `chat-config.js`, `CHATBOT_SETUP.md`, and `ATTRIBUTION.md` next to `index.html`. Keep the `backend` folder for deployment from your computer. Never put an API key or tutor access code into a public source file.

## Open locally

Extract the ZIP. Keep the front-end HTML, CSS and JavaScript files together. Double-click `index.html` to open it in Chrome or Edge. No installation or build is required for the lessons and labs. Google Fonts is optional: system fonts are used when offline. The original labs run locally; only the new AI tutor requires an API and backend. Lab state resets when you change topics; explored-topic progress and theme are saved in your current browser when storage is available. Connected chat is restricted to your published site's HTTPS origin.

## Deploy to GitHub Pages — browser-only method

1. Sign in at https://github.com and create a new repository named `ece251-study-studio`. For GitHub Free, use a public repository.
2. Open the repository and choose **Add file → Upload files** (or the upload link on a new repository).
3. Upload the extracted files, including `index.html`, `styles.css`, `app.js`, and `README.md`. Put them directly at the repository root. Do not upload the ZIP itself or put the enclosing folder above `index.html`. If your file picker shows `.nojekyll`, include it too; this site also works without it.
4. Commit the files to `main`.
5. Open **Settings → Pages**.
6. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
7. Select **main** and **/(root)**, then **Save**.
8. Wait for the Pages deployment to finish. Settings → Pages shows the live link, typically `https://YOUR-USERNAME.github.io/ece251-study-studio/`.

To update the site, replace the changed files and commit to the same branch. GitHub republishes automatically.

## Troubleshooting

- **404:** Confirm `index.html` is at the root of the selected branch; check the deployment in Actions.
- **Unstyled page:** Confirm `styles.css` is beside `index.html`, with exact lowercase spelling.
- **Controls missing or unresponsive:** Confirm `app.js` is beside `index.html`, JavaScript is enabled, and open the deployed website rather than GitHub's source-code preview. Hard-refresh with Ctrl+Shift+R after replacing files.
- **Local downloaded file does not run:** Extract the ZIP first; do not open the page from inside the compressed archive.

## Files

- `index.html`: page structure
- `styles.css`: responsive layout and light/dark themes
- `app.js`: lesson content and all interactive logic
- `.nojekyll`: optional static-site marker

## Content and scope

Based on the previously supplied ECE251 Student Notes ver6, printed pages 53–225, starting with combinational circuits. Summaries are paraphrased and examples include both notes-derived and original teaching examples. The original PDF is not redistributed. This is an independent study companion, not an official NJIT publication.

Conventions: active-high routing examples, edge-triggered flip-flop simulation, unsigned register/comparator examples, and an idealized relative-delay hazard model. The asynchronous lab enforces settling before another external input change. State-machine ROM sizing assumes D flip-flops. Worked simplification uses minterms with A as MSB.

## GitHub reference

https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
