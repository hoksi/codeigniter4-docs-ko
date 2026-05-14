# codeigniter4-docs-ko

CodeIgniter 4 User Guide Korean Translation.

## Files in this repository

- `LC_MESSAGES/*.po`: translation source files
- `LC_MESSAGES/*.mo`: compiled message catalogs
- `LC_MESSAGES/**/*.html`: generated HTML output for review

## Translation workflow

1. Update or generate `.po` files from the Sphinx gettext output.
2. Translate entries in `msgstr`.
3. Rebuild HTML and review the generated pages.
4. Keep markup rules consistent with the translation guide.

## Translation guide

Follow `TRANSLATION_GUIDE.md` in this folder for the Korean translation rules, especially:

- reST markup and Korean particle handling
- backslash usage after inline markup
- build and verification workflow
