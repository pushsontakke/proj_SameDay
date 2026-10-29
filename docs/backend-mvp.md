# SAME DAY. — Backend MVP Plan

## Recommended stack: Supabase

Use Supabase (PostgreSQL database and hosted API) with the existing static HTML page. It provides shared data without needing to build and host a separate server.

## Minimum implementation

- Create an `entries` table: `id`, `name`, `month`, `day`, optional `birth_year`, `avatar`, `created_at`.
- On page load, fetch entries from the database and use them for the calendar, people list, and stats.
- On submission, validate and save the entry to the database. Enforce validation and duplicate rules on the server/database too.
- Use Supabase Row Level Security to control database access.
- Allow removal only to the entry's owner (for example, with sign-in or a secure deletion token). The current browser-only “own entry” flag cannot safely authorize deletion.
- Remove the generated sample entries when showing live data. Add server-side rate limiting if spam becomes a problem.

## Important for users

Names and birthdays are publicly visible on the page. Tell people this before they submit. Avoid collecting birth year unless it is needed; it is optional in the current design.

## MVP result

A shared calendar where visitors can see entries, add one, and securely remove their own. The existing page can remain mostly as-is; replace its local demo storage with Supabase reads and writes.
