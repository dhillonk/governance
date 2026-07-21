# RFC: Documentation Philosophy for ODP Projects

This RFC proposes that Open Device Partnership (ODP) standardize on **co-located
project documentation**: each project owns its documentation inside the same
repository as its source code, published as an [mdBook][mdbook] site via
GitHub Pages. The existing `documentation` repository continues to exist, but
its role narrows to hosting **organization-level** content (charter,
governance, cross-project standards, the RFC index) and to acting as a landing
page that links out to each project's mdBook.

## Change Log

- 2026-07-21: Initial RFC draft created.

## Motivation

ODP projects today use two inconsistent approaches to documentation:

1. **Centralized.** Projects such as Secure EC and Standardized EC Services
   keep their documentation in the shared `documentation` repository,
   separate from the code it describes.
2. **Co-located.** Patina keeps its documentation inside the Patina
   repository, next to the code, published as an mdBook.

Having two models causes real friction:

- **Drift between code and docs.** When docs live in a different repository
  than the code, changes to behavior, APIs, or configuration frequently ship
  without matching documentation updates, because a code PR cannot atomically
  update centralized docs.
- **Discoverability.** Contributors browsing a project repository do not
  reliably find its documentation; they have to know that a separate
  documentation repository exists and how it is organized.
- **Ownership ambiguity.** Centralized docs are edited by many people across
  many projects, which weakens per-project ownership and review signal
  (CODEOWNERS, project maintainers, subsystem owners).
- **Cross-repo coordination cost.** Reviewers, release processes, and
  automation have to reason about two repositories to land a single logical
  change.
- **Inconsistent contributor experience.** New contributors have to learn a
  different workflow depending on which ODP project they are working on.

Patina has already validated the co-located mdBook approach in production
within ODP. Standardizing on it removes the split, aligns ODP with common
Rust ecosystem practice, and lets each project treat its documentation as a
first-class part of its codebase.

## Technology Background

- **mdBook** ([`rust-lang/mdBook`][mdbook]) is the documentation tool used by
  the Rust project and much of the Rust ecosystem. It builds a static book
  site from a tree of Markdown files and a `SUMMARY.md` table of contents.
- **GitHub Pages** publishes static sites directly from a GitHub repository,
  typically via a GitHub Actions workflow that builds the mdBook on push and
  deploys the rendered output.
- **Current state — `documentation` repository.** The `documentation`
  repository hosts a mix of organization-level material (charter, governance,
  process) and project-specific material for Secure EC and Standardized EC
  Services.
- **Current state — Patina.** Patina's documentation lives inside the Patina
  repository and is published as an mdBook site, updated by the same PRs
  that change the code.
- **Current state — governance.** This `governance` repository already
  hosts organization-level content (charter, steering committee, RFCs) and
  is a natural home for the org-level material the `documentation`
  repository retains after this change.

## Goals

1. Establish a single, consistent documentation model across all ODP
   projects.
2. Ensure that project documentation is co-located with the code it
   describes, so it can be updated atomically with code changes.
3. Standardize the documentation toolchain on **mdBook + GitHub Pages** so
   contributors, reviewers, and readers have a consistent experience across
   projects.
4. Preserve a clear home for **organization-level** documentation (charter,
   governance, cross-project standards, the RFC index) that is not
   project-specific.
5. Migrate existing centralized project documentation (Secure EC,
   Standardized EC Services) into the corresponding project repositories
   without loss of content or history where practical.
6. Keep the migration cost low and incremental, so projects are not blocked
   on a big-bang cutover.

## Requirements

1. **Project docs live in the project repo.** Every ODP project repository
   owns its own documentation under a conventional in-repo path
   (default: `docs/`).
2. **mdBook is the required tool.** Project documentation is authored as an
   mdBook (Markdown + `SUMMARY.md`, built with `mdbook`).
3. **GitHub Pages publishing.** Each project publishes its rendered mdBook
   via GitHub Pages from the project repository, using a GitHub Actions
   workflow that builds on push to the project's default branch.
4. **Code and docs ship together.** Changes to behavior, APIs,
   configuration, or user-visible workflows must update the corresponding
   in-repo documentation in the same PR. CODEOWNERS for the `docs/` path
   should reflect the project's maintainers/subsystem owners.
5. **The `documentation` repository becomes org-level only.** After
   migration, the `documentation` repository hosts only organization-level
   content: charter, governance, cross-project standards and conventions,
   the RFC index, and a landing page that links out to each project's
   published mdBook. It does not host project-specific technical
   documentation.
