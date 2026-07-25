## ADDED Requirements

### Requirement: Financial status is distinct from membership status
The system SHALL model membership status (`current` or `ceased`, with a cessation date) separately from financial status (whether the member has paid for a given year). A member who has not paid SHALL be treated as unfinancial and SHALL remain a member; non-payment alone SHALL NOT cease their membership.

#### Scenario: Unpaid member is unfinancial, not ceased
- **WHEN** a member does not pay for the year
- **THEN** the system records them as unfinancial while their membership status remains `current`

#### Scenario: Cessation is an explicit event
- **WHEN** a member resigns, dies, or is ceased by Committee resolution
- **THEN** the system records a membership status of `ceased` with a cessation date, independent of any payment record

### Requirement: Per-year financial and payment tracking
The system SHALL record, per member per year, whether they are financial and the date payment was recorded. This SHALL preserve history across years, including members who lapse (become unfinancial) and later become financial again.

#### Scenario: Marking a member financial for the year
- **WHEN** a club committee member marks one of their own club's members financial for the current year
- **THEN** the system records that member as financial for that year with the payment date

#### Scenario: Reporting financial and grade counts for a year
- **WHEN** the NZKF secretary needs the number of financial members (or members in a given grade) for a year
- **THEN** the system can report it from the per-year record

### Requirement: December fee cycle support
The system SHALL support the annual December cycle (balance date 31 December) in which each club's committee marks their own club's members financial for the year, and each club's totals are available for the NZKF treasurer to reconcile against the club's bulk bank transfer. Marking a member financial relies on the club committee's own confirmation; per-member payment-confirmation emails are out of scope (Phase 2).

#### Scenario: Club marks its members and treasurer reconciles
- **WHEN** an Auckland committee member marks their members financial for the year
- **THEN** Auckland's financial count/total is available so the NZKF treasurer can confirm receipt of Auckland's bulk transfer

### Requirement: Cessation reduces the record to name and cessation date
When a member ceases, the system SHALL record the cessation date and reduce the record to name and cessation date, removing contact details and all admin fields including grades (shogo, dan, kyu). Per-year financial history MAY be retained for reporting but personal contact and grade data SHALL be removed.

#### Scenario: A member ceases
- **WHEN** a member is recorded as ceased
- **THEN** their contact details and grade fields are removed and the record retains name and cessation date

#### Scenario: A ceased member rejoins
- **WHEN** a previously ceased member rejoins
- **THEN** they are re-admitted as a new/returning member and are responsible for re-supplying their details and grades, which were not retained

### Requirement: Seven-year retention purge
The system SHALL automatically delete a former-member record once more than seven years have passed since its cessation date. The purge SHALL key on the explicit cessation date and SHALL NOT affect current or rejoined members.

#### Scenario: Former record ages past seven years
- **WHEN** a former member's cessation date is more than seven years in the past
- **THEN** the scheduled purge deletes the remaining former-member record

#### Scenario: Rejoined member is not purged
- **WHEN** a member ceased more than seven years ago but has since rejoined and is current
- **THEN** the purge does not delete them, because their membership status is `current`

### Requirement: Self-serve register export
The system SHALL let the NZKF secretary or treasurer export, without developer involvement, the current statutory register together with the seven-year former-member trailing record (name and cessation date). The export SHALL be usable to satisfy a produce-on-request obligation.

#### Scenario: Secretary produces the register on request
- **WHEN** the NZKF secretary triggers the register export
- **THEN** the system produces the current statutory register plus the seven-year former-member record in a downloadable file, with no SSH, mysqldump, or developer help required

### Requirement: Member data archival separate from public content
Member data SHALL be stored in MySQL with a scheduled dump committed to a private repository. Member data SHALL NOT be written to the public content repository nor rendered into any public page.

#### Scenario: Nightly private archive
- **WHEN** the scheduled archive runs
- **THEN** the current member data is dumped to a private repository that is never published

#### Scenario: Member data never reaches the public site
- **WHEN** the public site is rendered
- **THEN** the render path has no access to the members table and no member personal data appears in public output
