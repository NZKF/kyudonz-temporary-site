## Why

Today every content change to kyudo.nz — a club's beginner-course dates, a venue, a contact person — routes through one person (Prae) editing hand-written HTML and pushing to git. Club committees can't update their own pages, so information like Auckland's beginner dates goes stale while they wait on Prae.

This change delivers the self-serve editing **platform and the public-site rebuild**: the hosting move, the authentication and role model that everything else builds on, and committee editing of club pages. It is **Phase 1a** — the piece with immediate value and the lowest legal risk, deliverable on its own. The member register migration builds on this platform in [member-register](../member-register/proposal.md) (Phase 1b); member self-service is [member-self-service](../member-self-service/proposal.md) (Phase 2).

## What Changes

- **BREAKING** — The public site moves from hand-written HTML files to HTML rendered from structured per-club data. Each club's content becomes a data file (venue, president, contact, schedule, fees, trial/beginner dates) plus a prose section, replacing the four-clubs-in-one-file structure of `locations.html`.
- **BREAKING** — Hosting consolidates from Netlify onto MyHost (cPanel PHP + MySQL). DNS and SSL for `kyudo.nz` move to MyHost alongside the already-hosted mail, ending the split-DNS confusion. The public site remains **static HTML served by Apache** with no runtime dependency on the application.
- New editing application (PHP + MySQL on MyHost) with **passwordless magic-link authentication** and long-lived (1-year) sessions. No passwords, no third-party OAuth/SSO.
- Three roles: **member** (defined here, used from Phase 2), **club committee** (scoped to their own club), **NZKF committee** (federation-wide). Roles and assignments live in the database with a web UI, not in repo config.
- Club committee can edit their own club's page and manage their own club's committee accounts. NZKF committee can do so for **any** club plus edit national pages. (Member-record management arrives with Phase 1b.)
- Content data is committed to a git repository (rollback + archival trail).
- Auto-deploy: code/template changes deploy on git push (cPanel Git); content edits render to the docroot directly on save (no deploy step).

## Capabilities

### New Capabilities
- `authentication`: Passwordless magic-link login, session lifetime, email delivery/deliverability, and account-to-record identity.
- `authorization`: The three-role model and the scoping rules that decide who may read/write which club pages and (from Phase 1b) which member records and fields.
- `club-content-editing`: The structured per-club data model, the committee-facing edit form, and rendering data to static public HTML in the docroot.
- `site-hosting`: Static public-site rendering on MyHost, DNS/SSL consolidation, git-backed content archival, and the deploy pipeline.

### Modified Capabilities
- None. This is the project's first spec-driven change; there are no existing specs in `openspec/specs/`.

## Impact

- **Public site**: All pages (`index.html`, `about.html`, `contact.html`, `locations.html`, `faq.html`, `events/*`) become rendered output. `locations.html` is the first to be split into per-club data. Duplicated `<head>`/nav across files collapses into a shared layout.
- **Hosting/DNS**: Migration off Netlify to MyHost; `kyudo.nz` DNS + SSL move to MyHost. `.htaccess` rewrites and Fathom analytics carry over.
- **New systems**: A PHP application, a MySQL database, a magic-link email sender, and the deploy pipeline (cPanel Git + render-on-save).
- **Depends on**: nothing — this is the foundation.
- **Unlocks**: [member-register](../member-register/proposal.md) (Phase 1b) reuses this platform's auth, roles, database, and hosting.
- **Out of scope here**: member records, membership lifecycle, retention (all Phase 1b); member accounts, self-editing, resources page (Phase 2).
