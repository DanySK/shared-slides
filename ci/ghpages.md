## Documenting repositories

Documentation of a project is **part of the project**

* **Documentation** must stay in the same repository of the project
* However, it should be *accessible to non-developers*

### Meet GitHub Pages

* GitHub provides an *automated* way to publish *webpages from **Markdown*** text
* Markdown is a *human readable markup language*, [easy to learn](https://learnxinyminutes.com/docs/markdown/)
  * These slides are written in Markdown
  * (generation is a bit richer, but just to make the point)
* Supports *Jekyll* (a Ruby framework for static website generation) out of the box
  * We will not discuss it today
  * But maybe in LSS...

---

## Setting up a GitHub Pages website

Two possibilities:
1. Select a branch to host the pages
    * Create an orphan branch with `git checkout --orphan <branchname>`
    * Write your website, one Markdown per page
    * Push the new branch
1. Use a `docs/` folder in a root of some branch
    * Could be `master` or any other branch

---

## Setting up a GitHub Pages website

Once done, enable GitHub pages on the repository settings:

![Enable GitHub Pages snapshot](https://raw.githubusercontent.com/DanySK/shared-slides/518047bf7b6686356cad8d9883351c205b7d4f02/ci/ghpages-enable.png)

---

## GitHub Pages URLs

* Repository web-pages are available at `https://<username>.github.io/<reponame>`
* User web-pages are available at `https://<username>.github.io/`
  * They are generated from a repository named `<username>.github.io`
* Organization web-pages are available at `https://<organization>.github.io/`
  * They are generated from a repository named `<organization>.github.io`
