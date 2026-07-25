## Context

kyudo.nz is currently ~8 hand-written static HTML files deployed to Netlify from git, plus DNS split between Netlify (site) and MyHost (mail). The federation is four clubs (Auckland, Palmerston North, Wellington, Christchurch) under NZKF; a person can hold both a club committee role and an NZKF committee role. Content edits (~35/year) bottleneck on one volunteer.

This is **Phase 1a**: the platform (hosting, auth, roles) and the club-page editing that runs on it. The member register migration ([member-register](../member-register/proposal.md), Phase 1b) and member self-service (Phase 2) build on this foundation and are out of scope here — but the auth and role model are designed so they slot in without reshaping.

Hard constraints discovered during exploration:
- **Host is MyHost cPanel: PHP + MySQL, SSH, Cron. No Node.js runtime.** This rules out Better Auth / any Node-based CMS.
- Cost must not increase; MyHost is already paid for (mail hosting), Netlify is free. Consolidating onto MyHost is cost-neutral and simplifies DNS/SSL.
- Committee members log in ~3–4 times/year and forget passwords. Not all have Google accounts.
- The successor admin (future NZKF IT/secretary) is non-technical and must run identity and content without SSH, git, or a build step. Prae may step away for years and remains only the last-resort maintainer of the build itself.

## Goals / Non-Goals

**Goals:**
- Club committees self-serve their own club page without waiting on Prae.
- The public site keeps **zero runtime dependency** on the application (the property that made this site resilient when the Joomla site went down).
- Boring, few moving parts, single host, recoverable by a non-developer for content and identity.
- An auth + role foundation that Phase 1b (member register) extends without reshaping.

**Non-Goals:**
- Not a general CMS. Content is largely structured fields, not free-form documents.
- No approval/PR workflow — the correct role is sufficient trust to publish directly.
- No passwords, no third-party OAuth/SSO providers.
- No member records, membership lifecycle, retention, or register export — those are Phase 1b.
- Phase 2 items (member accounts, self-editing, age band, members-only page) are excluded.

## Decisions

### D1 — One PHP+MySQL app on MyHost that renders static HTML to the docroot
The app writes structured data and, on save, renders the affected public pages to static `.html` in the Apache docroot. Apache serves those files directly.
- **Why over "everything dynamic PHP":** dynamic rendering makes the public site depend on the app being healthy — that is Joomla's failure shape, the thing that put this project here. Rendering to static preserves the resilience property.
- **Why over "Eleventy on Netlify + separate PHP app":** three systems (Eleventy, Netlify, PHP) + a GitHub token that expires + a webhook, all for a non-technical successor to understand. With no Node on the host, Eleventy would run off-host, splitting the stack. 8 pages and 4 club blocks is a `render()` function, not a framework.
- **Trade-off:** gives up Eleventy (which Prae knows) for ~100 lines of PHP templating. Accepted for single-host simplicity and successor-recoverability.

### D2 — Passwordless magic-link auth, sessions tiered by blast radius
Login = enter email → receive a one-time link → click → session cookie. Lifetime is tiered (threat-model Area A): **~1 year for club committee**, **~30 days for NZKF committee**; where a person holds both, the shorter lifetime governs.
- **Why over SSO (Google/Apple/Microsoft):** SSO was only ever proposed to fix password-forgetting. Magic links delete the password entirely. SSO adds provider app registrations, Apple Developer Program cost (US$99/yr), and — critically — Apple "Hide My Email" and provider-switching break the ability to match a login to a member record. The email *is* the identity, which matches the requirement that the member chooses which email is their account.
- **Why tiered sessions:** a club-committee login (own club page only) has a small blast radius, so ~1 year suits someone logging in 3–4×/year. An NZKF-committee session grants federation-wide read of all member PII and edit of every page — email-only auth plus a year-long cookie is too generous for that, so NZKF sessions expire in ~30 days.
- **Region fence (optional, host-dependent):** where MyHost supports source-IP geolocation, the app (not the public site) restricts logins to NZ/AU/JP with an admin override — defence-in-depth against opportunistic/bot login attempts, not a primary control (a VPN defeats it, and it can lock out a travelling officer).
- **Session hygiene:** HttpOnly/Secure/SameSite cookies, id rotated on login, server-side session state so a session can be revoked when a role is removed.
- **Risk → mitigation:** magic-link **deliverability** to Gmail from shared hosting is the main failure mode → use cPanel Email Deliverability (SPF/DKIM green) and/or a transactional email service free tier; volume is ~260 sends/year.

