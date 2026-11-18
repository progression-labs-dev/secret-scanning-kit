# Instructions for AI agents: install this kit into a repository

If you have been asked to install this secret scanning kit into a target repository, follow this file. Otherwise ignore it. Read `README.md` first. Do the steps in order. Stop and ask the human where a step says **Ask**.

## Before you start

1. Confirm the target repository: its local path and its GitHub `owner/name`.
2. Check whether you have administrator rights on it. Branch protection and repository variables need them. If you do not, prepare the pull request and give the human a list of steps for an administrator.
3. **Ask** the human for the GitHub username that should receive alerts.
4. Look for files the target already has: `.pre-commit-config.yaml`, `.gitleaks.toml`, `.gitleaksignore`, and any secret scan workflow. Merge into them. Never overwrite them.

## Steps

1. Create a new branch in the target repository.
2. Copy `.github/workflows/secret-scan-pr.yml` and `.github/workflows/secret-scan-history.yml`. Check that the `pull_request` trigger and the default branch name fit the target.
3. Add the gitleaks block from `.pre-commit-config.yaml` to the target's file, and keep its existing hooks. If there is none, copy the file.
4. For `.gitleaks.toml`: if the target has one, keep its allowlist and make sure it still contains `[extend]` with `useDefault = true`. If there is none, copy this one and delete the example allowlist entry.
5. Open a pull request with these changes. Do not merge it. The human decides.
6. **Ask**, then set the alert recipient: `gh variable set SECRET_SCAN_ASSIGNEE --body "<username>" -R <owner/repo>`.
7. **Ask**, then change branch protection (administrator only): mark the `gitleaks-files` check as required, include administrators, add a `CODEOWNERS` entry for `.github/workflows/`, `.gitleaks.toml` and `.gitleaksignore`, and require code-owner review. If there is any doubt, only list the steps.

## Verify with fake keys only

Make a fake key from a random string that starts with `ghp_` followed by 36 letters and digits. Never use a real key.

1. **Hook.** Run `pre-commit install`. Save the fake key in a new file, stage it and try to commit. The commit must be blocked. Then delete the file and unstage it.
2. **Pull request scan.** Open a throwaway branch with the fake key and a pull request from it. The `gitleaks-files` check must fail. Close the pull request without merging, and delete the throwaway branch, or the weekly history scan will flag it. The fake key stays in the pull request's history on GitHub, so do this step in a throwaway repository if the target might ever become public.
3. **History scan.** Run `gh workflow run secret-scan-history.yml -R <owner/repo> --ref <your-branch>`, which works before the pull request is merged, and confirm it passes on clean history. Test the alert path only in a throwaway repository: commit a fake key, delete it in the next commit, and run again. One issue must open.

Report to the human what you ran and what you did not. Do not say a layer works unless you ran its check.

## Rules

- Never create, paste or commit a real secret. If a scan finds a real one, stop and tell the human. Revoke the key first. Do not rewrite history as a first step.
- Do not invite anyone as a collaborator, change billing or seats, or make a repository public.
- Do not change a pinned version or digest without testing. To update gitleaks, change the version and its SHA-256 digest together in both workflows and in `.pre-commit-config.yaml` (use the commit of the release tag), then rerun the checks above. Take the digest from the release's checksums file and verify it.
- Use the human's configured commit identity. If their email is private, use their GitHub noreply address. Ask if unsure.
- Match the target's existing style. Change only what this install needs.

## Known limits

See the Limits and Test status sections of `README.md`. In short: a plain-word password can pass, the hook only works where it is installed, and the weekly schedule firing on its own has not been tested.
