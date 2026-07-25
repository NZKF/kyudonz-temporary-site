## ADDED Requirements

### Requirement: Structured per-club content
Each club's page content SHALL be stored as structured data (venue, map link, president, contact email, schedule, fees, trial dates and price, beginner-course dates and price) plus an optional free-form prose section, one data unit per club. The four clubs SHALL NOT share a single content file.

#### Scenario: Editing one club leaves others untouched
- **WHEN** Auckland's content is changed
- **THEN** only Auckland's data unit changes and Wellington, Palmerston North, and Christchurch data are unaffected

### Requirement: Committee-facing edit form
The system SHALL present club committee members a form for editing their own club's structured fields and prose section. NZKF committee members SHALL be able to use the same form for any club.

#### Scenario: Club committee edits their own club's beginner dates
- **WHEN** an Auckland committee member updates the trial and beginner-course dates in the form and saves
- **THEN** the new dates are stored for Auckland without requiring any developer involvement

#### Scenario: Structured fields are edited as fields, not raw HTML
- **WHEN** a committee member edits fields such as fees or dates
- **THEN** they edit discrete fields rather than hand-editing HTML markup

### Requirement: Render structured data to static public HTML
On save, the system SHALL render the affected public page(s) to static HTML files in the Apache docroot. The public site SHALL be served as static files with no runtime dependency on the application.

#### Scenario: A save publishes to the live site
- **WHEN** a committee member saves a valid club-page edit
- **THEN** the corresponding static HTML in the docroot is regenerated and served to the public

#### Scenario: Application outage does not take down the public site
- **WHEN** the editing application is unavailable
- **THEN** the already-rendered public pages continue to be served by Apache

### Requirement: Render targets are system-derived, never user-controlled
The docroot file path each render writes to SHALL be derived by the system from a fixed allow-list of known pages, never from user-supplied input (club slug, page name, or field value). A save SHALL only ever overwrite the intended page's file and SHALL NOT be able to write outside the docroot or to an arbitrary path.

#### Scenario: Edited content cannot redirect the write target
- **WHEN** a committee member saves an edit whose field values contain path characters or traversal sequences
- **THEN** the system writes only to the fixed, system-derived file for that page and never to a path influenced by the submitted content

### Requirement: Edit forms accept only allow-listed fields
The edit form handler SHALL accept only an explicit allow-list of writable fields for the page being edited, ignoring any other submitted parameters, so that a crafted request cannot set fields the form does not expose (mass assignment). State-changing saves SHALL be protected against cross-site request forgery.

#### Scenario: Unexposed field cannot be set via a crafted request
- **WHEN** a request includes parameters beyond the fields the form exposes
- **THEN** the system ignores the extra parameters and persists only the allow-listed fields

### Requirement: Content changes are archived in git
Rendered content and its source data SHALL be committed to a git repository so that any change can be rolled back and attributed.

#### Scenario: Reverting a bad content edit
- **WHEN** a club page is changed incorrectly
- **THEN** an administrator can restore the prior version from git history

### Requirement: Shared layout across pages
Common page structure (head metadata, navigation, analytics) SHALL be defined once in a shared layout and applied to all rendered pages, replacing the current per-file duplication.

#### Scenario: Updating navigation once
- **WHEN** the shared navigation is changed
- **THEN** all rendered pages reflect the change without editing each page individually
