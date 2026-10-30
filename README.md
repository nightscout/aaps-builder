# AAPS Builder

Build your own [AAPS](https://github.com/nightscout/AndroidAPS) app in GitHub, in your own private
repository. Nothing to install, no terminal, works from an Android phone.

**Start here: [https://nightscout.github.io/aaps-builder/](https://nightscout.github.io/aaps-builder/)**

Step-by-step guide with screenshots, also in other languages:
[Browser build with AAPS Builder](https://androidaps.readthedocs.io/en/latest/SettingUpAaps/BrowserBuildAapsBuilder.html)
in the AAPS documentation.

Do **not** fork or copy this repository. The setup page creates an empty private repository for
you and adds a short workflow file to it. That file runs the build steps kept here, pinned to one
reviewed commit.

## 🔄 Updating your copy

You created your repository from the old AAPS Builder template (a copy of this repository or of
`dio99/aaps-builder`, with a long `.github/workflows/build.yml` and this README)? Move to the
current version once. It takes 2 minutes, and your `KEYSTORE_SET` secret stays as it is:

1. Open the [setup page](https://nightscout.github.io/aaps-builder/) in a web browser (not the
   GitHub app). In step 1, type your GitHub username and the exact name of your repository.
2. In step 2, open **Updating the workflow, or the text area is empty?**
3. Tap **Copy workflow**, then **Open my workflow file**.
4. Replace the whole text with the copied workflow (on a phone: press and hold → **Select all** →
   **Paste**), then **Commit changes…** → **Commit changes**.
5. Build once to check (step 5 of the setup page). If you got an "AAPS Builder update available"
   issue, close it.

The same steps update the workflow later. Your repository never changes by itself: new versions
are announced in the AAPS documentation, on the page
[Docs updates & changes](https://androidaps.readthedocs.io/en/latest/Maintenance/DocumentationUpdate.html).

## Files

| File | What it is |
|---|---|
| `index.html` | Setup page (English), served by GitHub Pages at `nightscout.github.io/aaps-builder/` |
| `app.js` | Setup page logic (readable, unminified) |
| `style.css` | Setup page style |
| `forge.min.js` | node-forge 1.3.1, vendored (keystore creation in the browser) |
| `aaps-builder-build.yml` | The short build workflow (stub) the setup page adds as `.github/workflows/build.yml` of the user's repository |
| `.github/workflows/aaps-builder.yml` | The build steps: a reusable workflow (`on: workflow_call` only, never runs in this repository). Every user's stub calls it, pinned to one commit SHA |
| `VERSION` | Only for repositories made from the old template: their workflow compares it with its own version and opens an "update available" issue. Keep it above 2 |

User documentation: `docs/EN/SettingUpAaps/BrowserBuildAapsBuilder.md` in
[openaps/AndroidAPSdocs](https://github.com/openaps/AndroidAPSdocs) (label `aaps-builder`). It
describes every step of the setup page and what the workflow does, so it must change together with
these files.

## Trust boundary: approved sources only

The signing key ends up in the user's repository, next to the workflow, so everything that can run
there must come from an approved AAPS source:

- **Setup page**: every file comes from this repository, served from
  `https://nightscout.github.io/aaps-builder/`. The Content-Security-Policy (meta tag in
  `index.html`) allows scripts, styles and requests only from that folder, plus
  `https://api.github.com` to check that the typed username exists and
  `https://oauth2.googleapis.com/token` for the optional Google Drive step: no CDN, no other
  third-party request.
- **Google Drive step**: the same Google OAuth client, `drive.file` scope and PKCE flow as the AAPS
  preparation page (`nightscout/aaps-ci-preparation`), so `GDRIVE_OAUTH2` keeps its format
  (`base64(client_id|refresh_token)`). Google sends the browser to the loopback address
  `http://127.0.0.1:47913/aaps-builder`, where nothing listens: the tab shows an error and the user
  copies its address back into the page. The PKCE verifier and `state` stay in the page's tab, so
  the code is useless to anything else that sees it. `prompt=consent` makes Google send a refresh
  token also when the user granted access before. The only other request is reading `aaps-builder-build.yml` from the same folder (step
  "Add the build workflow"). The page puts its text into the link to GitHub's new-file page
  (`/new/main?filename=...&value=...`), so the user only confirms the commit.
- **Repository creation**: an **empty** private repository (`github.com/new` with `name`,
  `visibility=private`, `owner=@me`). **No template repository** is used, and this repository is
  not a template.
- **Workflow**: the stub in the user's repository only declares the **Run workflow** form and calls
  `nightscout/aaps-builder/.github/workflows/aaps-builder.yml@<commit SHA>`, passing the two secrets
  by name (`secrets: inherit` does not work across accounts). A full commit SHA cannot be moved:
  the user's repository runs exactly the reviewed build steps until the user replaces the stub. At
  run time the build steps only
  - clone the AAPS source from `github.com/nightscout/AndroidAPS` (release tags),
  - use GitHub's own actions `actions/setup-java` and `actions/upload-artifact`, each pinned to one
    commit SHA, on the pinned runner image `ubuntu-24.04`,
  - talk to `api.github.com` (read-only: its own repository's visibility check) and, only with the
    optional `GDRIVE_OAUTH2` secret, to Google (`oauth2.googleapis.com`, `www.googleapis.com`), the
    same endpoints as nightscout's `aaps-ci.yml`.
  - The Gradle build downloads AAPS's own dependencies, exactly as every other AAPS build method.
- **No update channel**: the workflow does not check anything outside the user's repository for
  updates, and the pinned SHA never follows a branch. Workflow changes are announced in the docs
  (Docs updates page and the AAPS Builder page); users replace the file themselves with the newer
  version from the setup page.
- **No automatic builds** (regulatory requirement): the only trigger is `workflow_dispatch`. Every
  build is started by the user, with the version and variant they choose. No `schedule`, no
  repository variables that change what is built, and the workflow token is read-only
  (`contents: read` everywhere): the build cannot write anything to the user's repository.

Review check before every change, in addition to the host list: the stub's `on:` block contains
only `workflow_dispatch`, the build steps' `on:` block only `workflow_call`, no `permissions:` line
says `write`, and the stub's `uses:` ends with a full 40-character commit SHA (never a branch or
tag).

```
grep -nE "uses:|https?://|curl|wget|git clone|ls-remote" aaps-builder-build.yml .github/workflows/aaps-builder.yml
grep -noE "https?://[^\"' )<]+" index.html app.js
```

Every host must be in the list above. `github.com/dio99/...` may appear only in the attribution
comments.

## Changing the workflow

A change always takes **two commits**, because the stub must name the commit that contains the
reviewed build steps:

1. Edit `.github/workflows/aaps-builder.yml` (raise the version in its first line), run the review
   check, commit and push it.
2. Copy that commit's full SHA (`git rev-parse HEAD`) into the `uses:` line of
   `aaps-builder-build.yml`, raise the version in its first line
   (`# AAPS Builder build workflow, version N`: the setup page shows it), commit and push. Never
   point it at a branch or a tag.
3. Update the docs page if steps, timings, artifacts, variants or error messages changed, and add
   an entry to `docs/EN/Maintenance/DocumentationUpdate.md` in openaps/AndroidAPSdocs telling users
   to replace their workflow file.
4. Test before announcing it: open the setup page, add the stub to a **private** test repository
   with **Add the workflow to my repository**, run a build, check the summary and the artifacts.

Only a commit that stays reachable on `main` may be pinned: never squash, rebase or force-push it
away. Users' stubs point at it for as long as they keep them.

## Testing the setup page

Serve the folder locally (`python -m http.server`) with the `Content-Security-Policy` meta tag
removed from a local copy of `index.html` (it allows only `nightscout.github.io`), or test on the
published page: no errors in the browser console, step 2 shows **Workflow version N is ready**,
**Add the workflow to my repository** opens
`github.com/<user>/my-aaps/new/main?filename=.github%2Fworkflows%2Fbuild.yml&value=...`,
**Copy workflow** shows the version, **Create a new key** with a test password unlocks step 4, and
**Create my private repository** points to
`github.com/new?owner=%40me&name=my-aaps&visibility=private` (no `template_` parameters).

## Origin and licence

Based on **AAPS Builder** by the AAPS Builder authors, https://github.com/dio99/aaps-builder,
**GNU AGPL-3.0** (see [LICENSE](LICENSE)). Imported into the AAPS documentation
(openaps/AndroidAPSdocs) after a full review, from upstream commit
`1115eea6c10a28b91486ccdee85a37e26ef75f41` (2026-10-08), maintained there up to build workflow
version 6, and moved here with version 7. Nothing is fetched from the upstream repository, neither
by the page, the workflow, nor any script in this repository.

Changes made since the upstream version (AGPL-3.0 section 5: modified version, notices kept in the
files):

1. No template repository: the user creates an empty private repository and adds the workflow from
   the setup page (step 2).
2. Workflow: removed the `builder-update` job (it read a version file from the upstream repository
   and asked users to copy a new workflow from there), removed the template check and the
   `BUILDER_REPO` / `BUILDER_VERSION` variables, English-only texts, error messages point to the
   setup page and the docs.
3. Setup page: English only, seven steps, texts in the HTML, forge served locally,
   Content-Security-Policy, a scrolling hint for GitHub's mobile build page (swipe on the text, not
   on the job diagram).
4. Workflow version 2: removed the weekly `schedule` trigger, the `AUTO_BUILD` and `AUTO_VARIANT`
   repository variables, the quiet skipping of scheduled runs and the `built/<version>-<variant>`
   tags that remembered scheduled builds. Permissions reduced to `contents: read`.
5. Workflow version 3: `aapsclient3Release` variant (AAPSClient3, AAPS 4.0 and later).
6. Workflow version 4: the build steps moved to a reusable workflow; users get a short stub that
   calls it at a pinned commit SHA. The setup page prefills the stub into GitHub's new-file page.
   The build steps also reject unknown variants.
7. Workflow version 6: GitHub's actions and the runner image pinned, clearer messages for a wrong
   `KEYSTORE_SET`.
8. Workflow version 7: build steps, stub and setup page moved from openaps/AndroidAPSdocs to this
   repository; error messages link to this setup page.
9. Setup page: step 7 gets `GDRIVE_OAUTH2` from Google itself (no preparation page, no File
   Manager Plus); step 1 warns about an existing repository, a fork, and the remembered name.
10. New keys created on the setup page use the alias `key0` (upstream: `aaps`), the same as the
   preparation page of the older browser build and Android Studio's default. Keys created before
   this change keep the alias `aaps`; the alias travels inside `KEYSTORE_SET`, so builds work with
   either.

## Checksums (of the committed files, LF line endings)

```
SHA-256(index.html)             = 312a8784c301e50d7a524ab028915d911f757c81ee3826bdcb6ff68b9c670687
SHA-256(app.js)                 = db12986c644ea237ed632ce0626d27b6b092b9004b3b008dfbc7a97170818e3d
SHA-256(style.css)              = b056b1d3fb4b92656c4f57c72d0844c30c77797febd2bc8041d95ced3c7b3aed
SHA-256(aaps-builder-build.yml) = 7f676bc4a748063b484c9898f3b54252102ef9d39d9860daaf49ebb475b924a1
SHA-256(forge.min.js)           = dc67fd132427ad96c9666c844b39565413c40ddb1f2d063c53512fbf6d387dfd
SHA-256(.github/workflows/aaps-builder.yml) = 1f0e3d4891ae2ae0db4f131decb9f7882b41f93f6d3c4cf0ef7cb1ec0f762789
```

`forge.min.js` is node-forge 1.3.1 (BSD-3-Clause or GPL-2.0, https://github.com/digitalbazaar/forge),
byte-identical to the npm package file `node-forge@1.3.1/dist/forge.min.js`.

On Windows with `core.autocrlf=true` the checked-out files can have CRLF line endings and a
different hash. Check the committed content instead: `git show HEAD:forge.min.js | sha256sum`.
**Update these lines in the same commit as any change to the files.**
