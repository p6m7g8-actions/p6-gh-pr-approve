# p6m7g8-actions/p6-gh-pr-auto-approve

- [p6m7g8-actions/p6-gh-pr-auto-approve](#p6m7g8-actionsp6-gh-pr-auto-approve)
  - [Usage](#usage)
  - [Inputs](#inputs)

## Usage

```yaml
      - name: Auto Approve PR
        uses: p6m7g8-actions/p6-gh-pr-auto-approve@main
        with:
          bot_token: ${{ secrets.P6_A_GH_TOKEN }}
```

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `owner` | yes | `pgollucci` | User login whose PRs are auto-approved. Must be a user login, not an org name. |
| `bot` | yes | `p6m7g8-automation` | Bot login whose PRs are auto-approved. |
| `bot_token` | yes | n/a | Token used to submit the approval. |

A PR is approved only when it is not a draft, does not carry the `do-not-merge` label, and either carries the
`auto-approve` label or was opened by `owner` or `bot`.
