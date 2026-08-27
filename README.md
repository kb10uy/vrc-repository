# VPM repository

A VRChat package repository, published to GitHub Pages. Everything is built by
[vpmrepo-neobuilder][nb]: the listing at `<site>/index.json` that VCC, vrc-get
and ALCOM consume, and the landing page that lists the packages for humans.

This repository holds configuration only. The layouts, stylesheet and
translations live in the [`site-template`][st] Hugo Module and arrive at build
time, so updates to the site's appearance come from `hugo mod get -u` rather
than from copying files around.

[nb]: https://github.com/kb10uy/vpmrepo-neobuilder
[st]: https://github.com/kb10uy/vpmrepo-neobuilder/tree/main/site-template

## Setting it up

1. Click **Use this template** and create your own repository.

2. Edit `source.toml`. `[listing] url` has to name the published listing, which
   is `<your Pages URL>/index.json`. Replace the example `[[github]]` entry with
   the repositories you publish. Every key is documented in the
   [design document][design].

3. Edit `hugo.toml`. `baseURL` has to be your Pages URL, and `title`,
   `params.description` and `params.links` are yours to fill in.

4. Point the module at your own repository name, and record the version of the
   site template you are on:

   ```sh
   hugo mod init github.com/<owner>/<repository>
   hugo mod get github.com/kb10uy/vpmrepo-neobuilder/site-template
   git add go.mod go.sum && git commit -m "Pin the site template"
   ```

5. In **Settings → Pages**, set **Source** to **GitHub Actions**.

6. Push to `main`. The workflow rebuilds on every push, once a day, and on
   demand from the Actions tab.

[design]: https://github.com/kb10uy/vpmrepo-neobuilder/blob/main/docs/design.md

## What the build does

```text
source.toml ──neobuilder──> static/index.json   the VPM listing, served verbatim
                        └─> data/listing.json   what the pages render
                                    │
                            hugo ───┴─────────> public/
```

Neither generated file is committed; both are rebuilt from `source.toml` every
time. The previous `index.json` is restored from the Actions cache and passed as
`--cache`, which lets the build skip re-downloading archives it has already
hashed. Losing that cache costs time and nothing else.

## Local preview

```sh
vpmrepo-neobuilder build --source source.toml static/index.json data/listing.json
hugo server
```

`hugo server` on its own works too and shows a repository with no packages.
Building the site needs Go on `PATH`, because `hugo mod` shells out to it.

## Updating the site template

```sh
hugo mod get -u github.com/kb10uy/vpmrepo-neobuilder/site-template
```

Commit the resulting `go.mod` and `go.sum`. The `hugo.toml` keys the module
expects are listed in [its README][st].
