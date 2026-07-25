## ADDED Requirements

### Requirement: Self-service annual check-in
The system SHALL support an annual (December) check-in in which each member receives a transactional email showing the details currently held for them, including their age band, and inviting them to correct anything that differs by editing their own record.

#### Scenario: Member receives and acts on the check-in
- **WHEN** the annual check-in is sent
- **THEN** each member receives an email showing their held details and age band, with a link to edit anything that is out of date

#### Scenario: Check-in complements the payment cycle
- **WHEN** the December cycle runs
- **THEN** the member self-service check-in covers details/age band while club-confirmed payment (from Phase 1) covers paid status; the two are independent
