# NZKF site — threat model (scaffold / work in progress)

> Status: **scaffold**, captured at the end of an explore session so a fresh
> session can continue without losing reasoning. The full STRIDE walk-through
> has NOT been done yet. This lists scope, the data flows to walk, and the six
> threat areas already identified (with the open decisions). Next step: walk
> each data flow, run STRIDE-lite, and thread resulting requirements into the
> `content-editing-and-hosting` (1a) and `member-register` (1b) specs, plus a
> security checklist in each change's tasks.md.

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
→ Decision pending. Affects `authentication` spec in 1a.

### B. Private PII dump is a new exfil surface — REOPENS 1b design D7
Nightly mysqldump → private git repo puts the whole register in plaintext git
history: replicated to every clone, effectively undeletable, and it DEFEATS the
7-year purge (gone from MySQL, still in old dumps). One misconfigured repo or
stolen laptop = full register leak.
Options: encrypt dump before commit (age/gpg) · keep dump on MyHost only, not
GitHub · drop the git dump, rely on JetBackup + manual encrypted export.
→ Decision pending. Affects `membership-lifecycle` spec + design D7 in 1b.

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

## Open decisions to resolve next session

1. **A** — privilege tiering for NZKF committee (session length / step-up / accept).
2. **B** — PII dump strategy (encrypt / on-host-only / drop).
3. Scope confirm: 1a + 1b together (recommended).
