# FreeREAC/.github

Organisation defaults for FreeREAC.

- `profile/README.md` is the page shown on the organisation's front page: what FreeREAC
  is and which repository holds what.
- `.github/workflows/sanitize-guard.yml` refuses a push that carries a site, venue or
  client name or a rig address.
- `SECURITY.md` is the security policy of every repository that has none of its own.

Packages are built and published by the shared release workflows of
[FreeMixer/.github](https://github.com/FreeMixer/.github), which each repository calls at `@v1`.

Licence: GPL-3.0-or-later (`LICENSE`).
