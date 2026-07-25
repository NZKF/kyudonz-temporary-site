## 1. Member login (reuse Phase 1 auth)

- [ ] 1.1 Activate the `member` role and a member login area using the existing magic-link flow
- [ ] 1.2 Enforce member scope: own record only, no club/national content, no other members

## 2. Invite & claim

- [ ] 2.1 Build club-scoped member invite (committee invites own club; NZKF any club), sent to the record's email
- [ ] 2.2 Implement account claim binding the account to the existing member record
- [ ] 2.3 Implement the correct-email-then-reinvite recovery path

## 3. Self-editing & age band

- [ ] 3.1 Build the member self-edit form over their own register fields
- [ ] 3.2 Add the coarse age-band field (fixed ranges, no DOB) to the record and form; confirm bands against IKYF categories
- [ ] 3.3 Add age-band count reporting for the NZKF secretary

## 4. Members-only resources

- [ ] 4.1 Build the resources page behind member/committee authentication
- [ ] 4.2 Present the Drive link (and optional curated index); deny unauthenticated access
- [ ] 4.3 Verify member data is never rendered on this page

## 5. Annual check-in

- [ ] 5.1 Build the December check-in email showing held details + age band with an edit link
- [ ] 5.2 Confirm it operates independently of the Phase 1 club-confirmed payment flow

## 6. Rollout

- [ ] 6.1 Pilot invites with one club; verify claim, self-edit, and resources access
- [ ] 6.2 Roll out invites club by club
- [ ] 6.3 Enable the self-service check-in for the next December cycle
- [ ] 6.4 Update the admin guide with member onboarding and invite steps
