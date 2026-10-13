# ORBIT

Multiple AI Brains. One Coding Universe.

ORBIT is a lightweight AI coding workspace. This repository is **Milestone 1**: the application shell. It is not a Windows package, and it has not been tested on the target computer.

Target computer, still unverified: Windows 7 32-bit, 4 GB RAM.

## Framework, entry point, and build

| Item | Value |
| --- | --- |
| Framework | Vanilla HTML, CSS, and JavaScript. No React, Electron app shell, TypeScript, or bundler |
| Entry point | `index.html` |
| Screen source | Eight screens. Five come from the Stitch HTML exports, the Code Editor follows its screenshot, and Dashboard and Terminal use that same shell. Markup is in `scripts/screens.js` |
| UI build command | None. There is nothing to compile |
| Test command | `node tests/run-tests.js` |
| Assemble command | `node desktop/build-candidate.js` |
| Executable | Not built. `npm run assemble` copies source. It does not create `ORBIT.exe` |

`package.json` is metadata. It has no `"main"` field. A draft NW.js manifest is `desktop/nw-manifest.json`. It is copied only into the assemble folder, and only after the script checks that the root package still says the desktop runtime is not selected.

## What works in this build

| Area | Status |
| --- | --- |
| Navigation across all eight Stitch screens | Working |
| Light, dark, and system theme | Working, stored in this browser profile |
| Density and accent | Working locally. They do not connect a cluster |
| Project catalog: create and select | Working as a **catalog only**. No folder is created or read |
| Activity history | Working, local only |
| Export and import of shell data | Working. Secret-named fields are discarded on import |
| Reset local shell data | Working. Does not delete disk files |
| Command search and Ctrl/Command+K | Working. Local commands only |
| AI Workspace send | OpenRouter sends a real request when a key is saved and a model is selected. Other providers still refuse. Failures are shown. No reply is invented |
| Agent draft, pause, abort, and approval | A draft can be saved locally. No agent runs, and approval does not change files. Saved role models are not called |
| File refresh, create, folder, and save | Call the filesystem adapter, which refuses |
| Code editor | In-memory sample buffers and CodeMirror 5. Save does not write disk |
| Terminal | Shows that the command was not executed. No shell is attached |
| Model Hub and Settings connection tests | OpenRouter uses a real request and shows Connected only after it succeeds. Other providers still refuse. Latency is shown only from that response |
| API key fields | OpenRouter can be saved in this browser profile. That storage is not secure. Other provider fields are still cleared and are not sent |

## What does not work yet

- Live calls to OpenAI, Anthropic, Gemini, or a custom endpoint
- Streaming. OpenRouter replies are requested as one JSON response
- An OS credential store. A saved OpenRouter key uses localStorage
- Reading or writing a real project folder
- Applying approved file changes
- Executing a terminal command
- A Windows desktop package
- Windows 7 compatibility. It has not been tested on the target computer

If a control cannot do the real operation, it says so. A refusal is not a simulated success.

## OpenRouter

Settings → API Keys has the OpenRouter key field. Show and Hide toggle the field. Save stores the key in this browser profile only. Remove deletes it. Test Connection calls `GET https://openrouter.ai/api/v1/key`. The status is Not configured, Testing, Connected, or Error. Connected is shown only after that request succeeds.

Model Hub → OpenRouter models loads `GET https://openrouter.ai/api/v1/models`. The list is not hardcoded. Search covers name, id, and publisher. A row shows context length and input/output prices when the endpoint sends them. Select stores that model id for AI Chat.

AI Chat posts to `https://openrouter.ai/api/v1/chat/completions` with the selected model id and the visible conversation. Planner, Developer, Code Reviewer, and Debugging Assistant can store a model id, but orchestration does not call them and they do not collaborate. Streaming is not used. Other providers still refuse, and a failed OpenRouter request does not invent a reply.

## Setup

No `npm install`. The UI has no package dependencies. Do not add API keys to this folder.

1. Keep `index.html`, `styles/`, `scripts/`, `services/`, `vendor/`, and `assets/` together.
2. Open `index.html` in a Chromium-class browser, or serve the folder:

```text
python3 -m http.server 8080 --bind 0.0.0.0
```

Then open `http://127.0.0.1:8080/`. Internet Explorer cannot run this shell.

3. On a development machine, not on the Windows 7 laptop:

```text
node tests/run-tests.js
node desktop/build-candidate.js
```

`npm test` and `npm run assemble` are the same two commands. Assemble writes `build/orbit-unpacked/` and prints that no executable was created. Details and the unverified runtime choice: `desktop/PACKAGING.md`.

