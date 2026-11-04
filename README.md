# 👋🧩 ffejhog Morphe Patches

Personal [Morphe](https://morphe.software) patches.

## ❓ About

Patches for apps I like.

### How to use these patches

Add this repo url to Morphe Manager: https://github.com/ffejhog/ffejhog_morphe_patches

<a href="https://morphe.software/add-source?github=ffejhog/ffejhog_morphe_patches">
  <img src=".github/assets/add-to-morphe.svg" alt="Add to Morphe" height="48">
</a>

## 📱 Bluesky

### Bypass age verification gate

Bluesky is a React Native app, so its UI is compiled to a Hermes bytecode bundle
(`assets/index.android.bundle`) rather than Dalvik bytecode. In regions that require age
assurance and for users who have not completed it, the app shell renders a full-screen
"share your age range" page instead of the app:

```
if      (session.status === 'deactivated')  -> DeactivatedScreen
else if (state.error === 'account-data')    -> DataUnavailableScreen
else if (state.access === Access.None)      -> NoAccessScreen     <- the age gate
else                                        -> RoutesContainer    <- the app
```

The patch swaps the conditional jump of that third branch for a non-branching comparison, so
the gate branch becomes unreachable and the shell always renders the app. The instruction is
the same length, so all other offsets stay valid, and the bundle's trailing SHA-1 footer is
recomputed so the Hermes runtime still accepts the file.

Age assurance state itself is untouched, so region-based content restrictions for unverified
users still apply. Only the blocking screen is removed.

The patch is a `RawResourcePatch` that matches a 12-byte window of the shell, anchored on the
two preceding instructions so the pattern is unique in the ~33 MB bundle. If a future app
version compiles the shell differently the patch fails with a clear error instead of producing a
broken APK; other versions are therefore marked experimental.

## 🩹 Patches list

<!-- PATCHES_START EXPANDED -->

<!-- Do not modify this section by hand. The patch list is generated when release.yml creates a new release.
     
     If you wish for the patches list to be collapsed, then remove the word 'EXPANDED' from the comment tag above.

     If you wish to manually keep this list updated then remove the PATCHES_START and PATCHES_END 
     comment blocks entirely. -->

#### A list of your patches will automatically be shown here after your first patches release is created.

&nbsp;

## 🚀 Getting development started

1. [Setup](https://github.com/MorpheApp/morphe-documentation/blob/main/docs/morphe-development/README.md) your development environment, including a GitHub PAT as described [here](https://github.com/MorpheApp/morphe-patcher/blob/main/docs/2_1_setup.md#-prepare-the-environment).
2. Enable "Allow GitHub Actions to create and approve pull requests" in Settings > Actions > General > Workflow permissions.
3. Add the `MORPHE_PACKAGES_TOKEN` secret (a classic PAT with `read:packages`). The Morphe Gradle
   settings plugin requires GitHub Packages credentials, and `app.morphe:morphe-patcher` is not
   readable anonymously. `MORPHE_SIGNING_KEY` is optional, see below.

### Optional: sign releases

Signing lets Morphe Manager reject a release that was modified after publication.

1. Run `./gradlew generateSigningKey`. It creates `morphe-signing.pub` and `morphe-signing.key`.
2. Commit `morphe-signing.pub`. Never commit `morphe-signing.key`, it is already ignored by git.
3. Add the contents of `morphe-signing.key` as the `MORPHE_SIGNING_KEY` secret, keep a backup
   somewhere safe, then delete the file.

Every release is signed from then on. If the key is lost or leaks, generate a new one the same
way; users then have to confirm the new key in Morphe Manager before receiving your next release.

## 🧑‍💻 Dev usage

To develop and release these patches:

- **Make all changes to the `dev` branch.**
- For local development work build your patches using the gradle task `./gradlew buildAndroid` to generate the mpp file found in `patches/build/libs/patches-*.mpp`. Apply your patches locally using Morphe Desktop tool like any other patch bundle.
- Always use [Semantic commit](https://kapeli.com/cheat_sheets/Semantic_Commits.docset/Contents/Resources/Documents/index) messages for commits. To keep it simple use only 3 commit message types: 
  - `feat: Added a new feature`
  - `fix: Some problem now fixed`
  - `chore: Random change you do not want in the user facing changelog`
- Commits of `fix:` and `feat:` will automatically generate new pre-releases and `chore:` will not create a new release.
- Users can apply your dev branch releases by enabling `pre-release` in Morphe Manager patch sources.
- When your dev branch is ready, and you want a stable release, merge dev branch to main (do not squash, and only merge).
- **Always use semantic release (release.yml)**. Do not manually upload or create releases by hand
  because many files must be updated and release.yml handles everything.

## 🤓 Tips
- See the [patcher documentation](https://github.com/MorpheApp/morphe-patcher/blob/main/docs/1_patcher_intro.md) for more examples of creating patches and fingerprints.
- Do not use AI to create new release scripts. The `release.yml` here already handles everything.
  If you need something custom with your releases then modify the existing `release.yml`
  and `.releaserc` instead of writing everything new from scratch.
- Do not manually edit or manually commit any generated files such as: `patches-list.json`,
  `patches-bundle.json`, `CHANGELOG.md`.  These files will be automatically updated by `release.yml`.
- Do not force push any semantic release commits as that will break all future releases.
  If you need to fix a broken release, it's always easiest to create a new release instead of 
  fixing an existing release.


<!-- The patches end tag is intentionally placed here so the first release will clean up 
     this readme of all developer instructions above. -->
<!-- PATCHES_END -->

### 🛠️ Building locally

- Run `./gradlew buildAndroid`
- The built patches .mpp file is found in `patches/build/libs/patches-*.mpp`
- Patch the mpp file using [Morphe-Desktop](https://github.com/MorpheApp/morphe-desktop)
  like any other patch bundle.

See the [Morphe documentation](https://github.com/MorpheApp/morphe-documentation) for more information.

## 📜 License

ffejhog Morphe Patches are licensed under the [GNU General Public License v3.0](LICENSE)
