## GitHub CLI: What and Why

* `gh` is the official CLI for GitHub
* Targets GitHub features that `git` does not cover

    * issues, pull requests, releases, workflows, notifications, secrets, API
* Works in terminals and scripts
* Auth once, reuse across repos and orgs

### Setup and Auth

Install from package manager, then:

```bash
gh --version
gh auth login            # guided OAuth or token
gh auth status           # verify
gh config set prompt disabled true   # non-interactive scripts
gh alias set prd 'pr create -d -f'   # example alias
```

Multiple accounts:

```bash
gh auth login --hostname github.com --scopes repo,workflow
gh auth switch --user <handle>
```

---

## Daily Ops: Issues and PRs

Issues:

```bash
gh issue list --label bug --state open
gh issue create -t "Crash on startup" -b "Steps…" -l bug -a @me
gh issue view 123 -w     # open in browser
```

Pull requests:

```bash
gh pr create -t "Fix: null check" -b "…" -B main -H feat/guard -r org/team
gh pr checks             # status of CI
gh pr review --approve
gh pr merge --auto --squash
```

Releases and notifications:

```bash
gh release create v1.2.0 dist/* -t "v1.2.0" -n "Changelog…"
gh notif list --unread
```

---

## CI/CD, Secrets, and Raw API

Workflows:

```bash
gh workflow list
gh run list
gh run watch --job 123456789
gh workflow run build.yml -f ref=main
```

Secrets:

```bash
gh secret set NPM_TOKEN --body "$NPM_TOKEN"              # repo
gh secret set PROD_KEY --org <org> --repos 'app-*'       # org with repo filter
```

REST access when features lag:

```bash
gh api repos/{owner}/{repo}/actions/runs --paginate | jq '.workflow_runs[].status'
gh api graphql -f query='query { viewer { login } }'
```

Tip: compose `gh` with `jq`, `xargs`, and `bash` for reliable automation.
