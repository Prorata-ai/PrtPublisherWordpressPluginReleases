# Gist Answers WordPress Plugin — Releases

This repository publishes production builds of the **Gist Answers** WordPress plugin. Each release is a git tag plus a GitHub Release with the installable plugin zip attached.

## Release rules

- **One tag per release**, named `vX.Y.Z` (for example `v1.0.16`).
- **The version must match in three places** inside the plugin, and the tag must match all of them:
  - the plugin header in `gist-answers.php`: ` * Version:            X.Y.Z`
  - the constant in `gist-answers.php`: `define( 'GIST_ANSWERS_VERSION', 'X.Y.Z' );`
  - `readme.txt`: `Stable tag:         X.Y.Z`
- **Each tag gets a GitHub Release** with a single asset named exactly `gist-answers.zip`.
- **Never move, delete, or reuse a tag.** To fix a bad release, publish the next patch version (e.g. `v1.0.16` → `v1.0.18`).
- **Keep settings and stored data compatible between versions.** The partner can roll a site back to the previous version, so a release must not make a one-way data change (destructive migrations, renamed or reformatted options, dropped tables, etc.) without talking to them first.

## What goes in `gist-answers.zip`

The zip has exactly one top-level folder, `gist-answers/`, with `gist-answers.php` directly inside it:

```
gist-answers.zip
└── gist-answers/
    ├── gist-answers.php
    ├── readme.txt
    ├── blocks/
    ├── includes/
    ├── languages/
    ├── public/          # built front-end assets
    ├── templates/
    ├── vendor/          # production Composer autoloader only
    └── vip-manifest.yaml
```

**Include** everything the plugin needs to run: PHP source, built `public/` assets, and the production `vendor/` autoloader (`composer install --no-dev`).

**Leave out** dev dependencies, tests, and CI files, for example:

- `.github/`, `bin/`, `tests/`
- `phpunit.xml.dist`, `playwright.config.ts`, `.phpcs.xml*`
- `node_modules/`, `package.json`, `package-lock.json`
- `composer.lock`, `.git*`, `.editorconfig`, and other dotfiles

## How to cut a release

The steps below use `v1.0.16` as the example. Run steps 1–2 in the plugin **source** repo and steps 4–6 in **this** repo.

### 1. Build the production release of the plugin

Build plugin from the PrtPublisherWordpressPlugin repo's Actions tab, specifying

* The `main` branch
* A target environment of `all`
* The next applicable version (eg. 1.0.16)
* A release description overview of the changes made

### 2. Download the production plugin build

Available in the Actions tab. Download the production version of the plugin and create two versions, named `gist-answers.zip` and `gist-answers-wp-plugin-<version>.zip`

### 3. Tag and publish the GitHub Release

Copy the `gist-answers-wp-plugin-<version>.zip` into the `production` directory. Add it and push to main.

Tag and create the release:

```sh
VERSION=1.0.17

git tag -a "v$VERSION" -m "Gist Answers Wordpress Plugin v$VERSION release"
git push origin "v$VERSION"

gh release create "v$VERSION" gist-answers.zip \
  --title "Gist Answers $VERSION" \
  --notes "<change log message entered when the plugin was built>"
```

The uploaded asset name must be exactly `gist-answers.zip`. Only commit the versioned zip to the repo, the generically named zip file lives in the Release.

Check the result:

```sh
gh release view "v$VERSION"
```

### 4. Email the partner

Send an email saying the release is ready, with:

- the version and a link to the GitHub Release
- the changelog entry
- any notes on data or settings changes (there should be no one-way changes unless already agreed)

They review it, add it to the platform, and choose the release date. It goes live on VIP's weekly platform release. Releases don't reach customer sites automatically, so there's no need to coordinate timing with them.

## Fixing a bad release

Don't edit, move, or delete the existing tag or swap its zip. Fix the problem in the plugin source, bump to the next patch version, and run the full process again. Mention in the email which release the new one replaces and why.
