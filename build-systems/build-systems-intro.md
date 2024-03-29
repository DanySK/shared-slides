## The build "life cycle"
#### (Not to be confused with the system development life cycle (SDLC))

The process of creating *tested deployable software artifacts*
<br/>
from *source* code

May include, depending on the system specifics:
* *Source code manipulation* and generation
* Source code *quality assurance*
* *Dependency management*
* *Compilation*, linking
* *Binary manipulation*
* *Test execution*
* Test *quality assurance* (e.g., coverage)
* API *documentation*
* *Packaging*
* *Delivery*

---

## Lifecycle styles

* **Custom**: select some phases that the product needs and perform them.
    * *Flexible and configurable*: tailored on each project's needs
    * *Hard to adapt and port*

* **Standard**: run a sequence of pre-defined actions/phases.
    * *Portable and easy to understand*: replicated on every product
    * *Limited configuration options*

---

## Build automation

Automation of the build lifecycle

* In principle, the lifecycle could be executed manually
* In reality *time is precious* and *repetitivy is boring*

$\Rightarrow$ Create software that automates the building of some software!

* All those concerns that hold for sofware creation hold for build systems creation...

---

## Build automation: basics and styles

Different lifecycle types generate different build automation **styles**

**Imperative**: write a script that tells the system what to do to get
from code to artifacts
* *Examples*: make, cmake, Apache Ant
* *Abstraction gap*: verbose, repetitive
* Configuration (*declarative*) and actionable (*imperative*) logics *mixed* together
* Highly *configurable*

**Declarative**: adhere to some convention, customizing some settings
* *Examples*: Apache Maven
* Separation between *what* to do and *how* to do it
  * The build system decides how to do the stuff
* *Configuration limited* by the provided options

---

## Hybrid automators

Create a *declarative infrastructure* upon an *imperative basis*, and
*allow easy access to the underlying machinery*

**DSL**s are helpful in this context: they can "hide" imperativity without ruling it out

Still, many challenges remain open:
* How to reuse the build logic?
    * within a project, and among projects
* How to structure multiple logical and interdependent parts?