`file://` is enough for this milestone because the page uses classic scripts, not ES modules, and it does not fetch other HTML files. If a browser blocks local script loading, serve the folder instead:

```text
python3 -m http.server 8080 --bind 0.0.0.0
```

Then open `http://127.0.0.1:8080/`. Do not put this folder on a public host. Anthropic, OpenAI, and the other designed key fields are cleared and are not sent. OpenRouter is the exception: Save stores that key in `localStorage`, and Test or chat sends it only to `https://openrouter.ai/api/v1`.

Optional checks, on a development machine only:

```text
node tests/run-tests.js
```

The Windows 7 laptop does not need Node.js installed for that. Do not install a current Node.js on Windows 7 unless that exact installer has been verified. It has not.

## Export the project

Copy the whole `ORBIT` directory. Do not use an older zip as the current build; it does not contain the Stitch screens.

Include:

- `index.html`
- `styles/` (`stitch.css` is the interface stylesheet)
- `scripts/`
- `services/`
- `vendor/codemirror/`
- `assets/` (mark and local fonts)
- `tests/`
- `desktop/`
- `README.md`, `COMPATIBILITY.md`, `package.json`, `.gitignore`

Do not add `.env`, `secrets.json`, or API keys before you copy it. This build does not create those files. `.gitignore` already ignores the usual secret names.

`package.json` in the repository root is metadata. It is not an NW.js manifest. Dropping this folder onto a current `nw.exe` is not a supported package. The assemble folder gets a separate draft manifest. That draft is still not a tested executable.

## Layout

```text
ORBIT/
  index.html
  package.json
  README.md
  COMPATIBILITY.md
  styles/stitch.css   Stitch visual system
  scripts/            Shell, eight screens, in-memory editor session
  services/           Provider registry, filesystem refusal, orchestration contract, local store
  vendor/codemirror/  CodeMirror 5 for the in-memory editor
  assets/             Mark and local fonts
  tests/run-tests.js
  desktop/PACKAGING.md
```

Screen markup is embedded in `scripts/screens.js`, so the shell still opens without `fetch`. Dashboard and Terminal were not in the uploaded Stitch exports. They use the same shell, type, and color system, and they do not invent cluster metrics or command output.

UI, provider logic, and filesystem logic are separate. `scripts/` never embeds an API key. `services/filesystem-adapter.js` cannot read a disk. `services/path-guard.js` is a containment check for a later desktop adapter. A path that passes the check is not permission to read a file, and nothing in this build grants that permission.

## Security

- No real API key is hardcoded. Placeholder key strings from the Stitch export were removed.
- The OpenRouter key, if you save it, is written to `localStorage` under `orbit.credentials.v1`. That is not secure and is not an OS credential store. It is not written into source files.
- Other key fields in Settings and Model Hub are not written to `localStorage` and are not sent.
- Shell export has no key field. Import drops fields named like `apiKey`, `token`, `secret`, `password`, or `credentials`.
- `localStorage` is not encryption. On a shared PC, anyone who can open this profile can read the catalog, the display name, and a saved OpenRouter key.
- Do not host this frontend publicly with secrets in a later build. Provider calls and secret handling belong in a desktop-side component, and even that does not make an unpatched Windows 7 system safe.
- Windows 7 has been out of support since 14 January 2020. Chromium 109 has had no browser security updates since early 2023. See `COMPATIBILITY.md`.

## Desktop runtime

A candidate has been identified and **not** selected: NW.js **0.72.0**, Windows **ia32**, Chromium **109.0.5414.87**, bundled Node **19.3.0**.

NW.js 0.73.0 and newer do not launch on Windows 7. Electron 23 and newer also drop Windows 7. Electron 22.3.27 has a 32-bit Windows zip, but it is end-of-life, uses Chromium 108, and has not been launched here, so it is not selected and no Electron config is in this repo.

This application has not been packaged and has not been run on the target laptop. Do not treat compatibility as promised.

Details and sources: `COMPATIBILITY.md`. Assemble steps that do not create an executable: `desktop/PACKAGING.md`.

## Language level

The interface is written so it can be tried in Chromium 109. It does not use CSS nesting, `color-mix()`, or `Array.prototype.toSorted`. Inter, Geist, JetBrains Mono, and a Material Symbols subset are files under `assets/fonts/`. The page does not call a font CDN.

## Next milestone

Milestone 2 is partly started: the dashboard, responsive layout, and application preferences are already in this shell. What remains for a real Milestone 2/3 boundary is provider configuration **without** putting secrets in a public page, then real adapters that report real errors.
