# frtag-firmware-inhouse

In-house HTTPS host for frtag2 OTA firmware images, served via GitHub Pages.

This is the **bench twin** of `farmranger-firmware`. Same layout, same
publish script, same manifest format - the only difference is who reads it:

| Host repo | Pages base URL | Who fetches it |
|---|---|---|
| `farmranger-firmware` | `https://ruangdejager.github.io/farmranger-firmware/` | every deployed field unit |
| `frtag-firmware-inhouse` | `https://ruangdejager.github.io/frtag-firmware-inhouse/` | only fr9 units built with `SWITCH_TAGFOTA_INHOUSE_HOST` |

Nothing published here can reach a field unit: a field fr9 has the in-house
URL compiled out entirely (see `fr9_application`, `fr_app/inc/tag_fota.h`).

## URLs

```
https://ruangdejager.github.io/frtag-firmware-inhouse/frtag2-firmware/version.json
https://ruangdejager.github.io/frtag-firmware-inhouse/frtag2-firmware/frtag2_<M>_<m>_<p>.bin
```

The OTA flow always starts with `version.json`, then downloads the file its
`"file"` field names - there is no `latest.bin` fallback.

## Layout

```
frtag-firmware-inhouse/
├── frtag2-firmware/
│   ├── version.json                Manifest - version / file / size / crc32 / xor
│   └── frtag2_<M>_<m>_<p>.bin      App image, raw binary (what the modem downloads)
├── .nojekyll                       Serve files verbatim, no Jekyll processing
└── README.md
```

Older versioned `.bin` files stay alongside so a rollback can be pinned by
editing `version.json` back to one of them.

## Publishing

The frtag2 post-build script (`C:\Code\frtag2\build_scripts\create_release_hex_files.ps1`)
writes the `.bin` and `version.json` into **both** host repos on every build.
It does not commit or push. After a build:

1. Check `frtag2-firmware/version.json` names the `.bin` that sits next to it.
2. `git add . && git commit -m "release v<M>.<m>.<p>" && git push`
3. GitHub Pages redeploys automatically - usually under a minute.

Pushing *this* repo affects bench units only. Pushing `farmranger-firmware`
is the one that goes to the field.
