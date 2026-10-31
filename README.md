# AAPS Builder

Build your own [AAPS](https://github.com/nightscout/AndroidAPS) app in GitHub, in your own private
repository. Nothing to install, no terminal, works from an Android phone.

> **No fork needed.** You do not fork AndroidAPS. The source code is downloaded from
> [nightscout/AndroidAPS](https://github.com/nightscout/AndroidAPS) at every build and signed with your own key.

## 👉 New here?

Start with the setup page in the AAPS documentation. It walks you through everything, and the
documentation explains each step with screenshots, in your language:

**[AAPS Builder setup page](https://androidaps.readthedocs.io/en/latest/SettingUpAaps/BrowserBuildAapsBuilderSetup.html)** ·
[Browser build with AAPS Builder](https://androidaps.readthedocs.io/en/latest/SettingUpAaps/BrowserBuildAapsBuilder.html)

Step 1 of the setup page creates your own **private copy** of this repository (GitHub's "Use this
template"). Do not fork it: a fork is public, and everything a public repository builds can be
downloaded by anyone.

## 🔧 Already have your own copy?

These buttons always open **this** repository, whatever you named it:

| | |
|---|---|
| 🔑 **[Add your key](../../settings/secrets/actions/new)** | Name: `KEYSTORE_SET` · Secret: the long code from the setup page (step 3) |
| ▶️ **[Build AAPS](../../actions/workflows/build.yml)** | Run workflow → Run workflow |
| 📦 **[My builds](../../actions)** | Open a finished build → tap **📱 Download AAPS** |

> **On a phone, use the web browser (Chrome), not the GitHub app.** The app cannot show or download your builds.
> In the app, open the build and tap **Artifacts ↗**, or tap the **share icon** (top right) → **Copy link**, then open
> Chrome and paste the link in the address bar. To stop GitHub links opening in the app: Android Settings → Apps →
> GitHub → Open by default → turn off **Open supported links**.

**New AAPS version?** Press **Build AAPS** → **Run workflow** again. `latest` always builds the newest release. Your
repository never builds anything by itself.

**📧 E-mail when a build is done (optional):** by default GitHub only e-mails you when a build fails. To also get an
e-mail when it succeeds: open [github.com/settings/notifications](https://github.com/settings/notifications) →
**System → Actions** → untick **"Only notify for failed workflows"**.

## 🔄 Updates

The build steps are updated for you: at every build, your repository runs the released version of
`.github/workflows/aaps-builder.yml` from this repository (branch `release`). Nothing in your repository changes.

You only replace your own `.github/workflows/build.yml` when the AAPS documentation announces it on the page
[Docs updates & changes](https://androidaps.readthedocs.io/en/latest/Maintenance/DocumentationUpdate.html), for
example for a new build variant, or once if your repository was made with the earlier setup page or from the old
template. It takes 2 minutes, and your `KEYSTORE_SET` secret stays as it is:

1. Open the newest file: **https://github.com/nightscout/aaps-builder/blob/main/.github/workflows/build.yml**
2. Click the **copy icon** ⧉ ("Copy raw file") at the top right of the file.
   *On a phone:* open the plain-text version instead:
   **https://raw.githubusercontent.com/nightscout/aaps-builder/main/.github/workflows/build.yml**,
   press and hold the text → **Select all** → **Copy**.
3. Open **your own** repository → **Code** → folder `.github` → `workflows` → `build.yml`.
4. Click the **pencil** ✏️ (Edit this file) at the top right.
5. Click in the text and select everything (**Ctrl+A**; on a phone: press and hold → **Select all**), then paste
   (**Ctrl+V** / **Paste**). The old text must be completely replaced.
6. Click the green **Commit changes…** button, then **Commit changes** again.

The first line of the file shows its version. To check, run **Build AAPS** once.

## ⚠️ Before every update

1. AAPS → Maintenance → **Export settings**
2. Copy the exported file **off the phone** (Google Drive, Dropbox…), together with the APK files. Keep several older
   exports.

[How to export settings](https://androidaps.readthedocs.io/en/latest/Maintenance/ExportImportSettings.html) ·
[AAPS FAQ: how to organize backups](https://androidaps.readthedocs.io/en/latest/UsefulLinks/FAQ.html#how-to-organize-my-backups)

## Why a private repository?

Everything a build produces is visible to anyone who can see the repository. A **private** repository keeps
your APK for your eyes only. The workflow refuses to run in a public repository unless the APKs go to
Google Drive instead. Private repositories get 2000 free GitHub Actions minutes per month; one build takes
about 15–30 minutes.

## Secrets

| Secret | Required | What |
|---|---|---|
| `KEYSTORE_SET` | yes | Keystore and passwords in one value, created by the setup page |
| `GDRIVE_OAUTH2` | no | Also upload the APKs to Google Drive (folder `AAPS/<version>`), created in step 6 of the setup page |

## Files

| File | What it is |
|---|---|
| `.github/workflows/build.yml` | Your build workflow: the **Run workflow** form, and one line that runs the build steps below from the `release` branch of nightscout/aaps-builder. The first line shows its version |
| `.github/workflows/aaps-builder.yml` | The build steps (a reusable workflow, no **Run workflow** button). Your build uses the released version from nightscout/aaps-builder, not the copy in your repository |
| `VERSION` | Only for repositories made from the old template (before version 3): their workflow compares it with its own version and opens an "update available" issue |
| `README.md`, `LICENSE` | This page and the licence |

## For maintainers

- This repository is a **template** (Settings → **Template repository**). Users create their copy from it with the
  setup page; GitHub Pages is not used.
- The setup page and the user guide live in the AAPS documentation,
  [openaps/AndroidAPSdocs](https://github.com/openaps/AndroidAPSdocs): `docs/EN/SettingUpAaps/BrowserBuildAapsBuilderSetup.md`
  (label `aaps-builder-setup`) and `docs/EN/SettingUpAaps/BrowserBuildAapsBuilder.md` (label `aaps-builder`). They
  describe the steps of the setup page and what the workflow does, so they change together with `build.yml`.
- Users' repositories call `.github/workflows/aaps-builder.yml@release`. To change the build steps: edit the file on
  `main`, raise the version in its first line, test a build in a private copy whose `build.yml` points at your
  branch, then move the `release` branch to the tested commit (`git push origin main:release`). Every user gets it
  at their next build; announce it in `docs/EN/Maintenance/DocumentationUpdate.md`.
- `.github/workflows/build.yml` (the stub copied into every user's repository) changes only when the
  **Run workflow** form changes, for example a new variant. Raise the version in its first line and tell users to
  replace their file, in the same docs entry.
- Builds are started by the user only (`workflow_dispatch`). No `schedule`, no variables that change what is built:
  automatic builds are not acceptable for a medical app.

## Origin and licence

Based on **AAPS Builder** by the AAPS Builder authors, https://github.com/dio99/aaps-builder,
**GNU Affero General Public License v3.0 (AGPL-3.0)**, see [LICENSE](LICENSE). Imported into the AAPS
documentation after a review, maintained there up to build workflow version 7, and moved here.
Changes since the upstream version (AGPL-3.0 section 5, notices kept in the files): no automatic or scheduled builds
and read-only workflow permissions; the build steps are a reusable workflow run from the `release` branch of this
repository, so users' copies need no update check; the `aapsclient3Release` variant (AAPSClient3, AAPS 4.0 and later) and rejection of unknown
variants; GitHub's actions and runner image pinned to exact versions; clearer messages for a wrong `KEYSTORE_SET`;
setup page moved to the AAPS documentation, in English and translated, with the Google Drive step on the page itself
and new keys using the alias `key0`.

### Key terms

* **Copyleft:** any modifications or derivative works must also be open-sourced under the AGPL-3.0.
* **Network trigger:** if you run a modified version of this software on a server and let users interact with it over
  a network, you must make your modified source code available to those users.
