## 1. Data model

- [ ] 1.1 Confirm the statutory required-field list and consent handling against NZKF's constitution with the committee
- [ ] 1.2 Create the `members` table (statutory subset + admin fields, `utf8mb4`), with membership status (`current`/`ceased`) + `ceased_on`, and a consent-held reference/flag
- [ ] 1.3 Create the `member_years` table (member, year, financial, paid_on)

## 2. Migration

- [ ] 2.1 Migrate the Google Sheet into MySQL; map columns to statutory subset vs admin fields
- [ ] 2.2 Mark all migrated existing members as having consent held
- [ ] 2.3 Verify counts and required fields against the Sheet; keep the Sheet read-only as fallback

## 3. Committee-facing management

- [ ] 3.1 Build member create/view/edit, scoped to own club (club committee) / any club (NZKF)
- [ ] 3.2 Implement field-level visibility (name/email/club to all committee; full record to own-club + NZKF)
- [ ] 3.3 Implement English-name sort/search over the member list
- [ ] 3.4 Extend role scoping (authorization delta) to member records

## 4. Financial cycle

- [ ] 4.1 Build the mark-financial flow (club marks own members financial for the year, with payment date)
- [ ] 4.2 Build per-club financial totals for NZKF treasurer reconciliation
- [ ] 4.3 Provide financial-count and grade-count reporting for IKYF

## 5. Lifecycle & retention

- [ ] 5.1 Implement cessation: record `ceased_on`, reduce record to name + cessation date (remove contact + grades)
- [ ] 5.2 Implement the nightly 7-year purge keyed on cessation date; run in dry-run/log mode first, then live
- [ ] 5.3 Confirm unfinancial→cessation timing with the committee; decide auto-transition vs committee-set status

## 6. Export & archival

- [ ] 6.1 Build the self-serve register export (current statutory register + 7-year former-member record) for secretary/treasurer
- [ ] 6.2 Set up the nightly member-data dump to a private repository
- [ ] 6.3 Verify member data never reaches the public repo and the public render path cannot read the members table

## 7. Go-live & handover

- [ ] 7.1 Update the admin guide with member management, marking financial, cessation, and register export
- [ ] 7.2 Retire the Google Sheet as system of record after a full December cycle or committee sign-off