### D3 — Three roles, scoped by data ownership
`member` (Phase 2), `club committee` (scoped to one club), `NZKF committee` (federation-wide superset). Permission checks reduce to scope: NZKF = all; club committee = `WHERE club = their_club` and club-page path prefix; member = self only.
- **Why no separate treasurer/secretary/president roles:** NZKF treasurer and secretary get the same permissions as NZKF committee for redundancy (any officer can cover another who is suddenly indisposed — a real past problem). Within a club, the three officers "police each other." Simplicity for volunteers beats granular least-privilege here; least privilege is still applied at the *field* level (D6).
- **NZKF acts on any club:** NZKF committee can mark any club's members paid and edit any club's committee — the escape hatch for a club that loses its secretary.
- **No club-admin sub-role:** any club committee member can add/remove their own club's committee accounts and members.

### D4 — Roles and users live in MySQL with a web UI, never in a repo config file
- **Why:** the successor must assign roles from a browser with no SSH/git/deploy. Any git-backed approach (roles.yml) means only people who can commit and deploy can change roles — i.e. only Prae. This is the concrete requirement that makes this an app, not a git CMS.

### D5 — Content lives in (public) git; the member datastore is deferred but reserved
- Club/national content → data files committed to a public git repo → rollback + public-safe audit trail.
- The authorization layer (D3) already reserves member-record and field-level scopes so Phase 1b can add the members table without reshaping roles. Phase 1b will hold member data in MySQL with a private dump, physically isolated from the public render path — but no member data exists in Phase 1a.

### D6 — Deploy: two distinct paths
- **Content deploy** (committee edits): app renders to docroot on save — there is no upload/deploy step.
- **Code deploy** (Prae changes templates/app): git push → cPanel Git Version Control (`.cpanel.yml`) pulls and deploys. Preserves git-push-to-deploy without Netlify.

### D7 — Host + DNS consolidation onto MyHost
Move `kyudo.nz` DNS and SSL to MyHost alongside mail (`mail.kyudo.nz` SSL already there). Retire Netlify for the public site.
- **Why:** one host, one SSL story, one place DNS lives; cost-neutral. Ends the current split where site DNS is on Netlify and mail is on MyHost.

## Risks / Trade-offs

- **Magic-link deliverability** → D2 mitigation (SPF/DKIM / transactional free tier). This is the single most likely thing to make the system feel broken, and it is worth proving before Phase 1b relies on it for member invites.
- **Losing Eleventy familiarity** → accepted per D1; templating is small and the successor benefit is large.
- **Custom auth/authz code is ours to maintain** → magic-link + scope checks is a small, well-understood surface; far smaller than a Node auth stack + DB + editor UI. Bus-factor mitigated by "boring, few moving parts."
- **DNS/SSL migration** carries a cutover risk (mail must not break) → migration plan sequences DNS carefully and verifies mail before/after.

## Migration Plan

1. Stand up the PHP app + MySQL on MyHost under a path/subdomain; no public traffic yet.
2. Restructure `locations.html` → per-club data files + shared layout; verify rendered output matches current site byte-for-intent.
3. Render all existing pages via the new layout on MyHost; compare against live Netlify site.
4. Consolidate DNS/SSL to MyHost — sequence so mail continues uninterrupted; verify mail send/receive before and after.
5. Cut public `kyudo.nz` over to MyHost-served static output; retire Netlify.
6. Enable magic-link auth for ~15 committee accounts and club-page editing.

**Rollback:** the public site is static files — reverting DNS to Netlify restores the prior site.

## Open Questions

- Magic-link sending: MyHost SMTP (SPF/DKIM via cPanel Email Deliverability) vs a transactional provider free tier — decide during implementation based on a deliverability test to Gmail.
- Whether the shared layout also absorbs the events pages now, or those follow later (they were the highest-churn pages historically but there is no current event to publish).
