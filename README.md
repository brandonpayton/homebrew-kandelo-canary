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

A successful write run proves all of the following together:

1. a public third-party tap can call Kandelo's reviewed reusable publisher;
2. repository identity and Homebrew tap identity remain distinct and correct;
3. the repository's built-in `GITHUB_TOKEN` can create
   `homebrew-kandelo-canary/hello` in GitHub Container Registry (GHCR);
4. the first package is public without a visibility-changing API call or a
   personal access token; and
5. the publisher can anonymously read the exact uploaded digest before it
   commits Formula and Kandelo sidecar metadata.

The `homebrew-` prefix in the GHCR namespace is intentional. It binds the
package to this public source repository; shortening the destination to
`kandelo-canary/hello` exercises a different package-creation path and is not
this canary.

## Maintainer procedure

First confirm that the package does not already exist when testing first-package
creation. The conservative sequence starts with the read-only build and
verification graph:

```bash
gh api -X POST repos/brandonpayton/homebrew-kandelo-canary/dispatches \
  -f event_type=dry-run-kandelo-bottles \
  -f 'client_payload[formulae]=hello' \
  -f 'client_payload[arches]=wasm32'
```

After the dry run succeeds, request the write publication:

```bash
gh api -X POST repos/brandonpayton/homebrew-kandelo-canary/dispatches \
  -f event_type=publish-kandelo-bottles \
  -f 'client_payload[formulae]=hello' \
  -f 'client_payload[arches]=wasm32'
```

For the initial first-package canary, maintainers may intentionally defer the
dry run when fast end-to-end proof is more valuable than avoiding a failed
upload attempt. The write workflow contains the same pre-upload planning,
build, and local handoff validation. It then exercises the critical behavior a
dry run cannot test: public package creation followed by credential-free digest
readback. Record that throughput decision with the run evidence.

Do not add `HOMEBREW_GITHUB_PACKAGES_TOKEN` or another package secret. The
caller grants a permission ceiling of `actions: read`, `contents: write`, and
`packages: write`; the reusable workflow narrows each job to the permissions it
actually needs. The uploader cannot write tap contents, and the tap finalizer
cannot write packages.

The normal publisher must fail before finalization if anonymous digest readback
fails. A successful run leaves the bottle under the repository-rooted GHCR
namespace and commits the generated `bottle do` block and `Kandelo/` sidecars
to this repository.
