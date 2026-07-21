# Kandelo third-party Homebrew tap canary

This public repository exercises Kandelo's supported third-party Homebrew
publisher contract with the conventional repository/tap identity pair:

- GitHub repository: `brandonpayton/homebrew-kandelo-canary`
- Homebrew tap: `brandonpayton/kandelo-canary`
- bottle namespace: `ghcr.io/brandonpayton/homebrew-kandelo-canary/<formula>`

The protected dispatch workflows publish with this repository's built-in
`GITHUB_TOKEN`. They do not accept or use a package personal access token.

This repository is an integration canary, not a supported end-user tap.

## Formula support source

The publisher intentionally requires each Formula to load its support module
from the same immutable tap checkout. That keeps Formula source, helper code,
and helper-owned runtime files in one reviewed source closure instead of
executing mutable code from another tap.

For that reason, the publisher-consumable part of
`Kandelo/formula_support/` is vendored here as one complete unit. Sync the
support module and every path outside its top-level `test/` directory from a
reviewed `Kandelo-dev/homebrew-tap-core` commit; do not copy only the Ruby file
or patch individual imports to satisfy a newer publisher check. The current
runtime support closure is synchronized from
`a821def9799bf45d742aae66bb34ffd50bbb41e5`. Its general cross-tap dependency
isolation recognizes both the attested primary tap and
`kandelo-dev/tap-core`, preserving same-tap dependencies without hard-coding
this canary's owner or tap name.

The top-level `test/` directory is intentionally tap-local and excluded from
bottle source identity. It carries the same shared helper tests where they
apply, plus assertions about this canary's own Formula, without importing
first-party Formula inventory tests.

## What this proves

The completed `m4` write proves all of the following:

1. a public third-party tap can call Kandelo's reviewed reusable publisher;
2. repository identity and Homebrew tap identity remain distinct and correct;
3. the repository's built-in `GITHUB_TOKEN` can create
   `homebrew-kandelo-canary/m4` in GitHub Container Registry (GHCR);
4. the package was public without a visibility-changing API
   call or a personal access token;
5. the publisher can anonymously read the exact uploaded digest before it
   commits Formula and Kandelo sidecar metadata; and
6. the `m4` canary can resolve its real `kandelo-dev/tap-core/dash`
   dependency, compose the resulting cross-tap closure into a VFS image, and
   boot that exact image under Node and Chromium.

The `homebrew-` prefix in the GHCR namespace is intentional. It binds the
package to this public source repository; shortening the destination to
`kandelo-canary/m4` exercises a different package-creation path and is not
this canary.

## Maintainer procedure

`m4` is the smallest dependency-bearing proof in this canary. Its Formula
truthfully uses the core tap's `dash` as the configured runtime shell. This
write must produce cross-tap closure evidence and boot the composed VFS image
under both supported hosts:

```bash
gh api -X POST repos/brandonpayton/homebrew-kandelo-canary/dispatches \
  -f event_type=publish-kandelo-bottles \
  -f 'client_payload[formulae]=m4' \
  -f 'client_payload[arches]=wasm32' \
  -F 'client_payload[require_vfs_acceptance]=true'
```

The write workflow contains the same pre-upload planning, build, and local
handoff validation as the dry-run workflow. It additionally exercises the
critical behavior a dry run cannot test: public package creation followed by
credential-free digest readback. The completed run also proves cross-tap VFS
composition and immutable release publication, so the earlier single-package
pilot is no longer part of the active canary.

Do not add `HOMEBREW_GITHUB_PACKAGES_TOKEN` or another package secret. The
caller grants a permission ceiling of `actions: read`, `contents: write`, and
`packages: write`; the reusable workflow narrows each job to the permissions it
actually needs. The uploader cannot write tap contents, and the tap finalizer
cannot write packages.

The normal publisher must fail before finalization if anonymous digest readback
fails. A successful run leaves the bottle under the repository-rooted GHCR
namespace and commits the generated `bottle do` block and `Kandelo/` sidecars
to this repository.
