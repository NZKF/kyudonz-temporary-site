## Why

Phase 1 ([content-editing-and-hosting](../content-editing-and-hosting/proposal.md) + [member-register](../member-register/proposal.md)) gives ~15 committee members self-serve editing and moves the member register into MySQL, but members themselves still cannot log in. That leaves committees manually keeping ~65 members' details current, members unable to find the members-only Google Drive link (those not on Google misplace it and repeatedly ask where it is), and no self-reported data for IKYF age-group reporting.

This change extends the platform to the ~65 members: each member can log in, keep their own record current, and reach members-only resources. It reuses Phase 1's magic-link authentication, three-role model, and member register unchanged.

## What Changes

- Member accounts are activated for the ~65 members. Club committees invite their own club's members; each member's account is keyed to the email they chose in Phase 1.
- Members can log in (magic link) and **edit their own record** — the fields the register holds for them — reducing manual upkeep by committees.
- Members can **self-report a coarse age band** (not date of birth) to support IKYF age-group reporting.
- A **members-only page** displays the Google Drive resources link (and optionally a curated index), visible only to authenticated members and committee. The Drive files themselves remain link-accessible; this page just makes the link reliably findable behind login.
- The annual December check-in becomes **self-service**: a transactional email shows each member their currently held details and age band and asks them to correct anything that differs.

## Capabilities

### New Capabilities
- `member-self-service`: Member-facing login, self-editing of their own record, and age-band self-reporting.
- `member-resources`: The authenticated members-only resources page (Drive link / curated index) and its access rule.
- `member-annual-checkin`: The self-service December check-in email showing held details and prompting corrections.

### Modified Capabilities
- None. Phase 1's `authentication`, `authorization`, and `member-register` capabilities are reused as-is; this change adds member-facing surfaces on top of them without changing their requirements. (If, during design, member self-editing is found to require changing a Phase 1 requirement, a delta spec for that capability will be added here.)

## Impact

- **Accounts**: Scales from ~15 committee accounts to ~80 total (committee + 65 members); increases magic-link email volume (still low — on the order of a few hundred sends/year).
- **New surfaces**: member login area, self-edit form, members-only resources page, age-band field, annual check-in email.
- **Data**: adds a self-reported age-band field to the member record; no date of birth is stored.
- **Support**: introduces member-facing login support load and member onboarding (invites), previously avoided in Phase 1.
- **Depends on**: Phase 1 being live (auth, roles, register, hosting).
