# Eneo Marketplace Hub

This repository is the future home of the central Eneo Marketplace Hub: a
service operated by Eneo/Sundsvall that lets approved municipalities and Eneo
instances discover and exchange governed Eneo package releases.

> **Status: planning only.** This repository does not yet contain a Marketplace
> service, API, deployment, database schema, or supported package registry. The
> implementation gate and required decisions are tracked in the linked Epics.

## TL;DR

We want municipal staff to open Eneo and find a shared library of solutions that
other municipalities have chosen to share. If Sundsvall builds a useful HR
Assistant or permit-handling Flow, another municipality should be able to reuse
that work without rebuilding it from the beginning.

The publisher gives each Flow, Skill, Assistant, or future App a clear
description, version, and audience. A receiving municipality can inspect the
complete contents inside Eneo, see which local choices or mappings it must make,
and install a new local copy after its tenant administrator approves the plan.
That copy belongs to the receiving municipality and keeps working during a
Marketplace outage.

Skills are the first building block. A large Assistant prompt often mixes many
jobs, such as HR guidance, IT support, management support, and document creation.
Skills let authors keep a clear base role and move each specialist job into a
named instruction set. Authors can reuse the same Skill across suitable
Assistants and create new revisions without changing parents pinned to an older
revision. An Assistant can later travel with its exact Skills as one complete
package.

The design has two owners:

1. **The central Marketplace Hub in this repository** manages approved
   contributors and instances, review, versions, audiences, publication,
   downloads, and audit.
2. **The Marketplace experience inside Eneo** lets a local tenant administrator
   connect to a Hub URL, browse, preview, resolve local requirements, and approve
   installation under that Eneo instance's own permissions.

The split protects local authority. The Hub cannot administer a municipality's
Eneo installation, and Eneo shares no human OIDC identity with the Hub. A
download gives the local administrator a package to review; it grants the Hub no
remote control.

Municipal staff spend less time duplicating work. They can share solutions that
have worked elsewhere, understand Assistants as smaller named capabilities,
choose when to adopt a new version, and move from manual package files to a
catalog with review, access control, and history.

## Intended outcome

The Marketplace should make it straightforward to publish, review, version,
find, preview, download, and install reusable Eneo resources across independent
Eneo installations. The long-term catalog may include:

- Flows;
- instruction-only Skills;
- Assistants with exact nested Skill revisions;
- Apps, after their own portability contract exists; and
- knowledge-bearing packages, only after a separate security, scanning,
  ingestion, retention, and cleanup gate is accepted.

Every published release should have a stable coordinate, semantic version,
description, compatibility metadata, immutable package bytes, exact digest,
publisher, audience, lifecycle state, and retained audit history.

## Repository boundary

The central Hub belongs in this separate `eneo-ai/eneo-marketplace` repository.
It has different identity, moderation, storage, operational, and release
responsibilities from an Eneo installation.

