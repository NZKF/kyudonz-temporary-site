## ADDED Requirements

### Requirement: Member login
Members SHALL authenticate using the same passwordless magic-link mechanism as committee members, with the same session lifetime. The `member` role SHALL scope a person to their own record only.

#### Scenario: Member logs in
- **WHEN** a member requests a login link at their registered email and clicks it
- **THEN** the system establishes a session scoped to their own record only

### Requirement: Member invite and account claim
Club committees SHALL be able to invite their own club's members; NZKF committee MAY invite any club's members. The invite SHALL be sent to the email on the member's existing register record, and claiming it SHALL bind the account to that record.

#### Scenario: Member claims their account
- **WHEN** a member clicks the invite sent to their registered email
- **THEN** an account is established for their existing member record

#### Scenario: Wrong email is corrected then re-invited
- **WHEN** a member's registered email is unreachable
- **THEN** their club committee (or NZKF) updates the record's email and re-sends the invite

### Requirement: Members edit their own record
A logged-in member SHALL be able to view and edit their own register fields. They SHALL NOT be able to view or edit other members' records or any club/national content.

#### Scenario: Member updates their mobile number
- **WHEN** a member edits their mobile number and saves
- **THEN** their record is updated without committee involvement

#### Scenario: Member cannot see others
- **WHEN** a logged-in member attempts to view another member's record
- **THEN** the system denies access

### Requirement: Age-band self-reporting
A member SHALL be able to self-report a coarse age band from a fixed set of ranges. The system SHALL NOT collect or store date of birth.

#### Scenario: Member selects an age band
- **WHEN** a member selects their age band
- **THEN** the band is stored on their record and no date of birth is requested or stored

#### Scenario: Age bands support IKYF reporting
- **WHEN** the NZKF secretary needs age-group counts
- **THEN** the system can report counts by band from members' self-reported values
