# overmind actions

Github Actions to run Terraform Impact Analysis on PRs


# Development

Install [nektos/act](https://github.com/nektos/act) and run

```
gh act pull_request -s GITHUB_TOKEN="$(gh auth token)" -s CLONE_ALL="${CLONE_ALL}" -s OVM_TOKEN="${OVM_TOKEN}"
```

to try out the `selftest` action locally. It's much faster than commit/push.
