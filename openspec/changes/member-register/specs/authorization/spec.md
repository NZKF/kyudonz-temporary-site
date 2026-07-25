## ADDED Requirements

### Requirement: Member-record scoping
Building on the three-role model from the content-editing-and-hosting change, the system SHALL scope member-record access by role. Club committee members SHALL be able to create, view, and edit member records only for their own club. NZKF committee members SHALL be able to do so for any club.

#### Scenario: Club committee manages only its own club's members
- **WHEN** an Auckland club committee member creates or edits a member record
- **THEN** the system permits it only for Auckland members and denies access to other clubs' members

#### Scenario: NZKF committee manages any club's members
- **WHEN** an NZKF committee member manages member records for any club
- **THEN** the system permits it, including marking those members financial for the year

### Requirement: Member writes are confined to an allow-list of fields per role
When a member record is created or edited, the system SHALL accept only an explicit allow-list of writable fields, ignoring any other submitted parameters (mass assignment). Fields that reassign a member to another club or set the consent-held flag SHALL be writable only within the actor's scope, so that a crafted request cannot move a member out of the actor's club or forge consent. State-changing member requests SHALL be protected against cross-site request forgery.

#### Scenario: Crafted parameter cannot reassign a member's club
- **WHEN** a club committee member submits an edit whose parameters include a different club than their own
- **THEN** the system ignores the out-of-scope club change and the member remains in the actor's club

### Requirement: Access to a single server-side authorization choke-point
All member-record reads and writes SHALL pass through a single server-side authorization decision that applies role scope and field-level visibility; authorization SHALL NOT rely on the UI hiding actions or fields. Access decisions SHALL be enforced on the server for every request, including direct requests that reference a record by identifier.

#### Scenario: Changing an identifier in a request does not bypass scope
- **WHEN** an Auckland committee member issues a direct request for a Wellington member's record by its identifier
- **THEN** the server-side authorization denies it, regardless of what the UI would have shown

### Requirement: Treasurer reconciliation needs only name, club, and financial status
The NZKF treasurer's reconciliation task SHALL be satisfiable using only member name, club, and financial status. The system SHALL NOT require exposing telephone, grades, or other personal fields to complete reconciliation.

#### Scenario: Reconciliation uses minimal fields
- **WHEN** the NZKF treasurer reconciles a club's bulk payment
- **THEN** the per-club financial totals and member name/club/financial-status are sufficient, without requiring other personal fields
