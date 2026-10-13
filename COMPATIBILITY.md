# Windows 7 compatibility

ORBIT’s target computer is a Windows 7 32-bit laptop with 4 GB RAM. Compatibility is a requirement, not an assumption. This file records what was checked on 13 October 2026, and what was **not** tested.

No desktop package has been built. No claim below means the app has been seen running on that laptop.

## Candidate runtime — not selected

| Item | Value | Source |
| --- | --- | --- |
| Runtime | NW.js 0.72.0 | Last NW.js release that can sit on Chromium 109 |
| Architecture | `win-ia32` | 32-bit Windows build exists |
| Chromium | 109.0.5414.87 | [NW.js 0.72.0 changelog](https://github.com/nwjs/nw.js/blob/nw72/CHANGELOG.md) |
| Bundled Node | 19.3.0 | Updated in NW.js 0.71.1 and not changed in the 0.72.0 changelog entry |
| Normal zip | `nwjs-v0.72.0-win-ia32.zip`, 115,866,607 bytes | HEAD request on 13 October 2026 returned HTTP 200 |
| SDK zip | `nwjs-sdk-v0.72.0-win-ia32.zip`, 152,752,545 bytes | Same check. SDK includes DevTools |
| Normal URL | https://dl.nwjs.io/v0.72.0/nwjs-v0.72.0-win-ia32.zip | Redirects to `dl.node-webkit.org` |
| SDK URL | https://dl.nwjs.io/v0.72.0/nwjs-sdk-v0.72.0-win-ia32.zip | Same |

The official 0.72.0 blog lists x64 downloads and points other platforms at the directory above. The ia32 archives were present when this note was written. Presence of a zip is not a test on Windows 7.

Google’s Chrome enterprise requirements state that Chrome 109 is the last release supporting Windows 7, and that Chrome 109 shipped on 10 January 2023: https://support.google.com/chrome/a/answer/7100626

NW.js 0.73.0 does not launch on Windows 7. A reported failure, reproduced by the reporter on 32-bit and 64-bit normal and SDK builds, is:

`The procedure entry point SetProcessMitigationPolicy could not be located in the dynamic link library KERNEL32.dll.`

Issue: https://github.com/nwjs/nw.js/issues/8029

That is evidence against 0.73.0 and newer. It is not a test that 0.72.0 is comfortable on a 4 GB 32-bit machine.

## Electron was checked and not selected

Electron 22.0.0 is the last major that supports Windows 7, 8, and 8.1. It uses Chromium 108 and Node 16.17.1. Electron 23 contains Chromium 110 and drops those operating systems. Electron 22.x support ended in 2023; the project extended security backports only through 10 October 2023.

On 13 October 2026, `electron-v22.3.27-win32-ia32.zip` answered HTTP 200. That shows a 32-bit build was published. It does not show that this application runs on the target laptop. Electron 22 is older than the NW.js 0.72 Chromium 109 candidate, and it is also end-of-life. No `electron`, `electron-builder`, or `electron-packager` dependency is installed. Adding one would not make the runtime verified.

Sources:

- https://www.electronjs.org/blog/electron-22-0
- https://www.electronjs.org/blog/windows-7-to-8-1-deprecation-notice
- https://github.com/electron/electron/releases/download/v22.3.27/electron-v22.3.27-win32-ia32.zip

## Why this was not packaged

- The candidate has not been launched on the target laptop.
- A 32-bit Chromium process has a limited address space, commonly about 2 GB. NW.js plus Windows 7 on 4 GB RAM may be tight. That risk is unmeasured here.
- Chromium 109 and Windows 7 no longer receive security fixes. Packaging would ship that exposure. It should be a conscious choice after a real run, not a default.
- `package.json` in this repository is metadata only. It is not an NW.js manifest.

## Do not install a separate Node.js on the laptop

Opening `index.html` does not need Node. Tests in `tests/run-tests.js` are for a development machine.

NW.js 0.72.0 bundles Node 19.3.0 inside its own process. That is not the same as installing the Node 19 Windows installer. Node’s own Windows 7 testing ended years earlier; community reports put the last installer-tested line around Node 13.x. A modern Node installer is not a verified dependency for this project, so the setup instructions do not ask for one.

## Browser shell versus desktop shell

| Capability | This milestone | Later desktop milestone |
| --- | --- | --- |
| Draw the interface | Yes, in a Chromium-class browser | Only after an NW.js build is actually launched |
| Remember theme and catalog | `localStorage`, if the browser allows it | Still not a secret store |
| Read a project folder | Refused | Not implemented. Must stay inside the selected project |
| Call a model | OpenRouter only, from the page, after a saved key and selected model. Other providers refuse | Still no OS credential store in this candidate |
| Run a command | Not available | Not implemented. Requires confirmation |
| Internet Explorer | Unsupported | Not the target engine |

The UI says “Browser preview” unless `nw.App` or `process.versions.nw` is present. Even if NW.js is detected, disk and commands stay off. OpenRouter is called with `fetch` from the page, not through Node. A saved key remains in `localStorage`.

## Interface constraints for Chromium 109

Kept out of the CSS and scripts so an older-than-current engine is not required:

- CSS nesting
- `color-mix()`
- `dvh` / `svh` units
- `Array.prototype.toSorted`, `Object.groupBy`, and other post-109 methods
- Remote fonts and CDN scripts

CodeMirror 5 is vendored under `vendor/codemirror/`. Inter, Geist, JetBrains Mono, and a Material Symbols subset are vendored under `assets/fonts/`. The page does not load those from a CDN.

## Security limitations of the legacy stack

These limits remain even after a successful package:

- Windows 7 extended support ended on 14 January 2020. The operating system does not receive security updates.
- Chrome 109 / NW.js 0.72.0 does not receive current Chromium patches. Vulnerabilities found since early 2023 are unfixed in that runtime.
- TLS stacks and certificate stores from 2023 can fail against newer server requirements. A connection error in a future build must be shown as a failure, not replaced with a fake reply.
- Local malware on an unpatched system can read files, memory, and browser storage. An API key on this laptop should be treated as exposed to the machine.
- `localStorage` is not encrypted. It stores a display name, theme, catalog, activity, and, if you save one, the OpenRouter key. That key storage is not an OS credential store.
- A desktop-side request component is the right place for later secrets, because a publicly hosted page must not hold them. Moving the key into NW.js does not repair the operating system.
- There is no promise of BitLocker, Credential Manager isolation, or a safe update channel.
- Confirm destructive file and process actions in later milestones. This milestone cannot perform them.

## Unsupported until tested

- NW.js 0.73.0 or any current NW.js release
- Electron, including 22.3.27, until a 32-bit Windows 7 launch is recorded
- React, TypeScript, or a compile step
- A packaged `.exe`
- GPU-heavy effects, remote fonts, and extra background processes
- Any statement that the 4 GB laptop “runs ORBIT” 

Re-test on the real machine before changing the candidate from “identified” to “selected”.
