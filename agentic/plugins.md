# Plugins: packaging agent configuration

---

## Don't Repeat Yourself, again

The same skills, MCP servers, and hooks are useful in *many* repositories

* Copy-pasting them means *divergence* and *no updates*
* We already solved this problem for builds and CI:

| Domain | Reuse unit | Distribution |
|---|---|---|
| Build | Gradle plugin | Gradle Plugin Portal, Maven Central |
| CI | Composite action, reusable workflow | GitHub repositories, Marketplace |
| Mergify | Shared configuration (`extends:`) | GitHub repositories |
| **Agents** | **Plugin** | **Marketplaces** (git repositories, registries) |

---

## Agent plugins

A **plugin** is a *directory with a manifest* bundling agent extensions

```text
my-team-plugin/
├── <manifest>          # name, version, description
├── skills/
│   ├── dependency-migration/SKILL.md
│   └── code-review/SKILL.md
├── mcp.json            # MCP servers to start
└── <vendor-specific>/  # hooks, commands, subagents... (not portable)
```

* **Portable part** ([Agent Plugins 1.0](https://agent-plugins.org), 2026): *skills* and *MCP servers*
    * Supported by several agents and IDEs
* Hooks, slash commands, subagents: supported by many agents, but in **vendor-specific** formats
* A **marketplace** is a catalog of plugins, typically a git repository with an index file
    * Users *add* a marketplace, then *install* plugins from it

---

## Plugins are dependencies

Installing a plugin means running *someone else's* code and instructions, *with your permissions*

Everything we said about dependencies applies:

* **Versioning**: plugins should follow *semantic versioning*
* **Pinning**: pin to a version (or a commit hash), update deliberately
    * Ideally, with a bot (like Renovate) and a pull request
* **Provenance**: prefer trusted sources; *read* what you install
* **Licensing**: skills and plugins are software, and need a license

<br>

A skill is *Markdown*, but it is also **code**: it tells an agent with shell access what to execute
