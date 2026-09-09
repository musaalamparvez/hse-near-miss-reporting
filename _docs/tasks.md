# Near-Miss Reporting Tool — Backlog

Stack: Django + Postgres monolith (see `_docs/plan.md` for full product spec).
Each task below is scoped to be completable in one sitting and includes enough
context to pick up without reading the others.

## 1. Project setup with a passing test
Goal: Get an empty Django project running with a green test suite.
Description: Initialize a new Django project and app structure, configure Postgres as the database backend (local dev config, e.g. via `.env` or `docker-compose`), and add one trivial test (e.g. a health-check view or `assert True`) that passes under `manage.py test` or `pytest`. Include a README section on how to run the dev server and tests.

## 2. Core data models: Site and Assignee
Goal: Create the `Site` and `Assignee` Django models.
Description: `Site` needs `id`, `name`. `Assignee` needs `id`, `name`, `email`, and a many-to-many or foreign key relationship to `Site` (an assignee can belong to one or more sites). Write and run the migrations, and add basic `__str__` methods for readability in the admin.

## 3. Core data models: Report and Closure
Goal: Create the `Report` and `Closure` Django models.
Description: `Report` needs `id`, `site` (FK to Site), `category` (fixed choices list + "Other" with a free-text detail field), `description`, `reporter_name` (nullable), `is_anonymous` (bool), `photo` (optional file field), `location` (free-text; GPS lat/lng fields can be added as optional numeric fields), `assignee` (FK to Assignee), `status` (choices: Open / In Progress / Closed), `created_at`, `closed_at`. `Closure` needs `report` (FK or one-to-one), `note` (required text), `photo` (required file field), `closed_by`, `closed_at`. Write and run migrations.

## 4. Django Admin registration for Site and Assignee management
Goal: Let staff manage the site list and assignee list without a custom UI.
Description: Register `Site` and `Assignee` models in Django Admin with list displays showing name/email/linked sites, and make the assignee's site relationship editable inline. This is the answer to "who maintains the site/assignee lists" — no separate admin screen is needed for MVP.

## 5. Public report submission form — site, name, anonymity
Goal: Build the first section of the mobile report form.
Description: Create a Django view + template for a mobile-friendly form where a worker selects their `Site` from a dropdown, sees a `reporter_name` field pre-filled (if a name is available, e.g. from a query param or session), and can check an "anonymous" box that clears/disables the name field before submission. No login is required to access this page.

## 6. Report form — category and description fields
Goal: Add category selection and description to the report form.
Description: Add a `category` field to the report form using a fixed dropdown list (e.g. Slip/Trip/Fall, Equipment, Chemical, Electrical, Other) where selecting "Other" reveals a free-text detail field. Add a required `description` textarea. This task only covers the form fields and client-side show/hide behavior for "Other" — not the final submit handling.

## 7. Report form — photo upload
Goal: Allow an optional photo to be attached to a new report.
Description: Add a file input to the report form accepting a single image (jpg/png), store it against the `Report.photo` field on submission, and add basic client-side validation for file type and a reasonable size cap (e.g. 10MB). Local filesystem storage is fine for this task; cloud storage is a separate task.

## 8. Report form — location capture
Goal: Let the reporter specify where the hazard is.
Description: Add a location input to the report form that supports either browser GPS capture (via the Geolocation API, storing lat/lng) or manual free-text entry (e.g. "Corridor B, near loading dock") as a fallback when GPS is unavailable or denied. Only one of the two needs to be filled in for the form to be valid.

## 9. Report form — assignee dropdown filtered by site
Goal: Let the reporter pick who the report gets assigned to.
Description: Add an `assignee` dropdown to the report form that dynamically filters to only show Assignees linked to the currently selected Site (via an AJAX call or a small JS fetch keyed off the site dropdown's `onchange`). Assume the `Assignee` model already has a site relationship (see task 2).

## 10. Report submission handling
Goal: Wire up the full report form to actually save a Report.
Description: Implement the POST handler for the report form built in tasks 5–9: validate all fields, create a `Report` row with `status="Open"` and `created_at=now()`, and redirect to a simple "thanks, your report was submitted" confirmation page. This task assumes the form UI already exists and focuses only on server-side save logic.

## 11. Email notification to assignee on new report
Goal: Notify the assignee by email when a report is created.
Description: On successful report creation, send an email to the assigned `Assignee`'s email address containing a short summary of the report (site, category, description snippet) and a link to the report's detail page. Use Django's email backend; a real transactional provider (e.g. SendGrid/SES/Postmark) can be configured via settings, with console/log backend for local dev.

## 12. Assignee report detail page with status updates
Goal: Let an assignee view a report and move it through statuses.
Description: Build a view (reachable via the link sent in the notification email, no login required — access via the report's unique URL/ID) showing the report's details, with controls to change `status` from Open → In Progress → Closed. Moving to "Closed" should not be allowed directly from this page — that's handled by the closure form in task 13.

## 13. Closure form — note and photo required
Goal: Require documentation before a report can be marked Closed.
Description: Build a form (linked from the report detail page) that requires a `note` (text) and a `photo` (file upload) before creating a `Closure` record and setting the parent `Report.status = "Closed"` and `closed_at = now()`. Both fields are mandatory — the form should not submit without them.

## 14. Access-code gating for dashboard and assignee links
Goal: Restrict dashboard/report access without building full user auth.
Description: Implement a lightweight access-code check (e.g. a middleware or view decorator) that requires a valid code in the URL, a form field, or a session cookie before granting access to the supervisor dashboard and/or assignee report links. Store the valid code(s) in settings/env vars for now — no user accounts or password hashing needed.

## 15. Supervisor/safety officer dashboard — list and filters
Goal: Give supervisors a live view of all reports.
Description: Build a dashboard view (gated by the access-code check from task 14) listing all `Report` rows with site, category, status, and created date, with dropdown filters for `site` and `status`. Sort by most recent first by default.

## 16. Cloud object storage for photo uploads
Goal: Move photo storage off the local filesystem to a durable backend.
Description: Configure Django's file storage backend to use an S3-compatible bucket (e.g. AWS S3 or Cloudflare R2) for `Report.photo` and `Closure.photo` fields, using environment variables for credentials/bucket name. Verify uploads and retrieval work end-to-end in a non-local environment.

## 17. Transactional email provider integration
Goal: Replace the dev email backend with a real delivery provider.
Description: Configure Django to send outgoing mail (from task 11) through a transactional email provider (e.g. SendGrid, AWS SES, or Postmark), using API keys from environment variables. Send a real test email to confirm delivery and check spam/deliverability basics (from-address, SPF/DKIM notes for the domain owner).

## 18. QR code generation for report form access
Goal: Produce a scannable entry point to the report form.
Description: Generate a QR code (can be a one-off script or a small admin/management command) that encodes the URL to the report submission form, optionally per-site if each site should have its own QR code pointing to a pre-selected site. Output as a downloadable/printable image.

## 19. Production deployment configuration
Goal: Get the app deployable to a production environment.
Description: Add production-ready Django settings (DEBUG=False, ALLOWED_HOSTS, secret key from env, static file serving via whitenoise or equivalent), a Postgres connection string via env var, and deployment config for the target host (e.g. Dockerfile + Procfile, or platform-specific config). Document the deploy steps in the README.
