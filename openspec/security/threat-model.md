# NZKF site — threat model

> Status: **walk-through complete** (2026-07-25). The STRIDE-lite walk across all
> eight data flows has been done; the two design forks (A, B) are resolved; and
> the resulting requirements are threaded into the `content-editing-and-hosting`
> (1a) and `member-register` (1b) specs, plus a "Security hardening" section in
> each change's tasks.md. This file now records the model and the decisions.
> Scope confirmed: **1a + 1b together**. See "Walk-through outcomes" below.

## Scope

Threat-model 1a + 1b **together** (auth and PII are inseparable). Phase 2
(member self-service) inherits the same model and is out of scope for now.

Surface is deliberately small: ~80 users, one PHP app, one MySQL DB, one host
(MyHost cPanel) we control, no payments, no public write path, no third-party
integrations. Risks are standard OWASP, not exotic.

## Assets to protect

- Member PII: name, email, telephone/mobile, IKF number, Japanese names,
  grades, training location. (Statutory register + admin fields.)
- Account/session integrity (esp. high-privilege NZKF committee accounts).
- Public site integrity (no defacement / injected script).
- The statutory register's correctness and availability (legal duty).

## Actors / trust levels

- Public (unauthenticated) — read public site only.
- `member` (Phase 2) — self only.
- `club committee` — own club's page + (1b) own club's members.
- `NZKF committee` — everything: all member PII, all pages. **High-value target.**
- Attacker with a compromised committee email inbox (magic-link = inbox owns account).
- Other tenants / the platform on shared cPanel hosting.

## Data flows to walk (STRIDE each)

1. **Login** — request magic link → email → click → session established.
2. **Club page edit** — committee edits fields/prose → render to docroot → served publicly.
3. **Member record read/write** (1b) — committee views/edits member; field-level visibility.
4. **December cycle** (1b) — mark financial; per-club totals to treasurer.
5. **Retention** (1b) — cessation reduction; nightly 7-year purge.
6. **Archival dump** (1b) — nightly mysqldump → private repo.
7. **Register export** (1b) — secretary/treasurer downloads register.
8. **Code/content deploy** — git push → cPanel Git; render-on-save.

## Threat areas identified so far

### A. Uniform auth vs non-uniform privilege — DESIGN FORK (settle before building 1a auth)
Every account gets magic-link + 1-year session. But NZKF committee can read all
65 members' PII and edit every page — email-only auth + year-long session is
generous for that blast radius.
Options: shorter sessions for NZKF committee · step-up re-auth for sensitive
actions (register export, full-member view) · accept for ~15 trusted people.
→ **RESOLVED (2026-07-25): shorter NZKF-committee sessions.** ~1 year for club
committee, ~30 days for NZKF committee (shorter governs when both are held).
Plus an *optional*, host-dependent region-fence on the app (not the public site)
to NZ/AU/JP source IPs with an admin override — defence-in-depth, not a primary
control. Threaded into the `authentication` spec (tiered lifetime, session
integrity/revocation, region-restricted access) and design D2.

### B. Private PII dump is a new exfil surface — REOPENS 1b design D7
Nightly mysqldump → private git repo puts the whole register in plaintext git
history: replicated to every clone, effectively undeletable, and it DEFEATS the
7-year purge (gone from MySQL, still in old dumps). One misconfigured repo or
stolen laptop = full register leak.
Options: encrypt dump before commit (age/gpg) · keep dump on MyHost only, not
GitHub · drop the git dump, rely on JetBackup + manual encrypted export.
→ **RESOLVED (2026-07-25): encrypted, on-host only.** Nightly dump is age/gpg
encrypted and kept on the host alongside JetBackup, **not** committed to any git
repo — this avoids the undeletable-plaintext-history problem *and* the
purge-defeat (encrypting a git-committed dump would fix confidentiality but still
retain purged members in history forever). Threaded into the
`membership-lifecycle` spec (archival requirement) and design D7; proposal +
migration plan references updated.

