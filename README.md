# ratio-app-templates

Scaffolding templates for `ratio app create`. The CLI downloads one pinned tag of this repo and renders `typescript/<template>/` locally.

## Layout

```
typescript/<template>/
  manifest.json          template name, description, substitutionKeys
  *.tmpl                 rendered with {{key}} substitution, `.tmpl` stripped
  gitignore              rendered as .gitignore
  ...                    everything else is copied byte for byte
```

Templates: `minimal`, `with-admin-ui`, `serverless`.

## Manifest contract

`manifest.json` has `name`, `description` and `substitutionKeys`. A `.tmpl` file may only use `{{key}}` placeholders listed in `substitutionKeys`; an unknown or unfilled key fails the render. The CLI supplies `appName`, `clientId`, `sdkVersion` and `templateName`.

## Naming rule for dotfiles (gitignore)

npm and some archive tools drop or special-case dotfiles, and a root `.gitignore` can hide files from this repo.

- A template's `.gitignore` is stored as `gitignore` at the template root; the CLI renames it on render.
- Env examples are stored as `.env.example.tmpl` (and `.dev.vars.example.tmpl`). The root `.gitignore` of this repo must not ignore `.env.*.tmpl`, or those files silently vanish from the tag.

## Releases

- Tags are never moved or deleted. A fix is a new tag.
- A tag must compile against the `@ratio-app/sdk` version the referencing CLI pins.
- Procedure: tag the commit, then bump `TEMPLATES_REF` and `TEMPLATES_TREE_SHA256` together in the CLI (`src/lib/scaffold/fetch-templates.ts`). The digest covers `typescript/**` at the tag.
- Protect `v*` tags in the repo settings so they cannot be updated or deleted.

## Rules for template content

- No `preinstall`, `install`, `postinstall` or `prepare` script in any template `package.json`. Generated apps run `npm install` on the developer's machine.
- No `export-ignore` attribute under `typescript/`; `git archive` would omit those files from the tarball the CLI downloads.
- Webhook handlers verify `X-GoKwik-Signature` with `verify` from `@ratio-app/sdk/webhooks` and dispatch on the signed `eventType` body field, never on a header. The only delivered topic handled is `app/uninstalled`.
