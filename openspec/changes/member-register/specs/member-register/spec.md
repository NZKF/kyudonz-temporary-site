## ADDED Requirements

### Requirement: Member register as single system of record
The member register SHALL be stored in MySQL as the single system of record, replacing the Google Sheet. The Google Sheet SHALL be retired as the system of record after migration.

#### Scenario: Register migrated off the Sheet
- **WHEN** the register is migrated
- **THEN** every current member's fields are present in MySQL and the Google Sheet is no longer the system of record

#### Scenario: Japanese text is stored correctly
- **WHEN** a member's name is stored in Japanese
- **THEN** the system stores and displays the characters correctly (full Unicode support)

### Requirement: Statutory register subset distinguished from admin fields
The system SHALL hold, and be able to identify as the statutory Register of Members, the fields required by the constitution: name, date the person became a member (recorded as "Unknown" where no record exists), email address, telephone number, and whether the member is financial or unfinancial. The system SHALL also hold NZKF admin fields — training location (club), IKF number, family and given names in Japanese and English, preferred English name, previous-experience note, shogo and date acquired, dan and date acquired, kyu, and mobile number — modelled as distinct from the statutory subset.

#### Scenario: Statutory fields are present for every current member
- **WHEN** a current member's record is stored
- **THEN** it carries name, date joined (or "Unknown"), email, telephone, and financial/unfinancial status

#### Scenario: Statutory subset can be produced without admin fields
- **WHEN** the statutory register is produced
- **THEN** it can be output using only the statutory subset, without requiring the admin fields

### Requirement: Consent to join is retained
The system SHALL record, for each member, that their signed written consent to become a member is held (with a date or reference). Migrated existing members SHALL be recorded as having consent held; new members SHALL have consent recorded at creation.

#### Scenario: Consent recorded on creation
- **WHEN** a committee member creates a new member record
- **THEN** the record captures that signed written consent to join is held

#### Scenario: Migrated members marked as consented
- **WHEN** existing members are migrated from the Sheet
- **THEN** their records are marked as having consent held

### Requirement: Committee-facing member record management
Club committee members SHALL be able to create, view, and edit member records for their own club. NZKF committee members SHALL be able to do so for any club. Each club is expected to hold at least a president, secretary, and treasurer among its committee.

#### Scenario: Club committee adds a new member
- **WHEN** an Auckland committee member creates a member record for their club
- **THEN** the record is stored against Auckland and the member can later be invited (Phase 2)

#### Scenario: Club committee cannot manage another club's members
- **WHEN** an Auckland committee member attempts to view or edit a Wellington member's full record
- **THEN** the system denies access

### Requirement: Field-level visibility
The system SHALL enforce field-level visibility independent of any user's trust level. Name, email, and club SHALL be visible to all committee members. The remaining personal fields (telephone/mobile, IKF number, Japanese names, grades, experience) SHALL be visible only to the member's own club committee and to NZKF committee.

#### Scenario: Cross-club committee sees only name, email, club
- **WHEN** a Wellington committee member views Auckland members
- **THEN** they see only name, email, and club — not telephone, grades, or other personal fields

#### Scenario: Own-club committee sees the full record
- **WHEN** an Auckland committee member views an Auckland member
- **THEN** they see the full record

### Requirement: Committee sort and search in English
The committee-facing member views SHALL sort and search by English names.

#### Scenario: Sorting the member list
- **WHEN** a committee member sorts or searches the member list
- **THEN** ordering and matching use the English name fields (while Japanese fields remain stored and displayed)
