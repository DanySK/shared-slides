## Dependencies in software

> nos esse quasi nanos gigantum humeris insidentes
<cite>Bernard of Chartres</cite>

All modern software depends on other software!
* the operating system
* the runtime environment (e.g., the Java Virtual Machine, the Python interpreter...)
* the base libraries
* third-party libraries
* external resources (icons, sounds, application data)

All the software we build and use depends on other software
* Which depends on other software
  * Which depends on other software
    * Which depends on other software
      * Which depends on other software
        * Which depends on other software
          * Which depends on other software
            * Which depends on other software
              * ...

$\Rightarrow$ Applications have a **dependency** tree!

---

## Transitive dependencies

Indirect dependencies (dependencies of dependencies) are called *transitive*

In non-toy projects, transitive dependencies are the *majority*
* It's very easy to end up with more than 50 dependencies

#### Complexity quickly gets out of hand!

We need a tool that can:
* *Find* (for example in known archives) the libraries we need
* *Download* them (if found)
* Include them into a discoverable path

To do this, however, we need to know some repositories and how to refer to and locate artifacts
* We need a *name* and a *version* for each library
* There is no standard, **every ecosystem has its own conventions**
    * Some created with the programming language (e.g., Rust)
    * Some evolved later with the ecosystem (Maven for Java, npm for JavaScript...)

---

## Transitive dependency conflicts

When two libraries depend on different versions of the same library, we have a *conflict*
* e.g.: dependency `A` requires `B` at version `1`, dependency `C` requires `B` at version `2`
* How would you solve this conflict?
    * Pick the latest and hope for the best?
    * Pick the earliest and hope for the best?

Different tools, different defaults:
* **Maven**: *nearest wins* -- the version declared closest to the root of the dependency tree is selected
* **Gradle**: *highest wins* -- the highest requested version is selected (optimistic upgrade)
* **npm**: *no single version needed* -- conflicting versions can coexist in nested `node_modules`
* **pip**/**uv**/**Poetry** (Python): *one version per environment* -- the resolver must find a version satisfying all constraints, or fail

---

## Dependency ranges

To reduce the risk of conflicts, some systems allow specifying *ranges* of acceptable versions
* A range is a *relaxed constraint* on the version specification

### Examples

* **Maven** (Java)
  * `[1.2,2.0)` means "from `1.2` (inclusive) to `2.0` (exclusive)"
  * `[1.2,1.3)` means "from `1.2` (inclusive) to `1.3` (exclusive)"
  * `[1.2,1.3]` means "from `1.2` (inclusive) to `1.3` (inclusive)"
  * `(,1.3]` means "up to `1.3` (inclusive)"
  * `[1.2,)` means "from `1.2` (inclusive) onwards"
* **Gradle** (Java, Kotlin, multiple languages)
  * Same syntax as Maven, plus *prefix* (dynamic) versions: `1.2.+` means "any `1.2.x` version", i.e., `>=1.2.0 <1.3.0`
  * **Rich versions** further refine the constraint: `strictly`, `require`, `prefer`, `reject`
* **npm** / **pnpm** / **yarn** (JavaScript/TypeScript)
  * `^1.2.3` means "compatible with `1.2.3`", i.e., `>=1.2.3 <2.0.0`
  * `~1.2.3` means "approximately `1.2.3`", i.e., `>=1.2.3 <1.3.0`
  * `1.2.x` means "any `1.2` version", i.e., `>=1.2.0 <1.3.0`
* **Python** (PEP 440)
  * `>=1.2, <2.0` means "from `1.2` (inclusive) to `2.0` (exclusive)"
  * `>=1.2, <1.3` means "from `1.2` (inclusive) to `1.3` (exclusive)"
  * `>=1.2, <=1.3` means "from `1.2` (inclusive) to `1.3` (inclusive)"
  * `<1.3` means "up to `1.3` (exclusive)"
  * `>=1.2` means "from `1.2` (inclusive) onwards"
  * `~=1.2.3` means "compatible release", i.e., `>=1.2.3, <1.3.0`
  * `==1.2.*` means "any `1.2` version", i.e., `>=1.2.0, <1.3.0`

---

## Dependency ranges resolution

When multiple ranges are specified for the same dependency, the system must find a version that satisfies all constraints.

The resolver typically relies on a **dependency graph**:
* Input: *roots* (direct dependencies) with version ranges, and *recursive dependencies* with version ranges
* Build a *graph*: nodes are dependencies, edges are "depends on"
* Traverse the graph, starting from the roots (or the most constrained nodes), and *reduce the ranges* removing incompatible versions
* If a node ends up with an *empty range*, there is a **conflict that cannot be resolved**
* Otherwise, all nodes have non-empty ranges, *pick a version for each node* (typically the latest in the range)

---

## Dependency locking

Introducing ranges leads to the **explosion of the version combinations** space.
* Testing all combinations is *impossible*!
* Different tests may resolve to different versions, leading to *inconsistent results*

**Dependency locking** is a technique to fix the versions of all dependencies (including transitive ones)
to a known good set.
* The *specified versions* are typically *ranges*, in the main project manifest
* The *locked versions* are stored in a *dedicated file*

#### There is a trade-off between flexibility and reproducibility!

**Exact pins** trade *low maintenance risk* for *high ongoing cost* and *poor ecosystem fit*:
* Libraries become uncomposable. If two libs pin different exact transitives, resolvers can’t find a common graph
* Pins don’t freeze transitives anyway. Without a lock you still get drift downstream

|                 | Ranges, no lock            | Ranges + lock                                  | Fixed versions (exact pins)                   |
|-----------------|----------------------------|------------------------------------------------|-----------------------------------------------|
| Updates         | *Instant*                  | Manual (regenerate lock file)                  | Manual (change version)                       |
| Reproducibility | Low                        | *Maximum* (transitive dependencies are locked) | High (transitive drift remains)               |
| Reliability     | Low (uncontrolled updates) | Medium (testing all ranges is impossible)      | *High* (controlled updates, transitive drift) |

---

## Dependency scopes

Dependencies may be needed in different contexts:
* **Compile-time**: needed to compile the code (any library whose types are referenced in the source code)
* **Runtime**: needed to run the code
    * Note: it is common for runtime dependencies to be a *superset* of compile-time dependencies
    (some dependencies are runtime-only, e.g., JDBC drivers or logging backends, loaded via reflection or `ServiceLoader`)
    * Note: even though most of the time compile-time dependencies are also runtime dependencies,
    this is not always the case (e.g., annotation-only libraries, some components of ANTLR4...)
* **Test**: needed to compile and run tests (e.g., testing and mocking frameworks)
* **Test runtime**: needed to run tests, but not to compile them (e.g., test engines such as the JUnit Platform launcher)
    * Note: similar to the runtime scope, test runtime dependencies are often a *superset* of test dependencies
* **Build time**: needed to build the code (e.g., code generators, linters, documentation tools)
* ...

Orthogonally, libraries distinguish **API** from **implementation** dependencies:
* **API**: types appear in the *public signatures* of the library, so consumers need them at *compile time* too
* **Implementation**: used only *internally*, consumers need them only at *runtime*
    * e.g., Gradle's `api` and `implementation` configurations

Scopes are typically _pre-defined in **declarative** build systems_ and _manually defined in **imperative** ones_
(there are exceptions to this rule).

*__Hybrid__ automators* often provide _pre-defined scopes_ and _ways to define additional custom scopes_.