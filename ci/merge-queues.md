# Merge queues

---

## Protecting the main branch

Once a project has CI, the main branch should only receive *verified* changes

Repository hosting platforms can *enforce* rules on selected branches:

* **No direct pushes**: changes enter only via pull requests
* **Required status checks**: selected CI jobs must succeed before merging
* **Required reviews**: at least $n$ approvals from (selected) humans
* **Linear history**: only fast-forward, rebase, or squash merges
* **Signed commits**, no force pushes, no deletion...

<br>

The platform does the policing, so humans (*and bots*) cannot skip the rules by mistake

---

## Green pull requests, red main branch

Two pull requests, opened from the same commit, both *pass* CI

```mermaid
flowchart RL
  master(master)
  rename(feat/rename)
  usef(feat/new-feature)

  CA(["A: rename f() into g()"]) --> C1([1])
  CB(["B: new code calling f()"]) --> C1

  master -.-> C1
  rename -.-> CA
  usef -.-> CB

  okA{{"CI on 1+A ✅"}} -.- CA
  okB{{"CI on 1+B ✅"}} -.- CB

  class master,rename,usef branch;
  class C1,CA,CB commit;
  classDef ok fill:#c8f7c5,stroke:#2e7d32,stroke-width:3px;
  class okA,okB ok;
```

There is *no textual conflict*: they touch different files

---

## Green pull requests, red main branch

Merge both, and the main branch **no longer compiles**: `f()` does not exist anymore

```mermaid
flowchart RL
  master(master)

  C1([1])
  CA(["A: rename f() into g()"]) --> C1
  CB(["B: new code calling f()"]) --> C1
  M1(["merge A"]) --> C1
  M1 --> CA
  M2(["merge B"]) --> M1
  M2 --> CB

  master -.-> M2
  ko{{"1+A+B: never tested ❌"}} -.- M2

  class master branch;
  class C1,CA,CB,M1 commit;
  classDef ko fill:#ffcdd2,stroke:#c62828,stroke-width:3px;
  class M2,ko ko;
```

* CI validated $1+A$ and $1+B$, but what got into `master` is $1+A+B$
* **Semantic conflict**: changes that are fine in isolation, but broken together
* The more people (and bots, and agents) contribute, the more likely this gets

---

## The "not rocket science" rule

