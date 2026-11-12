# BUSY Bar: start here

Instructions for AI coding agents helping a person make something for BUSY Bar. This page only picks the
instructions that fit. Pick one path below, open its document and follow it. Do not mix paths.

## What BUSY Bar is

A desk display that shows the people around you that you are busy, with a focus timer built in.

- **Screens:** the front is 72x16 RGB LED pixels; the back is a 160x80 monochrome OLED with 16 shades of gray.
  Apps can draw on both.
- **Controls:** Start / Pause, Back, a scroll wheel that also presses as OK, and a five-position mode selector:
  BUSY and CUSTOM (two focus modes), sleep, APPS (where JS apps run) and SETTINGS.
- **Connections:** USB, where it answers at `10.0.4.20`, and 2.4 GHz Wi-Fi. Everything on it can be driven
  through its HTTP API.
- **Apps:** it runs small JavaScript apps itself, with 128 KiB of memory by default (up to 512 KiB in the app's
  manifest). Redrawing the whole front screen from JavaScript manages only a few frames per second, so rich
  animation goes into `.anim` files that the firmware plays.
- Details: [Tech specs](https://docs.busy.app/bar/tech-specs.md).

BUSY App, for iPhone, Android and Mac (Windows is coming), is the person's app for focus sessions, Wi-Fi and
settings. It is not how apps for BUSY Bar are made: when a person asks for "an app", they mean a JS app on
BUSY Bar unless they say a phone or desktop app.

## 1. Check first

These checks only read; run them yourself before asking anything.

- **Already in a BUSY Bar app project?** If the working folder has `src/appmeta/manifest.json`, read the project's
  own `AGENTS.md` and follow it. If the person has no BUSY Bar and that file does not mention BUSY Bar Emulator,
  also use the emulator's path from the table below, without its `create` step. Stop here.
- **BUSY Bar on USB?** `curl -s -m 3 http://10.0.4.20/api/status/firmware` (in PowerShell: `curl.exe`). JSON with
  `version` means a BUSY Bar is connected, and `version` is its firmware. No answer means none on USB; the person
  may still have one on Wi-Fi. What the person says wins: if they say they have no BUSY Bar or ask for BUSY Bar
  Emulator, take the emulator's path even when a BUSY Bar answers here.
- **BUSY Bar Emulator installed?** Its pointer file exists:
  `~/Library/Application Support/BUSY Bar Emulator/install.json` on macOS,
  `%LOCALAPPDATA%\BUSY Bar Emulator\install.json` on Windows.

## 2. Ask only what the checks cannot tell

- **What to make**, if the person did not say: an app that runs on BUSY Bar, a widget that runs on their computer,
  or a change to the firmware. When unsure, it is an app.
- **Whether they have a BUSY Bar**, if none answered on USB and they did not say: on Wi-Fi (ask its address), or
  none.

If there is nobody to ask, make an app: on BUSY Bar if one answered, otherwise on BUSY Bar Emulator.

## 3. Paths

| The person wants | BUSY Bar | Open and follow |
|---|---|---|
| An app that runs on BUSY Bar | on USB or Wi-Fi | [JS apps on BUSY Bar](https://raw.githubusercontent.com/busy-app/busybar-firmware/dev/applications_js/AGENTS.md) |
| An app that runs on BUSY Bar | none | [JS apps on BUSY Bar Emulator](https://raw.githubusercontent.com/busy-app/busybar-emulator/main/public-docs/agents.md) |
| To send a finished app to the app catalog | any | [Catalog: assisting an application author](https://raw.githubusercontent.com/busy-app/busy-apps-catalog/main/AGENTS.md), then its [CONTRIBUTING](https://raw.githubusercontent.com/busy-app/busy-apps-catalog/main/CONTRIBUTING.md) |
| A widget: a program on their computer that shows things on BUSY Bar through its HTTP API | on USB or Wi-Fi | [Python library guide](https://raw.githubusercontent.com/busy-app/busylib-py/main/AGENTS.md); walkthrough: [Widget with AI](https://docs.busy.app/bar/dev/widget-with-ai.md) and [HTTP API](https://docs.busy.app/bar/dev/http-api.md) |
| A change to the firmware itself | any | [Firmware](https://raw.githubusercontent.com/busy-app/busybar-firmware/dev/AGENTS.md): clone the repository and work in it |

An app is a JavaScript app that runs on BUSY Bar itself and goes to the app catalog. A widget runs elsewhere and
needs that computer to stay on. Native firmware apps break official updates; build one only when asked to.

## 4. When sources disagree

- **The HTTP API is what the BUSY Bar's firmware says it is.** Its OpenAPI spec for a given firmware version:
  `https://api.busy.app/busybar/openapi.yaml?Name=<version>`, for example `Name=1.3.1`. Paths there start with
  `/busybar/`; on the BUSY Bar itself they start with `/api/`. When a guide shows a field or value the spec does
  not have, the spec is right.
- **Endpoints differ between firmware versions.** Check the version first, as in step 1, and use only what that
  version has.
- The tools are the npm packages `@busy-app/create-app`, `@busy-app/cli` and `@busy-app/busy-lib`. The npm
  package named plain `busy-cli` is not ours: never install it.

## Known pitfalls

Some documents are behind the tools. Until they are fixed:

- **Catalog builds need npm or a pinned pnpm.** The catalog accepts `package-lock.json`, or `pnpm-lock.yaml` with
  `"packageManager": "pnpm@<version>"` in `package.json`. `busy-cli create` picks pnpm and does not add that field:
  choose npm, or add the field yourself. Yarn and Bun projects are refused.
- **Settings:** in a project made by `npm create @busy-app/app`, `loadValues()` from `@shared/settings` always
  returns `null`: read settings with `device.AppsSettingsGet({ app_id })` from `@busy-app/busy-lib`. A project made
  by `busy-emulator create` has this fixed: there, follow its `AGENTS.md`. Apps can read settings, not write them,
  and read them only at start.
- **Windows, with `@busy-app/cli` 0.6.0:**
  - `npm create @busy-app/app` may not install dependencies (`spawn npm ENOENT`): run `npm install` in the new
    folder.
  - `build` fails with `Rolldown failed to resolve import ... #font-maps`. In
    `node_modules/@busy-app/cli/dist/lib/fontMapsPlugin.js`, the generated `import` puts a Windows path with
    backslashes in a string; make it use forward slashes. Do it again after every `npm install`.
  - Build from PowerShell, not Git Bash: Git's `tar` reads `C:\...` as a remote host.
  - BUSY Bar Emulator's own commands avoid all three.
- **busy-lib:** the installed version is 0.21.1 (`device.DisplayDraw`). The README on GitHub describes 1.0.0-rc.1
  (`displayDraw`, `BusyBarAppClient`), which a new project does not have.
- **busy-tools has no `AGENTS.md`.** Where a document says to read it, read the README and the new project's
  `AGENTS.md` instead.
- **docs.busy.app pages that say there is no app catalog yet are out of date.**
- **Widgets:**
  - Widget with AI uses the fonts `medium` and `big`, which do not exist; take font names from the spec. Text is
    printable ASCII only, so the degree sign does not draw. Center text with `align` (`top_mid`, `center`,
    `bottom_mid`) instead of by hand.
  - Install the Python library as `busylib` in a virtual environment (`uv pip install busylib` or
    `pip install busylib`). In its `AGENTS.md`, the parts about contributing to the library do not apply, and in the
    released version `clear_before_draw` is `True` or `False` only.
  - When the widget stops, clear what it drew: `DELETE /api/display/draw?application_name=<name>`.
- **To see what BUSY Bar shows:** `GET /api/screen?display=0` returns the front screen as base64 text of raw
  72x16 pixels, 3 bytes each in BGR order, not the BMP file the spec describes. `display=1` is the back screen,
  160x80 at 4 bits per pixel.

## More

- Docs index for agents: https://docs.busy.app/llms.txt ; any docs.busy.app page also reads as Markdown with `.md`
  added to its address.
- Docs for people: [JS Apps Docs](https://go.busy.app/js-apps), [Emulator Docs](https://go.busy.app/busy-emulator-docs).
