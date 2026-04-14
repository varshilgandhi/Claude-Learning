# Spec: Registration

## Overview
Implement working user registration so new visitors can create a Spendly account.
The `/register` route currently only handles GET. This step wires up the POST handler,
validates input, inserts the new user into the database with a hashed password, starts
a Flask session, and redirects the user to the dashboard placeholder. It is the first
step that touches authentication state and establishes the session pattern all future
auth steps will reuse.

## Depends on
- Step 01 — Database Setup (users table must exist)

## Routes
- `GET  /register` — render the registration form — public
- `POST /register` — validate input, create user, start session, redirect — public
- `GET  /dashboard` — placeholder landing page after login (new stub) — logged-in

## Database changes
No new tables or columns. One new helper function in `database/db.py`:

- `get_user_by_email(email)` — returns the users row matching the email or `None`
- `create_user(name, email, password)` — hashes the password with werkzeug and inserts
  a row into `users`; returns the new user `id`

Both must use parameterised queries.

## Templates
- **Modify:** `templates/register.html` — already has the form; add `{{ error }}` block
  display (already present) and ensure `value="{{ name }}"` and `value="{{ email }}"`
  are re-populated on validation failure so the user does not have to retype
- **Create:** `templates/dashboard.html` — minimal stub that extends `base.html`,
  shows "Welcome, {{ name }}!" and a note that the full dashboard is coming in a later step

## Files to change
- `app.py` — convert `/register` GET stub into a GET+POST route; add `secret_key`;
  add `/dashboard` stub route; import `session` from flask; import `create_user` and
  `get_user_by_email` from `database.db`
- `database/db.py` — add `get_user_by_email()` and `create_user()` functions
- `templates/register.html` — repopulate `name` and `email` fields on validation error

## Files to create
- `templates/dashboard.html` — post-login stub page

## New dependencies
No new dependencies. Uses:
- `flask.session` (built into Flask)
- `werkzeug.security.generate_password_hash` (already installed)

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — never use string formatting in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `app.secret_key` must be set before sessions work; use a hard-coded dev string for now
  (e.g. `app.secret_key = "dev-secret-change-in-prod"`)
- Session must store at minimum: `session["user_id"]` and `session["user_name"]`
- Redirect to `/dashboard` on successful registration using `redirect(url_for("dashboard"))`
- Validation rules to enforce:
  - Name: required, non-empty after strip
  - Email: required, non-empty after strip
  - Password: required, minimum 8 characters
  - Duplicate email: catch the sqlite3 `IntegrityError` and show a friendly error message
- On validation failure: re-render `register.html` with the error message and repopulate
  name and email fields (never repopulate the password field)
- Do not auto-login the demo seed user — registration is only for new sign-ups via the form

## Definition of done
- [ ] `GET /register` renders the registration form
- [ ] Submitting the form with valid data creates a row in the `users` table with a hashed password
- [ ] After successful registration the browser is redirected to `/dashboard`
- [ ] The dashboard page displays "Welcome, <name>!"
- [ ] Registering with an email that already exists shows an inline error and does not create a duplicate row
- [ ] Submitting with an empty name, email, or password shows a validation error and re-renders the form
- [ ] Submitting with a password shorter than 8 characters shows a validation error
- [ ] Name and email fields are repopulated after a validation error; password field is empty
- [ ] `session["user_id"]` and `session["user_name"]` are set after successful registration
- [ ] App starts without errors
