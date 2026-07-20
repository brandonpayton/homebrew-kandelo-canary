# Kandelo third-party Homebrew tap canary

This public repository exercises Kandelo's supported third-party Homebrew
publisher contract with the conventional repository/tap identity pair:

- GitHub repository: `brandonpayton/homebrew-kandelo-canary`
- Homebrew tap: `brandonpayton/kandelo-canary`
- bottle namespace: `ghcr.io/brandonpayton/homebrew-kandelo-canary/<formula>`

The protected dispatch workflows publish with this repository's built-in
`GITHUB_TOKEN`. They do not accept or use a package personal access token.

This repository is an integration canary, not a supported end-user tap.

## What this proves

Together, the two successful write runs prove all of the following:

1. a public third-party tap can call Kandelo's reviewed reusable publisher;
2. repository identity and Homebrew tap identity remain distinct and correct;
3. the repository's built-in `GITHUB_TOKEN` can create packages such as
   `homebrew-kandelo-canary/hello` in GitHub Container Registry (GHCR);
4. the repository's first package was public without a visibility-changing API
   call or a personal access token;
5. the publisher can anonymously read the exact uploaded digest before it
   commits Formula and Kandelo sidecar metadata; and
6. the `m4` canary can resolve its real `kandelo-dev/tap-core/dash`
   dependency, compose the resulting cross-tap closure into a VFS image, and
   boot that exact image under Node and Chromium.

The `homebrew-` prefix in the GHCR namespace is intentional. It binds the
package to this public source repository; shortening the destination to
`kandelo-canary/hello` exercises a different package-creation path and is not
this canary.

## Maintainer procedure

The two write canaries are intentionally independent. `hello` is the smallest
third-party/public-package proof and does not request VFS closure acceptance:

```bash
gh api -X POST repos/brandonpayton/homebrew-kandelo-canary/dispatches \
  -f event_type=publish-kandelo-bottles \
  -f 'client_payload[formulae]=hello' \
  -f 'client_payload[arches]=wasm32' \
  -F 'client_payload[require_vfs_acceptance]=false'
```

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
credential-free digest readback. Keep the two runs separate so a VFS or
cross-tap failure cannot obscure the minimal public-package result.

Do not add `HOMEBREW_GITHUB_PACKAGES_TOKEN` or another package secret. The
caller grants a permission ceiling of `actions: read`, `contents: write`, and
`packages: write`; the reusable workflow narrows each job to the permissions it
actually needs. The uploader cannot write tap contents, and the tap finalizer
cannot write packages.

The normal publisher must fail before finalization if anonymous digest readback
fails. A successful run leaves the bottle under the repository-rooted GHCR
namespace and commits the generated `bottle do` block and `Kandelo/` sidecars
to this repository.