6. **Existing centralized docs are migrated.** Documentation currently held
   in the `documentation` repository for Secure EC and Standardized EC
   Services is moved into those projects' repositories as mdBooks. The
   original locations in the `documentation` repository are then replaced
   with short link stubs pointing at the new canonical location.
7. **New projects follow this model on day one.** Any new project onboarded
   under ODP governance stands up its documentation in-repo as an mdBook
   with a GitHub Pages workflow as part of initial setup.
8. **Stable external URLs.** During migration, projects should preserve
   inbound links where reasonable by leaving redirect/link stubs at the
   previous locations in the `documentation` repository.

## Migration Plan

The migration is performed **per project** and is expected to run
incrementally. For each project currently documented in the `documentation`
repository (initially: Secure EC and Standardized EC Services), the owning
maintainers:

1. **Stand up mdBook in the project repository.**
   - Add a `docs/` directory containing `book.toml`, `src/SUMMARY.md`, and
     an initial `src/` tree.
   - Add a GitHub Actions workflow that builds the mdBook and deploys it to
     GitHub Pages on push to the default branch.
   - Enable GitHub Pages on the project repository.
2. **Move content.** Copy the project's documentation from the
   `documentation` repository into `docs/src/` in the project repository,
   preserving structure where it still makes sense. Update internal links
   to be relative to the new location.
3. **Publish.** Land the new mdBook, verify the published GitHub Pages
   site, and confirm the `SUMMARY.md` covers the migrated content.
4. **Cut over the `documentation` repository.** Replace the migrated pages
   in the `documentation` repository with short stubs that link to the new
   canonical location on the project's GitHub Pages site, so existing
   inbound links continue to work.
5. **Announce.** Note the cutover in the project's usual communication
   channels so downstream readers and contributors update their bookmarks.

Once all previously centralized project docs have been migrated, the
`documentation` repository's top-level index/landing page is updated to
enumerate ODP projects and link to each project's published mdBook.

## Unresolved Questions

- **Migration timeline.** This RFC does not fix a completion date. Should
  a target window (e.g. "by the next release cycle") be set for the
  initial two projects, or is best-effort acceptable?

## Prior Art

- **Patina** already co-locates its documentation with its source code and
  publishes it as an mdBook. This RFC generalizes Patina's approach to the
  rest of ODP.
- **Rust ecosystem practice.** Many core Rust projects publish their
  documentation as mdBook sites hosted alongside the code, including
  [The Rust Programming Language book][rust-book],
  [the Cargo book][cargo-book], and
  [the `rustc` dev guide][rustc-dev-guide]. Contributors moving between
  ODP projects and the broader Rust ecosystem will find a familiar model.
- **Docs-as-code more broadly.** Treating documentation as source that
  lives with the code, is reviewed in the same PRs, and is built by CI is
  a well-established pattern across open-source projects.

## Alternatives

- **Keep the status quo (mixed model).** Continue to allow both
  centralized and co-located documentation on a per-project basis.
  Rejected because it perpetuates the drift, discoverability, and
  ownership problems described in Motivation, and because contributors
  and reviewers continue to face inconsistent workflows across projects.
- **Centralize everything in the `documentation` repository.** Move
  Patina's documentation out of the Patina repository and require all
  projects to document themselves in `documentation`. Rejected because it
  worsens the drift problem, prevents atomic code+docs PRs, weakens
  per-project ownership, and reverses a model that is already working
  well for Patina.
- **Co-locate, but leave tooling open.** Require co-location without
  mandating mdBook or GitHub Pages. Rejected as a primary recommendation
  because it does not give contributors, reviewers, or readers a
  consistent experience across projects, and because mdBook + GitHub Pages
  is already the working approach in Patina and is well-supported in the
  Rust ecosystem. A future RFC can revisit the toolchain if a compelling
  reason emerges.
- **Publish from the `documentation` repository, author in project
  repositories.** Have project repositories author docs but centralize
  publishing. Rejected as unnecessarily complex; GitHub Pages publishing
  from the project repository is straightforward and keeps ownership in
  one place.

[mdbook]: https://github.com/rust-lang/mdBook
[rust-book]: https://doc.rust-lang.org/book/
[cargo-book]: https://doc.rust-lang.org/cargo/
[rustc-dev-guide]: https://rustc-dev-guide.rust-lang.org/
