# Agents in CI/CD

---

## Headless agents

Most coding agents can run **non-interactively**: a prompt in, a result out, no human at the keyboard

```bash
<agent-cli> --non-interactive "Fix the failing test in core/, then commit"
```

* Any CI system can run them: they are *just another command*
* Triggered by *events*, as any other pipeline:
    * an issue is labeled, a comment mentions the agent, a PR is opened, a schedule fires...
* Typical outputs: a *pull request*, a *review*, a *comment*, a *label*

<br>

Several vendors provide *hosted* versions:
<br>
assign an issue to the agent $\Rightarrow$ it works in a cloud sandbox $\Rightarrow$ it opens a *draft PR* and asks for review

---

## Example: from issue to pull request

```yaml
name: Agent on labeled issues
on:
  issues:
    types: [labeled]          # labeling requires triage permissions: trusted actors only
jobs:
  agent:
    if: github.event.label.name == 'agent'
    runs-on: ubuntu-latest
    timeout-minutes: 30       # agents may loop: always set a limit
    permissions:
      contents: read          # least privilege: pushing happens with a dedicated token
    steps:
      - uses: actions/checkout@v5
      - run: npm install -g some-agent-cli@2.3.1          # pinned
      - run: >-
          some-agent-cli --non-interactive
          "Solve issue #$ISSUE. Work on a new branch, run ./gradlew check, open a pull request."
        env:
          ISSUE: ${{ github.event.issue.number }}         # a number, never the issue text
          AGENT_API_KEY: ${{ secrets.AGENT_API_KEY }}
          GH_TOKEN: ${{ secrets.BOT_TOKEN }}              # GitHub App token, not GITHUB_TOKEN
```

* Pull requests opened with `GITHUB_TOKEN` *do not trigger workflows*: no CI on the agent's PR!
    * Use a dedicated *GitHub App* (or bot account) token
* The issue *text* is untrusted: never interpolate it in a `run:` command (*script injection*)

---

## What agents are used for in CI

* **Issue $\rightarrow$ PR**: well-specified, small, self-contained tasks
* **Code review**: an additional reviewer commenting on PRs
    * An *additional* reviewer: required human review stays
* **Fixing CI failures**: read the logs of a failed build, propose a fix
* **Triage**: label, deduplicate, ask for missing information on new issues
* **Scheduled maintenance**: update documentation, remove dead code, migrate deprecated APIs
    * Changes that deterministic bots cannot do, but are boring for humans

<br>

What makes a task suitable:

* *Clear* acceptance criteria, ideally **verifiable by CI** (tests!)
* *Small* scope: one PR, reviewable in minutes
* *Low* blast radius if wrong

---

## Hosted coding agents: onboarding

The first time a hosted agent works on a repository, it has *no context*

* Some agents run an **onboarding** procedure: they explore the repository and
  *write* an instruction file (`AGENTS.md` or a vendor-specific one) for their future runs
* The generated file is a *draft*: **review it**, trim it, fix it
    * Generated instruction files tend to be long, generic, and partially wrong
* The agent's environment must be able to *build and test* the project
    * Setup steps (toolchains, services) are configured like a CI job
    * If the agent cannot run the tests, it cannot verify its own work
