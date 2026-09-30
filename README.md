# SCSI Landing Page

Landing page for **Services Conseils Sensaroli Inc.** (SCSI), an IT consulting firm based in
Châteauguay, Québec. It is the company's web hub: a static single-page site that presents the
company and links out to its other, independently deployed projects.

French is the primary language; English is available as a runtime toggle. There is no backend.

- **Live:** <https://sensaroli-consulting.net>
- **Architecture and conventions:** see [ARCHITECTURE.md](ARCHITECTURE.md)

## Stack

| Layer            | Technology                                      |
|------------------|-------------------------------------------------|
| Framework        | Angular 22 — standalone components, signals     |
| Styling          | Tailwind CSS v4 + DaisyUI v5                    |
| i18n             | Transloco (`fr` default, `en` secondary)        |
| Tests            | Vitest, via `@angular/build:unit-test`          |
| Backend          | None                                            |
| Hosting          | Azure Static Web Apps (Free tier)               |
| Infrastructure   | OpenTofu (`terraform/`)                         |
| Dev environment  | Nix flake + direnv                              |

No SSR: the app is a pure client-side SPA.

## Getting started

### With Nix (recommended)

The flake provides Node 26, the Azure CLI, and OpenTofu, and installs the Angular CLI into
`~/.npm-global` on first entry (the Nix store is read-only, so `ng` cannot be installed into it).

```bash
direnv allow     # or: nix develop
npm ci
npm start
```

### Without Nix

Install Node 26 and npm yourself, then:

```bash
npm ci
npm start
```

The dev server runs at <http://localhost:4200/> and reloads on source changes.

## Commands

| Command           | Purpose                                                        |
|-------------------|----------------------------------------------------------------|
| `npm start`       | Dev server on port 4200                                        |
| `npm run build`   | Production build into `dist/scsi-landing-page/browser`         |
| `npm test`        | Unit tests (Vitest)                                            |

There is no end-to-end suite.

## Internationalization

Translations are plain JSON in `public/i18n/` (`fr.json`, `en.json`) and are fetched at runtime
by the `TranslocoHttpLoader` in `src/app/app.config.ts`. One build serves both languages — there
is no per-locale build and no locale in the URL.

To add a language: drop `public/i18n/<lang>.json` next to the others and add `<lang>` to
`availableLangs` in `src/app/app.config.ts`.

## Styling

Tailwind v4 is configured **in CSS, not JavaScript**. There is no `tailwind.config.js`; the whole
configuration is `src/styles.css`:

```css
@import 'tailwindcss';
@plugin 'daisyui';
```

Design tokens and theme overrides belong in that file. PostCSS wiring lives in `.postcssrc.json`.

## Deployment

Pushing to `main` triggers [`.github/workflows/deploy.yaml`](.github/workflows/deploy.yaml), which
builds the app and uploads it to Azure Static Web Apps with `Azure/static-web-apps-deploy@v1`.

Two things about that workflow are easy to break:

- **`skip_app_build: true` means `app_location` is the content root.** The action reads the
  directory verbatim and does *not* append `output_location`, so `app_location` points at the
  already-built `dist/scsi-landing-page/browser`. If you change the build output path, change it
  here too.
- **Oryx, Azure's build system, ignores the runner's Node version.** It runs in its own container,
  which is why the build happens in the workflow and is then skipped on Azure's side.

The workflow needs one repository secret, `AZURE_STATIC_WEB_APPS_API_TOKEN`, holding the Static Web
App's deployment token (see `deployment_token` in `terraform/outputs.tf`).

## Infrastructure

`terraform/` holds the OpenTofu configuration for the resource group and the Static Web App.
State is local and **not** committed, and neither is `terraform.tfvars`.

Create `terraform/terraform.tfvars` with the two required variables — everything else has a
default in `variables.tf`:

```hcl
subscription_id = "<azure-subscription-guid>"
tenant_id       = "<azure-tenant-guid>"
```

Then:

```bash
cd terraform
tofu init
tofu plan
tofu apply
```

The Static Web App is on the Free tier, which allows two custom domains.

## License

MIT — see [LICENSE](LICENSE).
