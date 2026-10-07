# Orthodox Web

Web application for the Orthodox Church, built with Angular 17. It serves the
public website (home, sections, events, memorials, scheduler, gallery, store,
etc.) with internationalization (Spanish/English) and consumes a separate
ASP.NET API.

- **Production URL:** https://www.ortodoxa.net
- **API URL:** https://api.ortodoxa.net
- **Deployment target:** SmarterASP.NET (FTP), as a sub-site at `/ortodox`

---

## Overview

This repository contains only the **frontend** (Angular). The backend is a
separate ASP.NET project that is deployed independently under the
`/ortodox/api` path (see [Deployment](#deployment-smarteraspnet)).

The app is a server-side rendered (SSR) Angular application. It is built into
a static browser bundle (`dist/orthodox-web/browser`) that is served by IIS on
SmarterASP.NET.

---

## Tech Stack

| Area            | Technology                                                       |
| --------------- | ---------------------------------------------------------------- |
| Framework       | Angular 17 (NgModule based, non-standalone components)           |
| Rendering       | Angular SSR / prerender (`@angular/ssr`, `@angular/platform-server`) |
| Language        | TypeScript 5.2                                                   |
| Styling         | Tailwind CSS 3.4 + global CSS                                    |
| UI Components   | PrimeNG 17, Syncfusion EJ2 (calendar, schedule, inputs)          |
| Icons           | Font Awesome (`@fortawesome/angular-fontawesome`), `@ng-icons`, PrimeIcons |
| i18n            | `@ngx-translate/core` (assets: `en.json`, `es.json`)             |
| Rich text       | `ngx-quill` / Quill                                              |
| Media           | `aplayer` (audio player), `ng-gallery` (galleries)               |
| API access      | `@angular/common/http` (`HttpClient`)                           |
| Server runtime  | Node.js + Express (`server.ts`) for local SSR                    |

---

## Requirements

- **Node.js:** `^18.13.0 || >=20.9.0` (required by Angular 17).
  LTS `18.13+` or `20.9+` is recommended. The CI pipeline uses Node 18.
- **npm:** 9 or later (bundled with the Node versions above).
- **Angular CLI:** 17 (installed locally via `devDependencies`; a global
  install is optional).
- A running instance of the backend API (or network access to
  `https://api.ortodoxa.net`) for data-driven pages.

---

## Local Setup

1. **Clone the repository**

   ```bash
   git clone <repository-url>
   cd orthodox-web
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **(Optional) Configure the API base URL**

   The API endpoint is defined in `src/environments/environment.ts`
   (development) and `src/environments/environment.prod.ts` (production):

   ```ts
   export const environment = {
     production: false,
     baseUrl: 'https://api.ortodoxa.net',
   };
   ```

4. **Start the development server**

   ```bash
   npm start
   ```

   Navigate to `http://localhost:4200/`. The app reloads automatically on
   source changes.

5. **(Optional) Run the SSR server after a build**

   ```bash
   npm run build
   npm run serve:ssr:orthodox-web
   ```

   The Express SSR server listens on `http://localhost:4000` by default
   (override with the `PORT` environment variable).

---

## npm Scripts

| Command                        | Description                                                    |
| ------------------------------ | -------------------------------------------------------------- |
| `npm start`                    | Start dev server (`ng serve`) at `http://localhost:4200/`.     |
| `npm run build`                | Production build to `dist/orthodox-web` (default config).      |
| `npm run watch`                | Development build in watch mode.                              |
| `npm test`                     | Run unit tests with Karma.                                     |
| `npm run serve:ssr:orthodox-web` | Run the built SSR server (`dist/orthodox-web/server/server.mjs`). |
| `npm run ng -- <args>`         | Run any Angular CLI command (e.g. `npm run ng -- generate component foo`). |

### Useful Angular CLI commands

```bash
ng serve
ng build
ng generate component <name>
ng test
```

---

## Build Output

`ng build` produces:

```
dist/orthodox-web/
├── browser/            # Static frontend to deploy (HTML, JS, CSS, assets)
│   └── web.config      # IIS rewrite rules (copied manually, see below)
├── server/             # SSR bundle (used for local SSR only)
└── prerendered-routes.json
```

Only the contents of **`dist/orthodox-web/browser`** (plus `web.config`) are
deployed to SmarterASP.NET.

The `web.config` at the repository root enables IIS URL rewriting so that
Angular client-side routes are served from `/`:

```xml
<rule name="Angular Routes" stopProcessing="true">
  <match url=".*" />
  <conditions logicalGrouping="MatchAll">
    <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
    <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
  </conditions>
  <action type="Rewrite" url="/" />
</rule>
```

---

## Project Structure

```
src/
├── app/
│   ├── app-routing.module.ts     # Route definitions
│   ├── events/                   # Lazily loaded events module
│   ├── memorials/                # Lazily loaded memorials module
│   ├── home/ header/ footer/     # Layout & common blocks
│   ├── sections/                 # Content sections / pages
│   ├── services/                 # API services (evento, cartelera, ...)
│   ├── models/                   # TypeScript interfaces
│   └── mappers/                  # Data mappers
├── assets/
│   ├── i18n/                     # en.json, es.json
│   ├── icons/  images/           # Static assets
├── environments/                 # environment.ts / environment.prod.ts
├── index.html
├── main.ts  main.server.ts
└── styles.css
server.ts                         # Express SSR entry point
web.config                        # IIS rewrite rules
angular.json                      # Angular workspace config
```

---

## Deployment (SmarterASP.NET)

The site is hosted on **SmarterASP.NET** using the **basic plan**, which allows
one main website and up to two sub-sites. The sub-site directory must live
**inside the main website folder**. This repository is deployed as a sub-site
at the `/ortodox` path, while the API lives in its own sub-site at
`/ortodox/api`.

### Target folder structure on the server

```
<main site root>/
└── ortodox/                 # Sub-site: THIS Angular app
    ├── index.html           # Contents of dist/orthodox-web/browser
    ├── main-*.js
    ├── styles-*.css
    ├── assets/
    ├── web.config           # Copied from the repository root
    └── api/                 # Sub-site: separate ASP.NET API (do NOT touch)
```

> **Important:** `/ortodox/api` is deployed from a **different, independent
> ASP.NET project**. When deploying the Angular build you must **not delete or
> overwrite the `api` folder**. The deployment must exclude `api` from any
> cleanup step.

### Build

```bash
npm install
npm run build
```

Then copy the `web.config` from the repository root into the browser output:

```bash
# Linux/macOS
cp web.config dist/orthodox-web/browser/web.config

# Windows (PowerShell)
Copy-Item web.config dist\orthodox-web\browser\web.config
```

### Deploy via FTP

FTP settings (see `.github/workflows/release.yml`):

| Setting  | Value                                  |
| -------- | -------------------------------------- |
| Server   | `win9081.site4now.net`                 |
| Username | `admin-huss`                           |
| Password | stored as the `FTP_PASSWORD` secret    |
| Directory| `/ortodox`                             |

Manual deployment steps:

1. Connect to the FTP server with your FTP client.
2. Navigate to `/ortodox`.
3. **Delete everything under `/ortodox` except the `api` directory.** This
   removes stale hashed bundles while preserving the API sub-site.
4. Upload the **contents** of `dist/orthodox-web/browser/` (including the
   copied `web.config`) into `/ortodox`.
5. Verify that `/ortodox/api` still contains the ASP.NET API files.

### Automated deployment (GitHub Actions)

The workflow `.github/workflows/release.yml` automates the process on pushes
to the `tmp` branch:

1. Builds the app with `ng build --configuration=production`.
2. Copies `web.config` into `dist/orthodox-web/browser/`.
3. Connects via `lftp`, lists `/ortodox`, and deletes every entry **except
   `api`**.
4. Uploads `dist/orthodox-web/browser/` to `/ortodox/` with
   `SamKirkland/FTP-Deploy-Action`.

The `FTP_PASSWORD` is read from GitHub repository secrets and must be
configured before the workflow can run.
