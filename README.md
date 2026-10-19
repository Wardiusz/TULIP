<p align="center">
  <img src="public/tulip-icon.svg" alt="TULIP App Icon" width="128" height="128">
</p>

<h1 align="center">TULIP</h1>

<p align="center">
  TULIP (<a href="https://p.lodz.pl/en/about-tul">TUL</a> Intelligent Planner) is a readable university timetable viewer, added by scanning a screenshot, importing JSON or entering classes by hand and browsing it in four views: <b>session</b>, <b>day</b>, <b>week</b> and <b>month</b>.

  Why? You may asked. Mostly because i want clearer and better way to read TUL timetables, including <b>extramural (weekend) studies</b>, where classes happen on a few weekends per semester ("zjazd" = session), and <b>regular weekday timetables</b> with numbered terms.</p>

---

<p align="center">
  <a href="https://github.com/Wardiusz/DCQuestCompleter/blob/master/LICENSE">
    <img src="https://img.shields.io/github/license/wardiusz/dcquestcompleter?style=flat-square" alt="License">
  </a>
</p>

<!-- <p align="center">
  <img src="public/screenshot.png" alt="DCQuestCompleter app screenshot">
</p> -->

## Table of contents

- [Features](#Features)
- [Quick start](#Quick-start)
- [Scanning timetable images](#Scanning-timetable-images)
- [How the calendar works](#How-the-calendar-works)
- [Project structure](#Project-structure)
- [Languages](#Languages)
- [Limitations](#Limitations)

## Features

- **Four views.** Session (a weekend with Saturday and Sunday side by side, extramural plans only), day, week
  (Monday to Sunday) and month. Month cells open the day view.
- **Live time tracking.** A red "now" line moves across today's column and the class in progress is highlighted.
- **Image scanning.** Turn a screenshot or photo of a timetable into class blocks with an AI model
  (see [Scanning](#scanning-timetable-images)).
- **Manual editing.** Click any block to edit or delete it, or add one from the **Timetable** dialog.
- **JSON export and import.** Move a timetable between devices or keep a backup. Export includes the session dates.
- **Light, dark and auto theme**, and a **Polish / English** interface (PL / EN switch in the header).
- **No backend.** Everything runs in the browser and data stays there.

## Quick start

Requires Node.js 18 or newer (CI uses 22).

```bash
npm install
npm run dev        # dev server at http://localhost:5173
npm run build      # production build in dist/
npm run preview    # serve the production build locally
```

`dist/` is plain static files. `base: './'` in `vite.config.js` lets it be hosted from a sub-path such as
`https://<user>.github.io/<repo>/`.

### Deploying to GitHub Pages

1. Push the project to a GitHub repository (the repository contents should be the project root, so `.github/`
   sits at the top level).
2. In **Settings → Pages**, set **Source** to **GitHub Actions**.
3. Push to `main`. The *Deploy (GitHub Pages)* workflow builds the app and publishes it.
   On a free account the repository must be public.

### GitHub Actions

| File                           | Trigger        | What it does                                 |
| ------------------------------ | -------------- | -------------------------------------------- |
| `.github/workflows/ci.yml`     | pull requests  | `npm ci` and `npm run build`                 |
| `.github/workflows/deploy.yml` | push to `main` | builds and deploys to GitHub Pages           |
| `.github/dependabot.yml`       | weekly         | proposes updates to Actions and npm packages |

## Scanning timetable images

Scanning is optional. Without it you can still add classes manually or import JSON.

| Where you run the app                    | How scanning works                                        |
| ---------------------------------------- | --------------------------------------------------------- |
|  **Direct** mode (default)               | The browser calls Gemini or Claude with your own API key. |
|  **Worker** mode                         | The browser calls your own proxy, which holds the key.    |

### Direct mode

Open **Timetable → Image scanning**, choose the provider, paste your API key and press **Save**.

- Gemini keys come from Google AI Studio, Claude keys from console.anthropic.com.
- The key is stored in this browser's `localStorage` only. It is not sent to the repository or other devices,
  so enter it on each device you use.
- A static site has nowhere to hide a secret. Use a key with a low quota. For Gemini you can restrict the key to your
  site's address in Google Cloud. Never hard-code a key in the source.
- Default models are `gemini-3.5-flash` and `claude-sonnet-5-5`. Set the **Model** field if a name changes.
- Temporary provider errors (HTTP 500, 502, 503, 504) are retried up to 3 times. You can also set a **fallback model**
  that is tried when the main one stays overloaded.

### Worker mode (optional)

`worker/scan-worker.js` is a Cloudflare Worker that keeps API keys server-side and supports both providers.
Deploy it from the dashboard or with `worker/wrangler.toml`, then enter its URL and access code in
**Timetable → Image scanning → Via your own service**.

| Setting                        | Type      | Purpose                                                     |
| ------------------------------ | --------- | ----------------------------------------------------------- |
| `ANTHROPIC_API_KEY`            | secret    | Claude key (optional if you use Gemini)                     |
| `GEMINI_API_KEY`               | secret    | Gemini key (optional if you use Claude)                     |
| `ACCESS_CODE`                  | secret    | shared code the page sends; strongly recommended            |
| `ALLOWED_ORIGIN`               | variable  | your site's origin, e.g. `https://user.github.io` (no path) |
| `MODEL_CLAUDE`, `MODEL_GEMINI` | variables | optional model overrides                                    |

Variables defined in `wrangler.toml` override those set in the dashboard on every deploy. The `worker/` folder is not
needed for the site itself and can be deleted.

> AI scanning can misread a dense table. Always compare the result with the original timetable.

## How the calendar works

The app supports two kinds of timetables, detected from the classes:

- **Extramural (session) mode**, when every block is on Saturday or Sunday. "Weeks: 1-5" means sessions 1 to 5.
  Session dates are editable in **Session dates**.
- **Weekday mode**, when any block is Monday to Friday. "Weeks" are *terms* (t1 to t15), not calendar weeks:
  Thursday and Friday can have different numbers than Monday, and some days count as another weekday
  (for example a Monday that follows Wednesday's schedule).

The calendar for **winter semester 2026/27** is built in: session dates in `ZD` and the term table in `TR`,
both in `src/lib.js`. For another semester edit those two constants. Weekday plans created after February 2027 fall
back to plain weeks starting on a Monday, editable in **Semester weeks**.

## Data format

**Timetable → Export** produces JSON that **Import** accepts:

```json
{
  "title": "Winter 2026/2027",
  "group": "1I",
  "zj": ["2026-10-03", "2026-10-10"],
  "events": [
    {
      "day": 6,
      "start": "08:15",
      "end": "10:00",
      "type": "Lecture",
      "weeks": "1-5",
      "title": "Advanced Processing Systems",
      "room": "A12 E5",
      "teacher": "Dr Smith",
      "group": "1I",
      "note": ""
    }
  ]
}
```

`day` is 1 (Monday) to 7 (Sunday). `weeks` accepts ranges and lists such as `1-4, 6`; an empty value means every week.
`zj` (session or week start dates) is optional. A bare array of events is also accepted.

## Storage and privacy

Everything is kept in the browser's `localStorage` (keys start with `plan-zjazdy-v1`). The app has no server of its own (yet).
Apart from loading the Google Fonts stylesheet, it sends data only when you scan an image, and then only to the
provider you chose.

## Project structure

```text
src/
  App.jsx        app state and layout
  lib.js         dates, calendar mapping, persistence, scanning
  views.jsx      time grid and month view
  dialogs.jsx    timetable, block editor and session-dates dialogs
  i18n.js        PL / EN strings
  styles.css     styles (light and dark via CSS variables)
worker/          optional Cloudflare Worker for scanning
.github/         CI, deploy and Dependabot configuration
```

Stack: React 18, Vite 5, plain CSS. There are no runtime dependencies besides React.

## Languages

The interface is available in Polish and English, switched in the header and remembered per browser. Strings live in
`src/i18n.js`: edit the `pl` and `en` dictionaries to fix a translation or add a language.
Subject names, class types and rooms come from the timetable and are shown as written.

## Limitations

- The built-in term calendar covers winter 2026/27 only (for now).
- In weekday mode with the built-in calendar, Saturday and Sunday blocks are not shown.
- Scan quality depends on the model and on the image. Check the result.
- Data is per browser. Use JSON export and import to move it between devices (temporary solution).

## License

This project is licensed under the MIT License.
