## ADDED Requirements

### Requirement: Public site served as static files from MyHost
The public site SHALL be served as static HTML by Apache on MyHost (cPanel, PHP + MySQL). The public site SHALL have no runtime dependency on the editing application or its database.

#### Scenario: Public page load requires no application
- **WHEN** a visitor requests a public page
- **THEN** Apache serves a pre-rendered static file without invoking the editing application or querying member data

### Requirement: DNS and SSL consolidated on MyHost
The `kyudo.nz` domain's DNS and SSL SHALL be managed on MyHost alongside the existing mail hosting, replacing the current split where site DNS is on Netlify and mail is on MyHost. Mail delivery SHALL remain uninterrupted through the migration.

#### Scenario: DNS is managed in one place
- **WHEN** an administrator needs to change DNS for kyudo.nz
- **THEN** they manage it in MyHost, where mail DNS/SSL already lives

#### Scenario: Mail continues working through cutover
- **WHEN** DNS/SSL is migrated to MyHost
- **THEN** sending and receiving mail for kyudo.nz addresses continues without interruption

### Requirement: Netlify retired for the public site
After migration, the public site SHALL be served from MyHost and SHALL no longer depend on Netlify. Existing behaviours — `.htaccess` URL rewrites and Fathom analytics — SHALL be preserved.

#### Scenario: Site runs on one host
- **WHEN** the migration is complete
- **THEN** the public site is served entirely from MyHost with URL rewrites and analytics still working

### Requirement: Deploy pipeline
Content edits SHALL publish by rendering to the docroot on save, with no separate deploy step. Code and template changes SHALL deploy via git push to cPanel Git Version Control.

#### Scenario: Content publishes on save
- **WHEN** a committee member saves a content edit
- **THEN** the affected static pages are rendered to the docroot immediately, with no manual upload

#### Scenario: Code changes deploy on push
- **WHEN** the maintainer pushes template or application changes to the repository
- **THEN** cPanel Git Version Control deploys them, preserving a git-push-to-deploy workflow without Netlify
