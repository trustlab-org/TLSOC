# TLSOC Ecosystem Standards

This document defines the conventions every TLSOC repository follows. It exists so
that each repository — current and future — feels like part of one product, and so
that new repositories can join the ecosystem without inventing anything.

## Naming convention

- **Component (product) names** are title-case with the TLSOC prefix and are
  used in all prose, titles, and diagrams: *TLSOC Engine*, *TLSOC Docker
  Deploy*, *TLSOC Reporting*.
- **Existing repositories keep their historical names** — renaming is
  deliberately avoided so that clones, links, and integrations never break:

  | Component | Repository |
  |---|---|
  | TLSOC (ecosystem home) | `tlsoc` |
  | TLSOC Engine | `foss-soc-engine` |
  | TLSOC Docker Deploy | `TLSOCDockerDeploy` |
  | TLSOC Reporting | `tlsoc-reporting-framework` |
  | TLSOC ML | `tlsoc-machinelearning-framework` |

- **Future repositories** are named lowercase-with-hyphens, prefixed `tlsoc-`:
  `tlsoc-dashboard`, `tlsoc-parser-plugins`, `tlsoc-examples`,
  `tlsoc-integrations`.
- Repository tables and links always show the real repository name; prose uses
  the component name.

## Standard repository structure

Every repository contains, at minimum:

```
repository/
├── README.md               # Standard layout (see below)
├── LICENSE                 # Apache-2.0
├── CHANGELOG.md            # Keep a Changelog format
├── CONTRIBUTING.md         # How to contribute to this component
├── SECURITY.md             # Private vulnerability reporting
├── CODE_OF_CONDUCT.md      # Contributor Covenant v2.1
├── .gitignore
├── docs/                   # All long-form documentation
│   ├── images/
│   │   └── tlsoc-logo.jpeg # The shared TLSOC logo
│   ├── architecture.md
│   └── roadmap.md
└── .github/
    ├── ISSUE_TEMPLATE/
    │   ├── bug_report.yml
    │   ├── feature_request.yml
    │   └── config.yml
    └── PULL_REQUEST_TEMPLATE.md
```

Implementation code, configuration, and additional docs vary per repository —
the files above do not.

## Standard README layout

Every README uses the same section order (omit a section only when it truly does
not apply):

1. Centered logo, project name, one-line tagline
2. Badge row (ecosystem badge first, then license, then repo-specific badges)
3. Quick navigation links
4. **Overview** — what it is, in a few sentences
5. **Key Features**
6. **Architecture** — a Mermaid diagram plus a link to `docs/architecture.md`
7. **Getting Started** — prerequisites and installation
8. **Usage**
9. **Repository Structure**
10. **Documentation** — a table linking into `docs/`
11. **TLSOC Ecosystem** — the shared related-projects table
12. **Roadmap** — highlights plus a link to `docs/roadmap.md`
13. **Contributing**
14. **Security**
15. **License**
16. **Standard footer** — TrustLab attribution and ecosystem links (exact block
    in [branding.md](branding.md#standard-footer))

The ecosystem home (`tlsoc`) additionally carries platform-level sections
(Project Status, Why TLSOC, Engineering Philosophy, Choose the Right Repository,
Supported Log Sources) that satellite repositories do not duplicate.

Formatting, badge style, positioning language, and the color palette are defined
in [branding.md](branding.md).

## The shared ecosystem table

Every repository's **TLSOC Ecosystem** section carries this table (with its own row
present but not linked):

| Repository | Purpose |
|---|---|
| [tlsoc](https://github.com/sankettaware16/tlsoc) | Ecosystem home — documentation, architecture, roadmap |
| [foss-soc-engine](https://github.com/sankettaware16/foss-soc-engine) | Log parsing and ECS normalization engine |
| [TLSOCDockerDeploy](https://github.com/sankettaware16/TLSOCDockerDeploy) | TLS-secured core stack (Kafka, Logstash, Elasticsearch, Kibana) |
| [tlsoc-reporting-framework](https://github.com/sankettaware16/tlsoc-reporting-framework) | Declarative executive reporting (HTML/PDF) |

When a new repository is published, add its row here and to every existing
repository's table.

## Documentation conventions

- Long-form documentation lives in `docs/`, not in the README. The README carries
  the overview, quick start, and links.
- Documentation file names are lowercase with hyphens: `writing-rules.md`,
  `web-ui-guide.md`.
- Diagrams are [Mermaid](https://mermaid.js.org/) blocks in Markdown — they render
  natively on GitHub and diff cleanly.
- Terminology is consistent across repositories: *the stack* (TLSOC Docker Deploy
  deployment), *the engine* (TLSOC Engine), *rule* (a YAML parser definition),
  *datasource* / *report* (TLSOC Reporting YAML units), *the envelope* (the Kafka
  JSON message contract), *ECS* (Elastic Common Schema).
- Real deployment data — IP addresses, hostnames, organization names, log samples —
  never appears in documentation or commits. Use `example_org`-style placeholders.

## Repository metadata (GitHub settings)

Each repository sets:

- **Description** — one sentence, starting with "TLSOC —" for satellite repos.
- **Topics** — always `tlsoc`, `soc`, `security`, plus repo-specific topics
  (`log-parsing`, `ecs`, `kafka`, `elk`, `docker`, `reporting`, …).
- **Homepage** — the `tlsoc` repository URL (until a dedicated site exists).
- **Default branch** — `main`.
- **Releases** — tagged versions with notes drawn from `CHANGELOG.md`.

## Versioning and releases

- Repositories version independently using [Semantic Versioning](https://semver.org/)
  (`MAJOR.MINOR.PATCH`).
- Every release is a Git tag (`v1.2.0`) with a GitHub Release whose notes come from
  the corresponding `CHANGELOG.md` section.
- `CHANGELOG.md` follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/):
  an `[Unreleased]` section on top, then one section per release.

## Checklist: adding a new repository to the ecosystem

1. Name it `tlsoc-<component>` (lowercase, hyphens).
2. Copy the standard structure above; reuse the community health files from any
   existing repository, adjusting the repo name.
3. Copy `docs/images/tlsoc-logo.jpeg` and follow the README layout and
   [branding](branding.md).
4. Add the repository to the ecosystem table in `tlsoc` **and** in every existing
   repository's **TLSOC Ecosystem** section.
5. Set the GitHub metadata (description, topics, homepage).
6. Add a row to the ecosystem [roadmap](roadmap.md) if it was listed as planned.