The native Eneo integration belongs in
[`eneo-ai/eneo`](https://github.com/eneo-ai/eneo): connection administration,
the Marketplace catalog interface, local preview and conflict planning,
tenant-admin confirmation, kind-owned installation, local receipts, and
installed-resource lifecycle.

The Hub distributes authenticated metadata and digest-pinned package bytes. It
must not call into an Eneo installation, hold an Eneo authoring credential, or
create, update, publish, disable, or delete a local Eneo resource. Installed
resources remain local and continue to work when the Hub is unavailable.

## Identity and authentication

Hub identity and Eneo human identity stay separate.

- Hub contributors, reviewers, publishers, and operators use Hub-owned human
  authentication, provisioning, roles, and audit.
- An Eneo installation authenticates to the Hub as a registered machine with a
  server-held credential and short-lived tokens limited to approved scopes.
- Eneo does not send its human OIDC session, user token, subject, role, group,
  or email identity to the Hub.
- The local Eneo tenant administrator is the authority that configures a Hub
  connection, reviews an exact installation plan, and confirms local mutation.
- Machine credentials never enter the browser, package archive, application
  logs, or installed resource.

The final instance-authentication mechanism, `private_key_jwt`, mTLS, or an
approved combination, remains a Marketplace Gate-0 decision together
with token lifetime, rotation, revocation, and incident ownership.

## Governance model

The Hub requires an enterprise permission model with separation of duties. The
design must distinguish at least:

- instance enrollment and credential administration;
- contribution and draft-release management;
- security/contract validation;
- moderation and approval;
- immutable publication;
- yanking and revocation;
- audience and municipality access administration; and
- Hub operations, audit, backup, and recovery.

A contributor cannot approve their own release or mutate published bytes.
Publication, yanking, revocation, rejected submissions, downloads, credential
changes, and administrative actions require durable Hub audit evidence. Eneo
checks local authorization under its own roles; a Hub role never replaces that
check.

## Package and release principles

- Packages use versioned, strict, kind-owned contracts with closed validation.
- Published release bytes are immutable. Corrections produce a new version;
  unsafe releases are yanked or revoked without rewriting history.
- The Hub verifies archive shape, declared size, exact digest, supported profile,
  and compatibility before a release becomes reviewable or installable.
- Eneo downloads bounded content from an approved origin, verifies the exact
  bytes again, produces a deterministic local plan, and rechecks authorization
  and release state before mutation.
- Initial installation creates new draft/inactive local resources. There is no
  silent overwrite, merge, auto-update, auto-publish, or remote rollback.
- Product-specific parsing, planning, installation, and concrete receipts remain
  in the owning Eneo vertical. The Hub is not a remote generic installer.
- No executable Skill, script, plugin, credential, MCP configuration, automatic
  tool grant, or remote code interpretation is accepted by the initial profiles.

## Intended Eneo journey

1. A tenant administrator enrolls the Eneo installation and stores the machine
   credential in server-side configuration.
2. Eneo fetches an authorized, bounded catalog from the configured Hub URL.
3. The administrator selects an immutable release and reviews its publisher,
   version, description, compatibility, digest, complete instructions,
   requirements, nested resources, mappings, and conflicts.
4. Eneo downloads and validates the exact package, then creates a deterministic
   local installation plan without mutating product resources.
5. The administrator confirms the unchanged plan. Eneo rechecks local authority,
   target permissions, release state, compatibility, size, digest, and plan
   identity.
6. The kind-owned installer creates new local draft/inactive resources and a
   concrete provenance receipt in one transaction.

Failure leaves no partial product graph. Retrying the same confirmed
operation must be idempotent or return an explicit existing receipt.

## Delivery gates

Marketplace implementation starts only after the Skills delivery sequence in
[`eneo-ai/eneo` Skills Epic #545](https://github.com/eneo-ai/eneo/issues/545)
completes:

1. S1 first-class local Skills merges into `develop`. The current draft is
   [PR #552](https://github.com/eneo-ai/eneo/pull/552).
2. S2 standalone Skill and Assistant-with-Skills portability starts from the
   updated `develop`, merges, and publishes clean-install conformance fixtures.
3. The execution-order gate is recorded complete in the Skills Epic.
4. Marketplace Gate-0 decisions are accepted before implementation tasks are
   created.

The first Marketplace end-to-end pilot remains the existing strict Flow
package vertical. Assistant-with-Skills enablement follows only after the Hub
pins the accepted S2 contract bundle and digest. App and knowledge-bearing
profiles remain later verticals with separate gates.

## Planning sources

- [Marketplace Epic #546](https://github.com/eneo-ai/eneo/issues/546) owns the Hub
  outcome, blockers, Gate-0 decisions, acceptance criteria, and future tasks.
- [Skills Epic #545](https://github.com/eneo-ai/eneo/issues/545) owns local Skills
  and portable Skill/Assistant package delivery.
- [Marketplace Hub, package portability, and Skills blueprint](https://github.com/eneo-ai/eneo/blob/feature/skills-first-class-local/docs/adr/marketplace-hub-package-portability-and-skills.md)
  owns the detailed architecture, invariants, implementation slices, security
  gates, tests, risk, and rollback plan while the S1 draft is under review.

Project planning belongs in the canonical Eneo organisation Project 5. This
repository does not create a separate Marketplace project or duplicate the
package contracts owned by `eneo-ai/eneo`.

## License

This repository is licensed under the existing [GNU Affero General Public
License v3.0](LICENSE). This planning change does not alter that license.
