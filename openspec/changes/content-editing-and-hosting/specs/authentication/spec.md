## ADDED Requirements

### Requirement: Passwordless magic-link login
The system SHALL authenticate users by emailing a single-use, time-limited login link to the address the user entered. The system SHALL NOT require or store user passwords, and SHALL NOT depend on any third-party OAuth/SSO provider.

#### Scenario: Requesting a login link for a known account
- **WHEN** a person enters an email address that matches an active account
- **THEN** the system sends a one-time login link to that address and shows a neutral "check your email" message

#### Scenario: Requesting a login link for an unknown address
- **WHEN** a person enters an email address that matches no account
- **THEN** the system shows the same neutral "check your email" message and sends no link, so account existence is not disclosed

#### Scenario: Using a valid link
- **WHEN** a user clicks a login link that has not expired and has not been used
- **THEN** the system establishes an authenticated session for that account and marks the link consumed

#### Scenario: Using an expired or already-used link
- **WHEN** a user clicks a login link that has expired or was already used
- **THEN** the system refuses the login and offers to send a fresh link

### Requirement: Session lifetime tiered by blast radius
The system SHALL keep club-committee sessions valid for approximately one year so that infrequent users (typically 3–4 logins per year) rarely need to re-authenticate. NZKF-committee sessions — which carry federation-wide access to all member PII and every page — SHALL expire substantially sooner (approximately 30 days), so the higher-blast-radius accounts re-authenticate far more often. Where a person holds both roles, the shorter (NZKF) lifetime SHALL govern their session.

#### Scenario: Club committee returns within the session window
- **WHEN** an authenticated club-committee user returns before their ~1-year session expires
- **THEN** the system recognises them without requiring a new login link

#### Scenario: NZKF committee session expires sooner
- **WHEN** an NZKF-committee user's session passes its ~30-day lifetime
- **THEN** the system requires a fresh magic-link login before granting access, even though a club-only account would still be valid

#### Scenario: Session expiry
- **WHEN** a user's session has passed its lifetime
- **THEN** the system requires a fresh magic-link login before granting access to protected areas

### Requirement: Session integrity and revocation
Session cookies SHALL be marked HttpOnly, Secure, and SameSite, and the session identifier SHALL be rotated on login. The system SHALL keep server-side session state so that a session can be revoked, and SHALL revoke a user's active sessions when their committee role is removed.

#### Scenario: Losing a role kills active sessions
- **WHEN** an administrator removes a person's committee role
- **THEN** that person's existing sessions no longer grant the removed access, without waiting for the session to expire

### Requirement: Region-restricted application access
Where the host supports source-IP geolocation, the editing application (login and all committee/admin surfaces) SHOULD reject requests originating outside New Zealand, Australia, and Japan, with an administrator override for a locked-out user. This restriction SHALL apply only to the application; it SHALL NOT apply to the public static site, which SHALL remain reachable worldwide.

#### Scenario: Login attempt from an unexpected region
- **WHEN** a request to the editing application originates from outside NZ/AU/JP and the host supports geolocation
- **THEN** the system rejects it, while the public static site continues to load from anywhere

#### Scenario: Public site stays global
- **WHEN** a visitor anywhere in the world requests a public page
- **THEN** the static site is served normally, unaffected by the application's region restriction

### Requirement: Email is the account identity
The system SHALL treat the email address as the account identity. A user SHALL be able to change their own account email, and the change SHALL take effect for subsequent logins.

#### Scenario: Member chooses which email is their account
- **WHEN** an account is created for a member
- **THEN** the account is keyed to the email the member chose to give their club/NZKF, regardless of provider (Gmail, iCloud, @kyudo.nz, etc.)

#### Scenario: Changing account email
- **WHEN** a user updates their account email to a new verified address
- **THEN** future login links are sent to the new address and the old address no longer authenticates the account

### Requirement: Reliable delivery of login emails
The system SHALL send login emails through a path configured for authenticated delivery (SPF/DKIM aligned) so that links reliably reach major providers such as Gmail.

#### Scenario: Login email reaches an external mailbox
- **WHEN** the system sends a login link to a Gmail or iCloud address
- **THEN** the message is authenticated (SPF/DKIM) so it is delivered to the inbox rather than rejected or spam-filtered
