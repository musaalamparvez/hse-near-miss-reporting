# Near-Miss Reporting Tool — MVP Scope

## Problem Statement
Workers on multi-site worksites regularly witness near-misses (situations that could have caused injury but didn't) but have no fast, low-friction way to report them. Without a simple capture-and-follow-up mechanism, hazards go unaddressed until an actual incident occurs. Supervisors and safety officers also lack a shared view of open hazards and who is accountable for fixing them.

## Goals
1. Reduce the time from spotting a hazard to logging it to under 2 minutes.
2. Ensure every reported near-miss is assigned to a specific person for follow-up.
3. Give safety officers and supervisors a live view of open, in-progress, and closed actions across all sites.
4. Increase reporting volume by removing identity friction (anonymous option) while still defaulting to accountability (name pre-filled).

## Non-Goals (MVP)
- **User login/authentication** — using a shared link/access code instead; full auth is a fast-follow once adoption is proven.
- **SMS notifications** — email only for v1; SMS/escalation is a v2 addition.
- **Analytics/trend dashboard** — out of scope until there's enough report volume to analyze.
- **Auto-assignment by category/location** — reporter manually picks assignee for MVP; smart routing can come later.
- **Kiosk/offline mode** — assumes reporter has a smartphone and signal/wifi at time of report.

## User Roles
| Role | Access |
|---|---|
| **Reporter** | Any worker; scans QR code, submits report, no login needed |
| **Assignee** | Selected from a site-filtered list; receives email, closes out action |
| **Safety Officer / Supervisor** | Views dashboard of all reports/statuses via shared link/access code |

## Core Data Model
**Report**
- id, site_id, category (fixed list or "Other" + free text), description
- reporter_name (nullable if anonymous), is_anonymous (bool)
- photo (optional), location (GPS pin OR free-text location)
- assignee_id, status (Open / In Progress / Closed)
- created_at, closed_at

**Site**
- id, name

**Assignee**
- id, name, email, site_id(s)

**Closure**
- note (required), photo (required), closed_by, closed_at

## Core Flows

**1. Submit a report**
1. Worker scans QR code → opens mobile web form
2. Selects site from dropdown
3. Name auto-filled (editable); checkbox to submit anonymously
4. Selects category from fixed list (or "Other" + free text)
5. Adds description, optional photo, and location (GPS pin or typed, e.g. "Corridor B, near loading dock")
6. Picks assignee from list (filtered by selected site)
7. Submits → status set to **Open**

**2. Notify assignee**
- Assignee receives an email with report summary and a link to the report

**3. Close out the action**
- Assignee opens report link, updates status: Open → In Progress → Closed
- On Closed, must add a note and a photo confirming the fix

**4. Dashboard view**
- Safety officers/supervisors access via shared link/access code
- View all reports, filterable by site and status

## Requirements

### Must-Have (P0)
- [ ] Mobile-friendly report form, accessible via QR code
- [ ] Site dropdown selector (multi-site support)
- [ ] Name field pre-filled with anonymous checkbox override
- [ ] Fixed category list + "Other" with free-text detail
- [ ] Photo upload
- [ ] Location field: GPS pin capture OR free-text entry
- [ ] Assignee dropdown, filtered by selected site
- [ ] Email notification to assignee on new report
- [ ] Status flow: Open → In Progress → Closed
- [ ] Closure requires note + photo
- [ ] Dashboard (shared link/access code) showing all reports, filterable by site + status

### Nice-to-Have (P1)
- Overdue reminders (e.g. auto re-notify if "Open" > 3 days)
- CSV export of reports from dashboard
- Basic search on dashboard (by keyword, date range)

### Future Considerations (P2)
- Full user authentication with role-based permissions
- SMS notifications / escalation chains
- Trend analytics dashboard (category frequency, site comparisons, time-to-close metrics)
- Auto-assignment by category or location
- Offline-capable submission (queues and syncs when back online)

## Open Questions
- Who maintains the site list and assignee list (admin panel needed, or manual backend setup for MVP)? — *engineering/stakeholder*
- Is there a cap on photo size/number per report? — *engineering*
- Should the "shared access code" be one code for everyone, or per-role (one for supervisors, one for safety officers)? — *stakeholder*
- Retention policy: how long are closed reports kept? — *stakeholder/legal*

## Timeline Considerations
- No hard deadline specified — suggest treating this as a single-phase MVP build, then reassessing P1/P2 items based on real usage after 4–6 weeks live.
