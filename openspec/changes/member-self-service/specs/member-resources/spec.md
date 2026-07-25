## ADDED Requirements

### Requirement: Members-only resources page
The system SHALL provide a resources page, visible only to authenticated members and committee, that presents the members-only Google Drive link (and optionally a curated index of its contents). The page SHALL NOT be publicly accessible.

#### Scenario: Authenticated member views resources
- **WHEN** a logged-in member opens the resources page
- **THEN** they see the Drive link (and any curated index)

#### Scenario: Public visitor is denied
- **WHEN** an unauthenticated visitor requests the resources page
- **THEN** the system does not serve the page or the link, and prompts login

### Requirement: Gate the link, not the files
The resources page SHALL gate discoverability of the Drive link behind login while the underlying Drive files remain accessible to anyone with the link (so members not on Google can still open them).

#### Scenario: Non-Google member opens a resource
- **WHEN** a logged-in member without a Google account follows the Drive link
- **THEN** they can open the resource because the files remain link-accessible, while the link itself is only obtainable behind login
