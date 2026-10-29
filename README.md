# OnePage releases

Installers and update files for **OnePage**, a local-first editor for one-page technical design
artifacts: architecture overviews, engineering one-pagers, trade studies and similar pages. A
person, the `onepage` command-line tool and an MCP agent all edit the same document through one
deterministic engine.

This repository holds **release files only** (installers, signatures and the update manifest).
The source code is kept elsewhere.

## Download

Open the [latest release](https://github.com/echozulucode/onepage-releases/releases/latest) and
download the files you need:

| File | What it is |
|---|---|
| `OnePage_<version>_x64-setup.exe` | The Windows installer |
| `OnePage_<version>_x64-setup.exe.sig` | The installer's update signature (see [Verifying a download](#verifying-a-download)) |
| `onepage-tools-<version>-<target>.tar.gz` | The `onepage` CLI and the `onepage-mcp` server as standalone programs, for scripts and MCP clients |
| `latest.json` | The update manifest that installed copies of OnePage read; you don't need to download it |

Each release's page lists what changed in that version.

## Install (Windows)

Requires Windows 10 or 11 (x64).

1. Run `OnePage_<version>_x64-setup.exe`.
2. It installs **for your user only**, into `%LOCALAPPDATA%\OnePage`, so no administrator prompt
   appears. If the WebView2 runtime is missing, the installer fetches it.
3. Start OnePage from the Start menu. Double-clicking an `.opd` file opens it in OnePage.

> **Windows SmartScreen will probably warn you.** The installer is not yet Authenticode-signed,
> so Windows may show "Windows protected your PC". Choose **More info**, then **Run anyway**.
> Updates are still cryptographically signed (see below).

Installing a newer version over an older one keeps your settings and documents. To uninstall, use
**Settings ▸ Apps ▸ Installed apps ▸ OnePage ▸ Uninstall**. Your `.opd` files are not touched.

## Automatic updates

OnePage checks this repository for a newer version:

- **at start-up**, at most once a day (turn this off under **Help ▸ Check for updates at
  start-up**), and
- whenever you choose **Help ▸ Check for updates…**.

When an update is available, a small notice offers **Restart to update**, **Later** or **What's
new**. If you have unsaved changes, OnePage asks you to save them first. It then downloads the
installer, **checks its signature against the public key built into OnePage**, installs it and
restarts. A download whose signature does not match is refused.

The updater arrived after OnePage 0.1.0-beta.6. A copy installed from 0.1.0-beta.6 or earlier
has no updater and needs one manual install of a newer release; from then on it updates itself.

## Verifying a download

Each installer is signed with the OnePage updater key, a [minisign](https://jedisct1.github.io/minisign/)
key. Its public half is:

```
untrusted comment: minisign public key: C366D6A8AE520C51
RWRRDFKuqNZmw7Ck7oNosYyrC6Leo0+/JWtOvdKzSqHxTge8iuqp7B0G
```

The `.sig` file next to the installer is that signature in base64 (the format the Tauri updater
uses). To check a download by hand with minisign:

```powershell
# PowerShell: decode the .sig, then verify
$sig = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String((Get-Content -Raw OnePage_<version>_x64-setup.exe.sig)))
Set-Content -NoNewline -Path OnePage.minisig -Value $sig
minisign -Vm OnePage_<version>_x64-setup.exe -x OnePage.minisig -P RWRRDFKuqNZmw7Ck7oNosYyrC6Leo0+/JWtOvdKzSqHxTge8iuqp7B0G
```

`Signature and comment signature verified` means the file is the one that was released.

## Command-line tool and MCP server

The installer puts `onepage.exe` (CLI) and `onepage-mcp.exe` (MCP server) next to the app in
`%LOCALAPPDATA%\OnePage`. The installer does not add them to `PATH`, so use their full paths or
the standalone programs in the `onepage-tools-*.tar.gz` archive.

- `onepage-mcp --headless` gives an MCP client a private document session.
- `onepage-mcp --attach active` connects to the open desktop window you used last. Turn on
  **Automation** in that window's status bar first; it starts off.
- `onepage inspect --compat FILE --format text` reports, without changing the file, which
  OnePage versions can open it and what each one needs.

## Files and compatibility

OnePage saves documents as `.opd` files. Every release keeps this promise:

- **A file saved by an older OnePage opens in every newer one**, draws the same page and loses
  nothing. Each release is tested against documents written by earlier released versions.
- **An older OnePage refuses a file that uses a feature it doesn't know** and says which feature
  is missing, leaving the file untouched, rather than dropping content. Keep everyone who shares
  files on the same version or newer.
- If a future version ever needs to upgrade a file's format, the first save keeps the original
  beside it (`<name>.opd.bak-v<N>`).

## Release channel

Releases are 0.x betas. They are published as normal releases (not "pre-release"), because
GitHub's "latest release" link, which the updater reads, skips pre-releases.

## Problems

If something goes wrong, open an issue in this repository with the OnePage version
(**Help ▸ About OnePage**), what you did, and what you expected.

## License

OnePage is licensed under the Apache License 2.0. The license text is shown by the installer.
