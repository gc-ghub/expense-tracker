# Spec: Registration

## Overview
This step makes account creation actually work. `GET /register` already renders `register.html` with a form that posts to `/register`, but there is no handler for the submission — the form currently has nowhere to go. This feature adds the `POST /register` handler that validates input, hashes the password, inserts a new row into `users`, and sends the user on to `/login`. It builds directly on the SQLite data layer from Step 1 and is a prerequisite for any future login/session work.

## Depends on
- Step 01 (Database Setup) — requires `get_db()`, `init_db()`, and the `users` table with a `UNIQUE` constraint on `email`.

## Routes
- `POST /register` — validate and create a new user account, then redirect to `/login` — public

`GET /register` already exists and is unchanged, aside from being able to re-render with an `error` message on validation failure.

## Database changes
No database changes. The `users` table (id, name, email, password_hash, created_at) already has everything registration needs, including the `UNIQUE` constraint on `email` that backs duplicate-email detection.

## Templates
- **Create:** none
- **Modify:** `templates/register.html` — no structural changes needed; it already renders `{{ error }}` via the `auth-error` block and posts to `/register`. Repopulate the `name`/`email` field values from the submitted form on validation failure so the user doesn't have to retype them.

## Files to change
- `app.py` — change the `/register` route to accept `GET` and `POST`; on `POST`, validate input and call the new `db.py` helper to create the user, then redirect to `url_for('login')` on success or re-render `register.html` with an `error` on failure
- `database/db.py` — add a function to insert a new user (hashing the password with `werkzeug.security.generate_password_hash`) and a function to check whether an email is already registered

## Files to create
None.

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug (`generate_password_hash`)
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- DB logic stays in `database/db.py`, never inline in the route
- Use `url_for()` for the redirect to `/login` — never hardcode the path
- Validate on the server even though the form has `required`/`type="email"` attributes client-side
- Re-render `register.html` with a clear `error` message instead of raising an unhandled exception on duplicate email or bad input
- Do not add session/login logic — that belongs to a later step

## Definition of done
- [ ] Submitting the register form with a new name/email/password creates a row in `users` with a hashed (not plaintext) password
- [ ] After successful registration, the browser is redirected to `/login`
- [ ] Submitting with an email that's already registered re-renders `register.html` with an error message and does not create a duplicate row
- [ ] Submitting with a missing name, email, or password re-renders `register.html` with an error message instead of a 500 error
- [ ] Submitting with an invalid email format is rejected with an error message
- [ ] Submitting with a password under 8 characters is rejected with an error message
- [ ] No plaintext passwords appear anywhere in the `users` table
- [ ] All new/changed queries in `database/db.py` use `?` parameterized placeholders
- [ ] App starts and runs without errors on port 5001