> Automatically maintain a repository of code that always passes all the tests
>
> --- Graydon Hoare (creator of Rust), describing the `bors` bot, 2014
> ([source](https://web.archive.org/web/20251220014900/https://graydon.livejournal.com/186550.html))

<br>

Reformulated:

**the main branch can only move to a commit that has been tested *exactly as it will be***

<br>

A pull request being green is *not enough*:
<br>
it must be green *on top of the current main branch*

---

## Naive solution: require up-to-date branches

Before merging, the branch must contain the *latest* main branch:

1. Update (merge/rebase) the PR branch on top of `master`
2. Re-run CI
3. Merge, *if nobody merged in the meanwhile*, otherwise go back to 1.

<br>

It works, but it **does not scale**:

* Pull requests are integrated *one at a time*
* With $n$ ready PRs and a CI taking $t$, the last one waits *at least* $n \cdot t$
* Every merge *invalidates* all other PRs, which must update and re-run CI again
* Humans end up *racing* to merge first, and babysitting PRs

---

## Merge queue: the idea

Ready PRs are not merged: they are **enqueued**

The queue *builds the future main branch*, tests it, and only then moves `master`

```mermaid
flowchart RL
  master(master)
  pr(feat/A)
  q(queue/A)

  C1([1])
  CA(["A"]) --> C1
  QA(["1+A"]) --> C1
  QA --> CA

  master -.-> C1
  pr -.-> CA
  q -.-> QA

  test{{"CI 🔄"}} -.- QA

  class master,pr branch;
  class C1,CA commit;
  classDef queue fill:#ffe0b2,stroke:#ef6c00,stroke-width:3px;
  classDef testing fill:#fff9c4,stroke:#f9a825,stroke-width:3px;
  class q,QA queue;
  class test testing;
```

1. A *temporary branch* is created with the PR merged on top of the current `master`
2. CI runs on the *temporary branch*
3. If CI passes, `master` is moved to the *tested* commit, otherwise the PR is *ejected*

---

## Speculative checks

Testing one PR at a time is *safe* but *slow*

**Bet** on success: test each PR *on top of all the PRs ahead of it in the queue*, *in parallel*

```mermaid
flowchart RL
  master(master)
  q1(queue/A)
  q2(queue/B)
  q3(queue/C)

  C1([1])
  QA(["1+A"]) --> C1
  QB(["1+A+B"]) --> QA
  QC(["1+A+B+C"]) --> QB

  master -.-> C1
  q1 -.-> QA
  q2 -.-> QB
  q3 -.-> QC

  t1{{"CI 🔄"}} -.- QA
  t2{{"CI 🔄"}} -.- QB
  t3{{"CI 🔄"}} -.- QC

  class master branch;
  class C1 commit;
  classDef queue fill:#ffe0b2,stroke:#ef6c00,stroke-width:3px;
  classDef testing fill:#fff9c4,stroke:#f9a825,stroke-width:3px;
  class q1,q2,q3,QA,QB,QC queue;
  class t1,t2,t3 testing;
```

* If everything passes, `master` fast-forwards to $1+A+B+C$: three PRs in the time of *one* CI run
* The price: *more CI runs in parallel* (and wasted ones if the bet fails)

---

## Speculative checks: when the bet fails

$B$ breaks the build: $B$ is ejected, and **every speculation built on top of it is discarded**

```mermaid
flowchart RL
  master(master)
  q3(queue/C)

  C1([1])
  QA(["1+A"]) --> C1
  QB(["1+A+B"]) --> QA
  QC(["1+A+B+C"]) --> QB
  QC2(["1+A+C"]) --> QA

  master -.-> QA
  q3 -.-> QC2

  t3{{"CI 🔄"}} -.- QC2

  class master branch;
  class C1,QA commit;
  classDef ko fill:#ffcdd2,stroke:#c62828,stroke-width:3px;
  classDef discarded fill:#eeeeee,stroke:#9e9e9e,stroke-width:2px,stroke-dasharray:5 5;
  classDef queue fill:#ffe0b2,stroke:#ef6c00,stroke-width:3px;
  classDef testing fill:#fff9c4,stroke:#f9a825,stroke-width:3px;
  class QB ko;
  class QC discarded;
  class q3,QC2 queue;
  class t3 testing;
```

* $A$ passed, so `master` advances to $1+A$
* $C$ is *re-tested* on top of the new hypothetical future: $1+A+C$
* The author of $B$ is notified, fixes the problem, and re-enqueues

---

## Batching

Alternative (or complement) to speculation: test **many PRs together, with a single CI run**

If the batch passes, all PRs are merged at once.
<br>
If it fails, *who is guilty*? **Split** the batch and test the halves (*bisection*, as in `git bisect`)

```mermaid
flowchart TB
  B0["A+B+C+D ❌"] --> B1["A+B ✅"]
  B0 --> B2["C+D ❌"]
  B2 --> B3["C ✅"]
  B2 --> B4["D ❌ → ejected"]

  classDef ok fill:#c8f7c5,stroke:#2e7d32,stroke-width:3px;
  classDef ko fill:#ffcdd2,stroke:#c62828,stroke-width:3px;
  class B1,B3 ok;
  class B0,B2,B4 ko;
```

* *Cheap* when failures are rare: one CI run for many PRs
* *Expensive* when failures are frequent: $\log_2(n)$ extra rounds to find the culprit
* Works well with *many small, low-risk* PRs (e.g., dependency updates)

---

## Priorities

Not all changes are equally urgent:

* A *security fix* or a *hotfix* for a broken release should not wait behind 20 dependency updates
* *Dependency updates* (from bots) can wait

```mermaid
flowchart LR
  new["🔥 hotfix (high)"]
  subgraph queue["merge queue"]
    direction LR
    F1["feature X (medium)"] ~~~ F2["feature Y (medium)"] ~~~ R1["bump lib-a (low)"] ~~~ R2["bump lib-b (low)"]
  end
  new == "jumps ahead" ==> F1
  F1 -- "next to merge" --> master(master)

  classDef high fill:#ffcdd2,stroke:#c62828,stroke-width:3px;
  classDef medium fill:#fff9c4,stroke:#f9a825,stroke-width:3px;
  classDef low fill:#e3f2fd,stroke:#1565c0,stroke-width:3px;
  class new high;
  class F1,F2 medium;
  class R1,R2 low;
  class master branch;
```

* **Priority**: higher-priority PRs are placed *ahead* in the queue
* **Preemption** (optional): running checks of lower-priority PRs are *interrupted*, wasting CI time
* **Multiple queues**: different rules (batch size, checks) for different kinds of changes

---

## Merge queues: costs and pitfalls

* **CI must run on the queue's temporary branches**, not only on PRs
    * If a required check never runs there, the queue *stalls forever*
* **More CI time**: every PR is tested at least twice (as PR, and in the queue)
    * Option: run *fast* checks on PRs, and the *expensive* ones only in the queue
* **Flaky tests poison the queue**: a random failure ejects an innocent PR,
  <br>and invalidates all speculations behind it
    * Options: retry failed checks, *fix flaky tests* (better)
* **Long pipelines** make the queue slow: speculation and batching mitigate, but do not cure
* **One more moving part**: when something does not merge, someone has to understand *why*

---

## Merge queue implementations

| Tool | Notes |
|------|-------|
| `bors` | The original (Rust project, 2014), now retired |
| Zuul | OpenStack's CI gatekeeper, pioneered speculative execution |
| GitLab *merge trains* | Built into GitLab |
| GitHub merge queue | Built into GitHub (org-owned repositories), CI must listen to the `merge_group` event |
| **Mergify** | GitHub App, *configuration as code*, priorities, multiple queues |
| Aviator, Trunk, Graphite | Commercial services, often monorepo-oriented |

<br>

Same concepts everywhere: **queue**, **temporary branch**, **speculation**, **batching**, **priorities**
