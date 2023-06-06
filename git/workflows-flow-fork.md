# DVCS: Workflows

> with great power comes great responsibility

and also

> power is nothing without control

Elements to consider:
* How *large* is the team?
* How *complex* is the project?
* Do team members work *together* (in spacetime)?
* Do team members *trust* each other?

---

## Trunk-based(-like) development

![Trunk-based development (like)](https://raw.githubusercontent.com/DanySK/shared-slides/de260d8b34eff7c4a409d98a096a5e7f2e79b4e4/git/dvcs-sink.svg)

---

## Trunk-based(-like) development


Single branch, shared truth repository, frequent merges

* *Small* teams, *low-complexity* projects, *colocated* teams, *high* trust
* Typical of small company projects

---

## Git flow (classic)

![Git flow](https://raw.githubusercontent.com/DanySK/shared-slides/518047bf7b6686356cad8d9883351c205b7d4f02/git/dvcs-flow-sink.svg)

---

## Git flow (classic)

Multiple branches, shared truth repository

* *Large* teams, *high-complexity* projects, *preferably colocated* teams, *high* trust
* Typical of large company projects

---

{{% import path="shared-slides/git/git-flow.md" %}}

---

## Forks versus branches

* In Git, separate *development lines* are separate *branches*
* However, everyone has a *copy* of the *same repository*
* Git *hosting services* can *identify copies* of the same project belonging to different users

These copies are called **forks**

* Branches on one fork can be requested to be merged on another fork
  * With **merge request** (also called **pull request**, depending on the host)
* Pull requests enable easier code review
  * Necessary when the developer *does not trust* the contributor
  * But very useful anyway
* Working with pull requests is **not part of git** and *requires host support*
  * GitHub, GitLab, and Bitbucket all support pull requests

---

## Single branch, multiple forks

![Trunk-based development (like)](https://raw.githubusercontent.com/DanySK/shared-slides/de260d8b34eff7c4a409d98a096a5e7f2e79b4e4/git/dvcs-fork.svg)

---

## Single branch, multiple forks

* Single branch, multiple independent repository copies
* *Unknown* team size, *low-complexity* projects, *sparse* teams, *low* trust
* Typical of **small open-source projects**

---

## Git flow over multiple forks

![Trunk-based development (like)](https://raw.githubusercontent.com/DanySK/shared-slides/de260d8b34eff7c4a409d98a096a5e7f2e79b4e4/git/dvcs-flow-fork.svg)

---

## Git flow over multiple forks

* Multiple branches, multiple independent repository copies
* *Unknown* team size, *high-complexity* projects, *sparse* teams, *low* trust
* Typical of **complex open-source projects**
