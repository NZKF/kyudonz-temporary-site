## 1. Foundation & hosting

- [ ] 1.1 Provision the PHP app + MySQL database on MyHost under a non-public path/subdomain
- [ ] 1.2 Set up the app's git repository and cPanel Git Version Control with `.cpanel.yml` for code deploys
- [ ] 1.3 Configure `utf8mb4` on the database and all tables (Japanese text support)
- [ ] 1.4 Run a magic-link deliverability test to Gmail/iCloud; decide MyHost SMTP (SPF/DKIM via cPanel Email Deliverability) vs a transactional provider free tier

## 2. Authentication

- [ ] 2.1 Create accounts/sessions/login-tokens schema (no password fields)
- [ ] 2.2 Implement magic-link request → send → verify flow with single-use, time-limited tokens
- [ ] 2.3 Implement neutral "check your email" response for unknown addresses (no account enumeration)
- [ ] 2.4 Implement ~1-year session lifetime and session expiry handling
- [ ] 2.5 Implement account email change (verified) so the email remains the identity

## 3. Authorization

- [ ] 3.1 Create roles + role-assignment schema (member, club committee scoped to club, NZKF committee)
- [ ] 3.2 Implement scope checks: NZKF = all; club committee = own club; member = self
- [ ] 3.3 Build the web UI for assigning/removing roles (no SSH/git/deploy required)
- [ ] 3.4 Enforce that NZKF committee can act on any club; club committee only on their own

## 4. Club content editing

- [ ] 4.1 Define the per-club structured data model (venue, map, president, contact, schedule, fees, trial dates/price, beginner dates/price, prose)
- [ ] 4.2 Migrate `locations.html` into four per-club data units
- [ ] 4.3 Build the shared layout (head, nav, analytics) and a `render()` for all pages
- [ ] 4.4 Render every existing page via the layout on MyHost; diff against the live Netlify site
- [ ] 4.5 Build the committee-facing club-page edit form (fields + prose), scoped by role
- [ ] 4.6 Implement render-to-docroot on save
- [ ] 4.7 Commit rendered content + source data to the (public) content git repo for rollback

## 5. DNS/SSL migration & cutover

- [ ] 5.1 Prepare kyudo.nz DNS records on MyHost; verify mail records first
- [ ] 5.2 Cut DNS/SSL over to MyHost; confirm mail send/receive before and after
- [ ] 5.3 Point public kyudo.nz at the MyHost-served static output; verify `.htaccess` rewrites and Fathom analytics
- [ ] 5.4 Retire Netlify for the public site

## 6. Go-live & handover

- [ ] 6.1 Create the ~15 committee accounts and assign roles
- [ ] 6.2 Verify each club committee can edit only their own club page; NZKF committee can edit any
- [ ] 6.3 Write a short non-technical admin guide (assign roles, edit a club page, deploy)
