# Paparats catalog

The official catalog of [Paparats](https://github.com/paparats-app/paparats), a curated app store
for Windows. This repository holds what people decide (which apps, how they install, the texts)
and what the bot found (versions, files, hashes, signers). GitHub Pages serves the signed result
at `https://paparats-app.github.io/catalog/v1/`.

Paparats installs every app straight from its authors' GitHub Releases. Nothing is mirrored or
re-hosted here.

## What is where

| Path | Who writes it | What it is |
|---|---|---|
| `apps/<id>.toml` | people | One app: where its releases are, how to install it silently, how to recognise it among installed apps, texts in English, Belarusian and Russian |
| `editorial.toml` | people | What Home and Browse feature: hero slides, banners, collections |
| `icons/<id>.png` | people | The app's icon, 256×256, taken from its own repository |
| `lock.json` | the bot | The release and file picked for every app: version, URL, size, SHA-256, signer, scan result, repository facts |
| `.github/bot-ref` | people | The commit of `paparats-app/paparats` the bot is built from |

## How an app gets updated

Once a day the [Resolve](.github/workflows/resolve.yml) workflow runs the bot
(`crates/paparats-catalog-bot` in the main repository) on a Windows runner. For every app it:

1. takes the latest stable release and finds the file by the app's pattern;
2. downloads it, computes SHA-256 and compares it with GitHub's own digest;
3. checks the Authenticode signature (EXE, MSI) and pins the signer it saw first: a release
   signed by someone else is held back until a person decides;
4. scans the file with Microsoft Defender;
5. keeps a newer release in **quarantine for 48 hours** before it replaces the shipped one, since
   compromised releases are usually noticed within hours;
6. records stars, license, the repository's age and the links from `.github/FUNDING.yml`.

The bot commits `lock.json`, and [Publish](.github/workflows/publish.yml) builds the four
manifests, signs them with the catalog key (Ed25519) and deploys the site. The Paparats client
verifies the signature and refuses anything else. It also installs a file only if the file's
hash is the one the bot pinned. Releases that are held back, apps that fail and stale projects
go to the "Bot: needs attention" issue.

Once a week [Smoke install](.github/workflows/smoke.yml) installs every app on a clean Windows
runner with the client's own installer, checks that the client's inventory finds it and
uninstalls it. Its summary lists the Uninstall entries each install added, which is how a
detection rule gets fixed.

## Adding an app

Suggest it with an [issue](../../issues/new/choose), or open a pull request with
`apps/<id>.toml` and `icons/<id>.png`. A minimal file:

```toml
id = "example"                 # lower-case slug, same as the file name
name = "Example"
publisher = "Example Team"
category = "utilities"         # productivity, development, design, media, utilities,
                               # system, lifestyle, games, education

[source]
repo = "owner/example"
asset = '^Example-[\d.]+-x64-setup\.exe$'   # a regular expression for the file name

[install]
strategy = "exeSilent"         # msi, exeSilent, msix, portableZip, portableExe
args = ["/S"]                  # exeSilent only
# scope = "machine"            # the installer asks for administrator rights
# launch = "Example.exe"       # portable apps: the program to start
# signer = "none"              # the author signs nothing (default "auto": pin the signer)

[detection]                    # every strategy but the portable ones
displayNameRegex = 'Example.*' # the whole DisplayName in "Installed apps", any case
# displayNames = ["Example"]   # exact names; versions and "(x64)" are ignored
# productCodes = ["{GUID}"]    # the Uninstall key name: an MSI product code, "App_is1"
# msixFamilyNames = ["Publisher.App_abcdefghjkmnp"]

[text.en]
tagline = "One short line."
long = "Two or three sentences about what it does."
[text.be]
tagline = "…"
long = "…"
[text.ru]
tagline = "…"
long = "…"
```

A young project for the Rising shelf also gets the editors' note:

```toml
[rising.note]
en = "Why we picked it."
be = "…"
ru = "…"
```

The [Validate](.github/workflows/validate.yml) workflow checks every pull request.

What we take: the app publishes a Windows build in its official repository's GitHub Releases,
its license allows free use, it shows no ads and bundles nothing. We do not take system
"optimisers" and cleaners, or unofficial clients of commercial services.

## License

The catalog's files and texts are under the [MIT License](LICENSE). The icons belong to their
apps' authors and are used to identify the apps.
