## Why

NZKF's 65-person member register lives in a Google Sheet that committee members without a Google account cannot edit, forcing them to relay changes through others. The register is also a **statutory Register of Members** under the Incorporated Societies Act 2022 (and NZKF's constitution), with defined required fields, a former-member record, and a 7-year retention duty — obligations currently resting on a shared spreadsheet with "sketchy" access control. The annual December fee cycle (each club marks its members financial, transfers a bulk payment, NZKF reconciles) costs an estimated 10–15 volunteer hours.

This change (**Phase 1b**) migrates the register into the platform built by [content-editing-and-hosting](../content-editing-and-hosting/proposal.md), reusing its magic-link auth, role model, database, and hosting. It gives committees a proper editor, enforces field-level privacy, encodes the statutory retention rules, and supports the December financial cycle — a privacy and compliance improvement over the Sheet.

## What Changes

- The member register migrates off Google Sheets into MySQL as the **single system of record**. The Sheet is retired.
- The data model distinguishes the **statutory register subset** (name, date joined, email, telephone, financial/unfinancial status) from **NZKF admin fields** (training location, IKF number, Japanese and English names, preferred name, previous experience, shogo/dan/kyu and dates, mobile).
- **Financial status and membership status are modelled as distinct**, per the constitution: a member who has not paid is *unfinancial* (still a member, no rights), not *ceased*. Cessation is a separate recorded event (resignation, death, or Committee resolution) with its own date.
- Committee-facing member record management: club committee create/view/edit their **own club's** members; NZKF committee for **any** club.
- **Field-level visibility**: name + email + club visible to all committee (as in the existing Google Group); the full record only to the member's own-club committee and NZKF committee.
- **Per-year financial + payment tracking** supports the December cycle: each club marks its own members financial for the year with a payment date; per-club totals let the NZKF treasurer reconcile the bulk transfer. Balance date is 31 December.
- **Statutory retention**: on cessation the record is reduced to **name + cessation date** (contact and grades removed); former-member records are purged once more than seven years past the cessation date. Implemented as an automated cron job.
- **Consent capture**: the constitution requires retaining each member's signed written consent to join — recorded as a stored reference/flag per member.
- **Self-serve register export**: the NZKF secretary or treasurer can produce the current register plus the seven-year former-member trailing record, without developer involvement — satisfying the "produce on request" duty.
- **Data isolation and archival**: member data lives in MySQL with a nightly **encrypted, on-host** dump (not committed to any git repository — see threat-model Area B); it is never written to the public content repo and the public render path has no access to it.

## Capabilities

### New Capabilities
- `member-register`: The member data model (statutory subset + admin fields), committee-facing record management, migration off Google Sheets, field-level visibility, and consent capture.
- `membership-lifecycle`: Financial vs. membership status, per-year financial/payment tracking, the December cycle, cessation handling, statutory 7-year retention, the register export, and member-data isolation/archival.

### Modified Capabilities
- `authorization`: Extends the role scoping from [content-editing-and-hosting](../content-editing-and-hosting/proposal.md) to member records — club committee scoped to their own club's members, NZKF committee to any, plus field-level visibility. (Delta spec against the `authorization` capability introduced in Phase 1a.)

## Impact

- **Depends on**: [content-editing-and-hosting](../content-editing-and-hosting/proposal.md) (Phase 1a) being live — auth, roles, MySQL, hosting, deliverability.
- **New systems**: member tables in the existing MySQL database; a cron job (cessation reduction + 7-year purge); a nightly encrypted on-host dump; a register-export feature.
- **Data**: Google Sheet migrated and retired as system of record; the Sheet kept read-only as fallback until sign-off.
- **Legal**: statutory Register of Members, former-member record, 7-year retention, consent retention, and produce-on-request now sit in this system.
- **Out of scope**: member self-editing, age-band self-reporting, members-only resources page, per-member payment-confirmation emails — all Phase 2 ([member-self-service](../member-self-service/proposal.md)).
