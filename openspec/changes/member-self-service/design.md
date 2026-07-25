## Context

This builds directly on Phase 1. Authentication (passwordless magic link, 1-year sessions, email-as-identity), the three-role model, the MySQL member register, and MyHost static hosting all already exist. This change only adds member-facing surfaces and one new field (age band). The driving problems: committees manually maintaining 65 records, members (especially non-Google members) unable to find the members-only Drive link, and no self-reported age data for IKYF.

## Goals / Non-Goals

**Goals:**
- Members keep their own records current, reducing committee upkeep.
- Members reliably reach members-only resources behind a login they can actually use.
- Capture coarse age-band data by self-report for IKYF, without storing sensitive DOB.

**Non-Goals:**
- No new auth mechanism — reuse Phase 1 magic links exactly.
- No payment processing or dues collection through the site (December stays: club confirms, treasurer reconciles).
- No storage of date of birth.
- No public exposure of the resources page or member data.

## Decisions

### D1 — Reuse Phase 1 auth and roles unchanged
Members use the same magic-link login. The `member` role (defined but dormant in Phase 1) is simply activated. A member's scope is "self only," already specified in Phase 1 authorization.
- **Why:** avoids a second auth surface; the hard identity work (email-as-identity, deliverability) is already solved.

### D2 — Invite/claim flow keyed to the Phase 1 email
Club committees invite their own club's members. The invite goes to the email already on the member's Phase 1 record; clicking it establishes the member's account for that record.
- **Why:** sidesteps the account-to-record matching problem (e.g. Apple Hide My Email) — the record's email is authoritative and the member logs in with it. A member who wants a different email changes it via the Phase 1 self-service email change once logged in.
- **Recovery:** if a member's email is wrong/unreachable, their club committee (or NZKF) corrects the record's email, then re-invites.

### D3 — Age band is a coarse self-reported enum, refreshed annually
Store a band (e.g. `<18, 18–29, 30–49, 50–64, 65+`), never DOB.
- **Why:** it is exactly what IKYF asks for, far less sensitive than DOB, and carries no birthday-based identity-theft surface.
- **Staleness:** a band drifts as people age, so the December check-in re-prompts it.

### D4 — Members-only page gates the link, not the files
The page lives behind member/committee login and shows the Drive link (and optionally a curated index). The Drive files stay link-accessible (as today, since not all members are on Google).
- **Why:** the actual problem is "members lose the link," not file-level access control. Gating the link behind login makes it reliably findable without re-architecting Drive access. Public visibility of the link is explicitly not wanted even though the files are link-viewable.

### D5 — December check-in is self-service email + edit
The transactional email shows the member their currently held details and age band and links them to edit anything that differs.
- **Why:** turns the committee's manual "chase and update" into member-driven correction. Phase 1 already handles the money side (club marks paid); this is the details side.

## Risks / Trade-offs

- **Member login support load** (65 infrequent users) → mitigated by magic links (nothing to forget) and 1-year sessions; committees can re-invite.
- **Onboarding 65 accounts** → invites are club-driven and can be staggered; not all-at-once.
- **Email deliverability at higher volume** → same mitigation as Phase 1 (SPF/DKIM / transactional provider); volume still low.
- **Resources page leaking** → enforced behind authentication; member data never rendered there.
- **Self-reported data quality** (age band, details) → annual check-in refresh; committees can still correct records.

## Migration Plan

1. Activate the member role and member login area (reusing Phase 1 auth).
2. Add the age-band field to the member record and the self-edit form.
3. Build the members-only resources page and its access rule.
4. Pilot invites with one club; verify claim flow and self-edit.
5. Roll out invites club by club.
6. Introduce the self-service December check-in email for the next cycle.

**Rollback:** member-facing surfaces can be disabled without affecting Phase 1 committee operation or the public site; committees resume manual record upkeep.

## Open Questions

- Exact age-band boundaries — confirm against the specific IKYF reporting categories.
- Whether the resources page is just the Drive link or also a curated index of what's inside.
- Invitation cadence — all clubs at once vs staggered — and who sends invites (club committee vs NZKF).
