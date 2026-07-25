## ADDED Requirements

### Requirement: Member-record scoping
Building on the three-role model from the content-editing-and-hosting change, the system SHALL scope member-record access by role. Club committee members SHALL be able to create, view, and edit member records only for their own club. NZKF committee members SHALL be able to do so for any club.

#### Scenario: Club committee manages only its own club's members
- **WHEN** an Auckland club committee member creates or edits a member record
- **THEN** the system permits it only for Auckland members and denies access to other clubs' members

#### Scenario: NZKF committee manages any club's members
- **WHEN** an NZKF committee member manages member records for any club
- **THEN** the system permits it, including marking those members financial for the year

### Requirement: Treasurer reconciliation needs only name, club, and financial status
The NZKF treasurer's reconciliation task SHALL be satisfiable using only member name, club, and financial status. The system SHALL NOT require exposing telephone, grades, or other personal fields to complete reconciliation.

#### Scenario: Reconciliation uses minimal fields
- **WHEN** the NZKF treasurer reconciles a club's bulk payment
- **THEN** the per-club financial totals and member name/club/financial-status are sufficient, without requiring other personal fields
