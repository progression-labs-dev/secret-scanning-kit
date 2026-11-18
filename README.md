# Secret scanning kit

Status: draft, not yet reviewed.

Three checks that look for keys and tokens in your code. Layers 1 and 2 block the change. Layer 3 raises an alert. The setup guide in `docs/secret-scanning-setup-guide.pdf` explains why each check exists and what it misses. All three checks use gitleaks 8.30.1 and the same `.gitleaks.toml`.

| Layer | What it does | File |
|---|---|---|
| 1. Laptop hook | Blocks a commit that contains a recognised key | `.pre-commit-config.yaml`, `.gitleaks.toml` |
| 2. Pull request scan | Fails the pull request check when a key is found in the files | `.github/workflows/secret-scan-pr.yml` |
| 3. Weekly full-history scan | Scans all git history and opens or updates an issue when it finds a key | `.github/workflows/secret-scan-history.yml` |

## How to use it

An AI agent can do this for you: point it at `AGENTS.md`.


1. Copy the files above into your repository, keeping the same paths.
2. Set the alert recipient: `gh variable set SECRET_SCAN_ASSIGNEE --body "github-username"`, or add a repository variable with that name in the repository settings. If it is empty, the issue is still opened, with no assignee.
3. Install the `pre-commit` tool, then run `pre-commit install` once in each clone.
4. In the repository's branch protection or ruleset settings, mark the `gitleaks-files` check as required, and include administrators.
5. Add a `CODEOWNERS` entry for `.github/workflows/`, `.gitleaks.toml` and `.gitleaksignore`, and turn on required review from code owners. A pull request can otherwise edit the scanner settings and pass its own check.
6. Run the weekly workflow once by hand from the Actions tab and confirm it passes.

## What the weekly scan does

- Exit code 3 from gitleaks means a key was found. The workflow then opens an issue titled "Secret scan found a key", or adds a comment to the open one that the workflow itself created, and marks the run as failed. An issue with the same title made by a person is ignored. If the named user cannot be assigned, the issue opens without an assignee.
- Any other failure, such as a download error, a checksum mismatch or a broken config, marks the run as failed without a "key found" alert. Watch for failed runs as well as issues.
- Both workflows ignore inline `gitleaks:allow` comments, so a developer cannot hide a key with one. Use `.gitleaksignore` for reviewed exceptions.
- The gitleaks download is checked against a pinned SHA-256 digest. Update the version and the digest together.
- On a public repository, the issue and the run log show where the key is. Revoke the key first.

## Limits

- All three checks match known key formats and some generic patterns. A plain-word password with no recognisable pattern can pass.
- The hook only works on laptops where it is installed.
- Deleting a key from the code does not remove it from git history. Revoke the key first.
- To silence a finding you have already dealt with, add its fingerprint from the scan output to a `.gitleaksignore` file.

## Test status

Tested on a laptop with fake keys and the gitleaks 8.30.1 binary (checksum checked): it detected GitHub, OpenAI and Anthropic key formats, scanned test folders and Markdown files, honoured `.gitleaksignore`, and exited with 0 (clean), 3 (key found) and 1 (broken config). An inline `gitleaks:allow` comment hid a key by default, and `--ignore-gitleaks-allow` exposed it. The hook pinned to the 8.30.1 commit blocked a fake Anthropic key.

Tested on GitHub, by Progression Labs in a private throwaway repository with the final workflow files:

- With a fake key committed and then deleted, the weekly scan failed and opened one issue assigned to the variable's user. A second run added a comment and opened no second issue.
- An issue with the same title made by a person was ignored. A mistyped assignee opened one issue without an assignee.
- A broken config and a wrong checksum each failed the run without a "key found" alert.
- A pull request with a fake key only in `tests/` failed the check, and so did one whose key carried a `gitleaks:allow` comment.
- With the check required and administrators included, a failing pull request was blocked and a clean one was not.

Not tested: releases newer than 8.30.1, the weekly schedule firing on its own (only manual runs), required code-owner review, and pull requests from forks.

## Licence

The files in this repository (configuration, workflows and instructions) are under the MIT licence: see `LICENSE`. The setup guide in `docs/` is under CC BY 4.0: see `docs/LICENSE-CC-BY-4.0.txt`. Copyright Progression Labs.
