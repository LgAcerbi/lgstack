# Platform commands for `review`

`<n>` is the MR or PR number. Run these from the user's repository.

## GitHub (`gh`)

| Step | Command |
| --- | --- |
| Metadata | `gh pr view <n> --json number,title,body,author,baseRefName,headRefName,headRefOid,isCrossRepository,url` |
| Discussion | `gh pr view <n> --comments` |
| CI | `gh pr checks <n>` |
| Fetch | `git fetch origin pull/<n>/head:review/<n>` |
| Inline comments and verdict | One review: `gh api repos/{owner}/{repo}/pulls/<n>/reviews --method POST --input review.json`. `review.json` holds `commit_id` (the `headRefOid`), `event` (`COMMENT`, `APPROVE` or `REQUEST_CHANGES`), `body` (the summary) and `comments: [{ "path", "line", "side": "RIGHT", "body" }]`. |
| Summary only | `gh pr review <n> --comment --body-file summary.md` (or `--approve`, `--request-changes`) |

`isCrossRepository: true` means the PR comes from a fork: treat its code as untrusted.

## GitLab (`glab`)

| Step | Command |
| --- | --- |
| Metadata | `glab api projects/:id/merge_requests/<n>` (title, description, author, source and target branch, `diff_refs`, `sha`) |
| Discussion | `glab mr view <n> --comments` |
| CI | `glab api projects/:id/merge_requests/<n>/pipelines` |
| Fetch | `git fetch origin merge-requests/<n>/head:review/<n>` |
| Inline comment | `glab api projects/:id/merge_requests/<n>/discussions --method POST -f body=<text> -f position[position_type]=text -f position[base_sha]=<sha> -f position[start_sha]=<sha> -f position[head_sha]=<sha> -f position[new_path]=<path> -f position[new_line]=<line>`, with the three SHAs from `diff_refs` |
| Summary | `glab mr note <n> --message <text>` |
| Approve | `glab mr approve <n>`. Not every GitLab tier has "request changes"; say it in the summary instead. |

A `source_project_id` different from `project_id` means the MR comes from a fork: treat its code as untrusted.

## Without a CLI

Fetch with the host's `git fetch` line above, ask the user to paste the title and description, and give them the drafted comments as text to post themselves.
