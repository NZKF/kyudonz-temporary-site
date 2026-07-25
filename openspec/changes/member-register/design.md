## Context

Phase 1a ([content-editing-and-hosting](../content-editing-and-hosting/proposal.md)) delivers the platform: PHP + MySQL on MyHost, magic-link auth, the three-role model, and the deploy pipeline. This change adds the member register on top of it.

The register is governed by NZKF's constitution (Incorporated Societies Act 2022). Reading the constitution corrected two assumptions from exploration and pinned down the required fields:

- **The statutory Register of Members is narrow.** For each current member it requires only: name; date they became a member ("Unknown" allowed); email; telephone; and whether financial or unfinancial. For former members it requires only: name and the date they ceased, kept for 7 years. Everything else in the Google Sheet (IKF number, Japanese names, grades, preferred name, previous experience) is NZKF admin data, not the statutory register.
- **"Financial/unfinancial" and "member/ceased" are different axes.** A member who has not paid is *unfinancial* — still a member, but without rights (voting, facility access). They *cease* only by resignation, death, or a Committee resolution (e.g. remaining unfinancial past the constitutional window). The Sheet's single active/inactive flag conflated these; the design separates them.
- **Balance date is 31 December**; the financial year is 1 Jan–31 Dec, aligning with the December fee cycle.
- **Consent must be retained.** The constitution requires keeping each member's signed written consent to join.

## Goals / Non-Goals

**Goals:**
- One system of record for the register, editable by committee members without a Google account.
- Encode the statutory obligations (required fields, former-member record, 7-year retention, produce-on-request, consent) so compliance is structural, not manual.
- Field-level privacy that improves on the shared Sheet.
- Support the December financial cycle and IKYF reporting.

**Non-Goals:**
- No member self-editing or member accounts (Phase 2).
- No payments processing / dues invoicing — clubs collect and transfer; the system records financial status and per-club totals only.
- No grade-certification workflow. Grades are stored as current values for admin/IKYF, not as an audit trail.
- The statutory *contact person* for the Registrar (org metadata filed with the Registrar) is not modelled here.

## Decisions

### D1 — Model financial status and membership status as separate axes
- **Membership status:** `current` or `ceased` (+ `ceased_on` date). Cessation is an explicit recorded event, not derived from non-payment.
- **Financial status:** whether the member is financial for a given year, from the per-year payment record. The register's required "financial/unfinancial" field is derived from the current year.
- **Why:** the constitution treats them separately; conflating them (as the Sheet did) would either wrongly start the 7-year clock on a member who simply skipped a year, or wrongly retain full records of people who have actually ceased.

### D2 — Separate the statutory register subset from admin fields
- **Statutory subset:** name, date joined, email, telephone, financial/unfinancial.
- **Admin fields:** training location (club), IKF number, family/given names in Japanese and English, preferred English name, previous-experience note, shogo + date, dan + date, kyu, mobile.
- **Why:** the export (D5) must be able to produce *the statutory register* cleanly; retention (D4) acts on the subset while admin fields are dropped on cessation. Modelling the line now avoids entangling them later.

### D3 — Per-year financial/payment record for the December cycle
`member_years(member_id, year, financial, paid_on)`.
- **Why:** supports the December flow (club marks its members financial with a payment date), per-club totals for the treasurer's bulk-payment reconciliation, and IKYF "financial members in year N" / grade counts. Preserves history across lapse-and-rejoin. Churn is ~2–5 members/club/year, so the table is small.

