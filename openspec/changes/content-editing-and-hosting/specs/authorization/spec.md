## ADDED Requirements

### Requirement: Three-role model
The system SHALL support exactly three roles: `member`, `club committee`, and `NZKF committee`. A single person MAY hold both a club committee role (for one or more clubs) and an NZKF committee role. Roles and role assignments SHALL be stored in the database and managed through a web UI, not in repository configuration files.

#### Scenario: A person holds both a club and an NZKF role
- **WHEN** a person is an NZKF committee member and also on the Auckland club committee
- **THEN** they have NZKF-wide permissions and are additionally scoped to Auckland where club-specific attribution matters

#### Scenario: Roles are managed without developer tooling
- **WHEN** an authorised administrator assigns or removes a role
- **THEN** they do so entirely through the web UI, with no SSH, git, or deploy step

### Requirement: NZKF committee has federation-wide access
NZKF committee members SHALL be able to edit any club's page, edit national pages, and manage any club's committee accounts. (Phase 1b extends this to member records; Phase 2 to member accounts.)

#### Scenario: NZKF steps in for a club without a secretary
- **WHEN** a club loses its committee members
- **THEN** any NZKF committee member can edit that club's page and manage its committee accounts

### Requirement: Club committee access is scoped to their own club
Club committee members SHALL be able to edit only their own club's page and manage only their own club's committee accounts. They SHALL NOT be able to edit another club's page or manage another club's committee accounts, nor edit national pages.

#### Scenario: Auckland committee cannot edit Wellington
- **WHEN** an Auckland club committee member attempts to edit Wellington's club page
- **THEN** the system denies the action

#### Scenario: Club committee members co-manage their own club
- **WHEN** any Auckland club committee member adds or removes another Auckland committee account
- **THEN** the system permits it (club officers police each other; no separate club-admin sub-role)

### Requirement: The member role is reserved but inert in Phase 1a
The system SHALL support assigning the `member` role, but a `member` SHALL have no editing capabilities until Phase 2 activates member-facing surfaces. A `member` SHALL never have club-page, national-page, or committee capabilities.

#### Scenario: Member role carries no committee capability
- **WHEN** a person holds only the member role
- **THEN** they have no club-page, national-page, or committee editing capability
