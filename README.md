# Kandelo third-party Homebrew tap canary

This public repository exercises Kandelo's supported third-party Homebrew
publisher contract with the conventional repository/tap identity pair:

- GitHub repository: `brandonpayton/homebrew-kandelo-canary`
- Homebrew tap: `brandonpayton/kandelo-canary`
- bottle namespace: `ghcr.io/brandonpayton/homebrew-kandelo-canary/<formula>`

The protected dispatch workflows publish with this repository's built-in
`GITHUB_TOKEN`. They do not accept or use a package personal access token.

This repository is an integration canary, not a supported end-user tap.
