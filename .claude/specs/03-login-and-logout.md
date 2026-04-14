# Spec: Login and Logout

## Overview
Implement working login and logout so existing Spendly users can authenticate
and end their session. The `/login` route already renders the form via GET but
the POST handler is missing. The `/logout` route is a stub. This step wires up
credential verification using `werkzeug`, starts a Flask session on success,
and clears it on logout. It also adds route guards so unauthenticated users
cannot access the dashboard, and updates the navbar to reflect the logged-in
state — replacing "Sign in / Get started" with the user's name and a logout
link.

## Depends on
- Step 01 — Database Setup (users table must exist)
- Step 02 — Registration (session pattern, `get_user_by_email()`, and dashboard stub)

## Routes
- `GET  /login` — render the login form — public
- `POST /login` — verify credentials, start session, redirect to dashboard — public
- `GET  /logout` — clear session, redirect to landing page — logged-in

## Database changes
No new tables or columns. No new helper functions needed — `get_user_by_email(email)`
already exists in `database/db.py` and returns the full users row (including
`password_hash`). Use `werkzeug.security.check_password_hash` in `app.py`
to verify the supplied password against the stored hash.

## Templates
- **Modify:** `templates/login.html` — ensure the `{{ error }}` block is
  rendered (it is already present); repopulate the `email` field on validation
  failure using `value="{{ email }}"` so the user does not retype it
- **Modify:** `templates/base.html` — make the navbar conditional:
  - When `session.user_id` is **not** set: show "Sign in" and "Get started" links (current behaviour)
  - When `session.user_id` **is** set: show the user's name (non-linked) and a "Sign out" link pointing to `/logout`
- **Modify:** `templates/dashboard.html` — add a login guard: if
  `session.user_id` is not set, redirect to `/login`

## Files to change
- `app.py` — implement `POST /login` handler; implement `GET /logout`
- `templates/login.html` — repopulate email field on validation failure
- `templates/base.html` — conditional navbar (guest vs. logged-in state)
- `templates/dashboard.html` — login guard redirect

## Files to create
No new files.

## New dependencies
No new dependencies. Uses:
- `werkzeug.security.check_password_hash` (already installed)
- `flask.session` (built into Flask)

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — never use string formatting in SQL
- Passwords verified with `werkzeug.security.check_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Session must store at minimum: `session["user_id"]` and `session["user_name"]`
- Login validation rules:
  - Email: required, non-empty after strip
  - Password: required, non-empty
  - Wrong email or wrong password: show a single generic error "Invalid email or password." — do not reveal which field is wrong
- On validation failure: re-render `login.html` with the error and repopulate
  the email field; never repopulate the password field
- Logout must call `session.clear()` then redirect to `url_for("landing")`
- Dashboard guard must use `redirect(url_for("login"))` — not a 401 response
- Do not change any other placeholder routes (profile, add_expense, etc.)

## Definition of done
- [ ] `GET /login` renders the login form
- [ ] Submitting valid credentials sets `session["user_id"]` and `session["user_name"]` and redirects to `/dashboard`
- [ ] The dashboard displays "Welcome, <name>!" for the logged-in user
- [ ] Submitting an email that does not exist shows "Invalid email or password." and re-renders the form
- [ ] Submitting a correct email with a wrong password shows "Invalid email or password." and re-renders the form
- [ ] The email field is repopulated after a failed login; the password field is empty
- [ ] `GET /logout` clears the session and redirects to the landing page
- [ ] After logout, visiting `/dashboard` redirects to `/login`
- [ ] The navbar shows "Sign in" and "Get started" when no session exists
- [ ] The navbar shows the user's name and a "Sign out" link when a session exists
- [ ] The demo seed user (`demo@spendly.com` / `demo123`) can log in successfully
- [ ] App starts without errors