### C. Broken access control / IDOR — MOST LIKELY TO BITE (OWASP #1)
Invisible in a demo; fails when someone changes an ID in a URL. Solo-dev failure
mode: permission checked in UI but not on the server handler.
Mitigation is architectural: ONE authorization choke-point every data access
flows through, not scattered per-endpoint checks.
→ Shapes app structure. Belongs in `authorization` spec (both changes).

### D. Stored XSS via the render pipeline
Committee edits → static HTML served to public. Prose field especially. Raw HTML
passthrough lets a compromised/careless committee account inject script that runs
on every public visitor.
Decision: prose = Markdown through a sanitizing renderer, output-encoded, never
raw HTML.
→ Belongs in `club-content-editing` spec in 1a.

### E. Secrets & shared-hosting hygiene (checklist)
DB creds, email API key, session secret. On cPanel the app user ≈ web user:
config outside docroot, `.env` gitignored, never in the repo (we use git for
code+content). Free wins in cPanel: ModSecurity on, **2FA on the cPanel panel
itself** (master key to everything).

### F. Rate-limit the magic-link endpoint (checklist)
Without it: email-bomb a victim; enumerate accounts by timing. Cheap; easy to forget.

## Cross-cutting checklist (thread into tasks.md of 1a/1b)

- [ ] Session cookies: HttpOnly, Secure, SameSite; rotate on login; server-side revocation
- [ ] Session revocation when someone leaves a committee (kill their sessions)
- [ ] Magic-link tokens: high entropy, single-use, short expiry, never logged, not in URLs that leak via Referer
- [ ] Authorization enforced server-side at a single choke-point (IDOR)
- [ ] Parameterised queries everywhere (no string-built SQL)
- [ ] Output encoding / sanitisation on the render path (XSS)
- [ ] Mass-assignment protection on edit forms (allow-list fields)
- [ ] CSRF protection on state-changing POSTs
- [ ] Rate limiting on login/magic-link
- [ ] Secrets outside docroot, gitignored; never in code+content repo
- [ ] ModSecurity enabled; cPanel panel 2FA enabled
- [ ] PHP kept patched (MultiPHP); a plan for who patches when Prae is away
- [ ] PII dump strategy resolved (area B) and, if kept, encrypted
- [ ] Register export access-controlled to secretary/treasurer only

## Walk-through outcomes (2026-07-25)

Scope confirmed: **1a + 1b together**. Both design forks resolved (A, B — see above).
STRIDE-lite covered all eight data flows; areas A–F mapped to flows and, beyond
the original cross-cutting checklist, the walk surfaced six new requirements now
threaded into the specs:

1. **Render targets are system-derived** from a fixed page allow-list — a save can
   never write outside the docroot / to a user-influenced path (Flow 2, arbitrary
   write). → `club-content-editing`.
2. **Edit forms accept only allow-listed fields** + CSRF on state-changing saves
   (Flow 2/3, mass assignment). → `club-content-editing`, `authorization` (1b).
3. **Single server-side authorization choke-point** for member access; scope +
   field visibility enforced per request, never in the UI (Flow 3, IDOR / OWASP #1).
   → `authorization` (1b).
4. **Register export streamed as an authenticated download**, never written to a
   public path (Flow 7). → `membership-lifecycle`.
5. **Cessation is an attributable event** (actor + reason category + date) recorded
   before the record is reduced (Flow 5, statutory / repudiation). →
   `membership-lifecycle`.
6. **Sensitive member actions logged** (mark-financial, cessation, export) with
   actor + time (Flows 4/5/7, repudiation). → `membership-lifecycle`.

The cross-cutting checklist above is threaded into each change's tasks.md as a
"Security hardening" section (1a §6, 1b §7).

### Still open / to confirm during implementation

- Whether to add step-up re-auth on register export **in addition** to shorter
  NZKF sessions (deferred; shorter sessions chosen as the primary control).
- Region-fence is contingent on MyHost supporting source-IP geolocation
  (`mod_maxminddb`/`mod_geoip`) — verify host capability before relying on it.
- Deliverability proof (D2) before Phase 1b relies on email for member flows.
