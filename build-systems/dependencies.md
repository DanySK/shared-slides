## Dependencies in software

> nos esse quasi nanos gigantium humeris insidentes
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

To do this, however, we need to know some repositories and how to refer and locate artifacts
* We need a *name* and a *version* for each library
* There is no standard, **every ecosystem has its own conventions**
    * Some created with the programming language (e.g., Rust)
    * Some evolved later with the ecosystem (Maven for Java, npm for JavaScript...)