### D4 — Statutory retention as an automated cron job, reducing to the minimum
- **On cessation:** record `ceased_on`, then reduce the record to **name + cessation date** — remove contact details and all admin fields including grades. (Matches the constitution's former-member record and the decision not to keep grade history; a rejoiner re-supplies their details.)
- **Nightly cron:** delete former-member records whose `ceased_on` is more than 7 years in the past. The purge keys on the explicit cessation date, so an active or rejoined member is never affected.
- **Why:** makes the rolling 7-year retention structural rather than a task someone must remember; reduces stored personal data to the statutory minimum as soon as it is lawful to do so.

### D5 — Self-serve register export
- A button lets the NZKF secretary/treasurer download the **current statutory register** plus the **7-year former-member trailing record**, with admin fields available as a separate/optional export.
- **Why:** satisfies "held at the registered office, produced on request" and removes Prae/SSH as the access path — the successor requirement. The nightly encrypted on-host dump (D7) is a backup of this, not the access route.

### D6 — Field-level visibility enforced by the system
- `name + email + club` → all committee (matches the existing announcements Google Group).
- Full record (telephone/mobile, IKF number, Japanese names, grades, experience) → the member's own-club committee + all NZKF committee.
- The NZKF treasurer, for reconciliation, needs only name + club + financial status; broader personal fields are visible to NZKF committee generally but not required for the treasurer's task.
- **Why:** the system enforces privacy independent of trust; least privilege applied where it does not complicate volunteers' work.

### D7 — Member data isolated from the public site, archived encrypted and on-host
- Member data lives in MySQL only. A nightly `mysqldump` is **encrypted (age/gpg) before it is written** and kept on the host alongside JetBackup — it is **not** committed to any git repository.
- The public render path (Phase 1a) has no access to the members table; no member personal data is ever written to the public repo or rendered to the docroot.
- **Why (revised after threat-model, Area B):** the original plan committed the dump to a private git repo. That puts the whole register in plaintext git history — replicated to every clone, effectively undeletable, and it **defeats the 7-year purge** (a member removed from MySQL survives in old dumps forever, breaching the retention *ceiling*). One misconfigured repo or a stolen laptop with a clone would leak the full register. Encrypting and keeping the dump on-host removes the undeletable-history and purge-defeat problems while still protecting confidentiality; JetBackup provides the offsite disaster-recovery copy.
- **Trade-off:** loses git's rotation-proof offsite history; JetBackup (DR) plus the manual encrypted register export (D5) cover the archival need.

### D8 — Consent capture
- Each member record carries a reference/flag recording that signed written consent to join is held (with date/where). Migration records consent as already held for existing members; new members capture it at creation.
- **Why:** the constitution requires retaining consent; a flag makes its presence auditable without storing the document itself in the web system.

## Risks / Trade-offs

- **Member data on an internet-facing app** vs a Sheet shared with ~15 Google accounts → mitigated by field-level visibility (D6), data minimisation (D2/D4), isolation from the public site (D7), and Phase 1a's auth.
- **Cron purge and cessation reduction are destructive** → run in dry-run/log mode first; the nightly private dump is the safety net; reduction only fires on an explicit cessation event.
- **Migration fidelity** → verify counts and required fields against the Sheet; keep the Sheet read-only as fallback until a full cycle or sign-off.
- **Financial-status derivation** could drift from the constitution's exact unfinancial windows → confirm the constitutional timing (1 month / 60 working days) with the committee before encoding automated status transitions; Phase 1b may record status as set by committees rather than auto-transition.

## Migration Plan

1. On the Phase 1a platform, create the member tables (`members`, `member_years`) as `utf8mb4`.
2. Confirm the required-field list and consent handling against the constitution with the committee.
3. Migrate the Google Sheet → MySQL; verify counts and required fields; mark existing members' consent as held; keep the Sheet read-only as fallback.
4. Build committee-facing member management + field-level visibility.
5. Build the December financial cycle (mark financial, per-club totals).
6. Implement cessation reduction and the 7-year purge in dry-run, then live.
7. Build the register export; verify the secretary/treasurer can produce it unaided.
8. Set up the nightly encrypted on-host dump (age/gpg, not git-committed); verify member data never reaches the public repo/site.
9. Retire the Sheet as system of record after a full December cycle or committee sign-off.

**Rollback:** member features can be disabled without affecting the public site or Phase 1a committee editing; the Sheet remains the fallback until step 9.

## Open Questions

- The exact constitutional timing for unfinancial → cessation transitions — whether Phase 1b auto-transitions or records committee-set status.
- Register export format (CSV vs PDF vs both) and whether point-in-time snapshots are needed beyond "current + who-left-when".
- Whether Phase 1b must be live before December 2026 (run the new cycle) or that December runs on the Sheet with Phase 1b landing after.
- Whether the treasurer needs a dedicated reconciliation view or the general NZKF-committee member view suffices.
