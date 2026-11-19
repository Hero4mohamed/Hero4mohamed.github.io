# Hero4mohamed.github.io

The **root page** of `https://hero4mohamed.github.io/`.

## Why this repository exists

The Android app of TurkMaps (a Trusted Web Activity) hides the browser bar only if the **root of the web address** vouches for it, by serving a file at

```
https://hero4mohamed.github.io/.well-known/assetlinks.json
```

The transit map lives in a *project site* (`/Metro-Istanbul-General-City-Map/`), and a project site cannot serve files at the root. Only a repository named **exactly** `Hero4mohamed.github.io` can. Without this file the Android app still runs, but it opens as "a website in a tab" with a browser bar.

**Nothing else changes.** The project sites on this address (the transit map, the Turkish-learning app, and any other) keep being served from their own repositories and are not affected by this one.

## What is in it

| File | What it is |
|---|---|
| `.well-known/assetlinks.json` | The address file. **`[]` (valid, empty) until the app's first upload to Google Play**, then it names the app's package and certificate fingerprints. |
| `.nojekyll` | An empty file. Without it GitHub's Jekyll step **silently skips folders that start with a dot**, and `.well-known/` would return 404. |
| `index.html` | A small start page (English + Turkish) pointing to TurkMaps and its privacy notice. Today the bare address is a 404. |
| `404.html` | A plain "page not found" for this root site only. |

Everything is static, has no script, loads nothing from other sites, and contains no personal data.

## Publish it (you, once)

From inside this folder (it is already a git repository with one commit):

```bash
gh repo create Hero4mohamed/Hero4mohamed.github.io --public --source . --remote origin --push
```

Then check **Settings > Pages**: a repository with this exact name is normally published from the default branch automatically; if it says nothing is published, choose **Deploy from a branch**, branch **main**, folder **/ (root)**. The first publish takes a minute or two.

Check it from any computer:

```bash
curl -i https://hero4mohamed.github.io/.well-known/assetlinks.json
```

You want **`HTTP/2 200`** and a body of `[]`. A `404` means the Pages step has not finished, or `.nojekyll` is missing. Also open `https://hero4mohamed.github.io/` in a browser (the start page) and confirm `https://hero4mohamed.github.io/Metro-Istanbul-General-City-Map/` still opens the map.

## After the first upload to Google Play (the address file gets its content)

1. In Play Console open the app, then **App integrity > App signing**, and copy the **SHA-256 certificate fingerprint** of the **app signing key**. (With Play App Signing this is Google's key, not yours.) For builds you install yourself, also take the SHA-256 of your own upload key (`keytool -list -v -keystore <file>`).
2. In the TurkMaps project run (it refuses a SHA-1, a fingerprint with a missing byte, or a bad package name):

   ```bash
   node transit_data/play/make-assetlinks.cjs io.github.hero4mohamed.turkmaps <SHA-256 of Google's key> <SHA-256 of your upload key> --out <path to this folder>/.well-known/assetlinks.json
   ```

   (Replace the package name if you chose another one in the build step; it must match the app exactly.)
3. Commit and push this repository. Wait a minute, run the `curl` above again, and confirm the body now names the package.
4. (Optional) Google's own checker reads the same file and tells you whether it parses:
   `https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://hero4mohamed.github.io&relation=delegate_permission/common.handle_all_urls`
5. Install the app from the internal-testing link on a phone. **Success = the app opens with no address bar at the top.** If the bar is there, the fingerprint or package name does not match what is installed (the most common cause is using your upload key's fingerprint when Play re-signed the app, or the reverse for a sideloaded build).

## Rules for this repository

- Keep `.well-known/assetlinks.json` **valid JSON at all times**. A broken file turns every installed Android app into "a website in a tab" until it is fixed.
- Do not put anything private here: it is public.
- If the app is ever moved to its own domain, this file is no longer the one that matters and this repository can be deleted.
