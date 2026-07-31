# p6m7g8-actions/p6-gh-pr-auto-approve

- [p6m7g8-actions/p6-gh-pr-auto-approve](#p6m7g8-actionsp6-gh-pr-auto-approve)
  - [Usage](#usage)
  - [Requirements](#requirements)
  - [Inputs](#inputs)
  - [Which identity approves](#which-identity-approves)

## Usage

```yaml
    permissions:
      pull-requests: write
    steps:
      - name: Auto Approve PR
        uses: p6m7g8-actions/p6-gh-pr-approve@main
        with:
          bot_token: ${{ secrets.P6_A_GH_TOKEN }}
```

## Requirements

Two settings gate the bot-authored path, and both fail in ways that do not
mention the real cause.

**`permissions: pull-requests: write` on the calling job.** The bot-authored
path approves with the caller's `GITHUB_TOKEN`, so without this grant it fails
with `403 Resource not accessible by integration`. A job-level `permissions:`
block satisfies this even when the org's default workflow permission is `read`,
which is the case in `p6m7g8-actions`.

**"Allow GitHub Actions to create and approve pull requests" enabled for the
org.** This is a separate toggle from the default workflow permissions, exposed
as `can_approve_pull_request_reviews`. With it off, `github-actions[bot]` cannot
approve anything regardless of token or permissions, and the error is actively
misleading:

```text
Unprocessable Entity: "GitHub Actions is not permitted to approve pull requests."
This typically happens when you try to approve the pull request with the same
user account that created the pull request.
```

The suggested cause is wrong in that case. Check the org setting first:

```sh
gh api "orgs/$ORG/actions/permissions/workflow" \
  --jq '{default_workflow_permissions, can_approve_pull_request_reviews}'
```

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `owner` | yes | `pgollucci` | User login whose PRs are auto-approved. Must be a user login, not an org name. |
| `bot` | yes | `p6m7g8-automation` | Bot login whose PRs are auto-approved. |
| `bot_token` | yes | n/a | Token used to submit the approval, on the non-`bot` path only. |

A PR is approved only when it is not a draft, does not carry the `do-not-merge`
label, and either carries the `auto-approve` label or was opened by `owner` or
`bot`.

## Which identity approves

GitHub refuses to let an identity approve its own pull request, so the approving
token depends on who opened the PR:

| PR author | Approving identity |
| --- | --- |
| `bot` | `github-actions[bot]`, via the caller's `GITHUB_TOKEN` |
| anyone else | `bot`, via `bot_token` |

Every PR opened by the bot, meaning file distribution and dependency upgrades, is
authored by the same account that holds `bot_token`. Approving those with
`bot_token` always failed with
`Unprocessable Entity: "Review Can not approve your own pull request"`, which is
one of the two reasons distribution PRs never merged. The other was unsigned
commits; see `p6m7g8-actions/p6-gh-distributor`. `github-actions[bot]` is never
the author in this fleet, so it can approve what the bot cannot.

One consequence: a bot-authored PR is approved by `github-actions[bot]`, which
satisfies a `required_approving_review_count` rule but can never satisfy a
CODEOWNERS or named-reviewer requirement, because an app identity cannot be a
code owner. No repo in this fleet sets `require_code_owner_review`, so approval
and mergeability coincide today. If that changes, a bot PR will show as approved
and still be blocked, and there is no way to fix it here, since the bot cannot
approve itself regardless of which token is used.
