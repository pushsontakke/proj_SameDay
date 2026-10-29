# SAME DAY. — Supabase Setup (MVP)

These steps create a shared database for the existing single-file HTML page. The browser can read and add entries. For this first version, remove unwanted entries from the Supabase dashboard; visitors cannot delete rows. No login or separate server is required.

## 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com/) and sign in or create an account.
2. Create a new project. Choose an organization, enter a project name, set a strong database password, and choose a region near most visitors.
3. Wait for the project to finish provisioning.

## 2. Create the table and access rules

1. Open the project’s **SQL Editor** and choose **New query**.
2. Paste the complete SQL below and click **Run**.
3. Keep this table public-read and public-add only. The SQL does not allow public updates or deletions.

```sql
create table public.entries (
  id bigint generated always as identity primary key,
  name text not null check (char_length(trim(name)) between 2 and 30),
  month smallint not null check (month between 0 and 11),
  day smallint not null check (
    day between 1 and (array[31,29,31,30,31,30,31,31,30,31,30,31])[month + 1]
  ),
  birth_year smallint,
  avatar text not null default 'neutral'
    check (avatar in ('neutral','baby','teen','adult','elder')),
  created_at timestamptz not null default now(),
  check (birth_year is null or birth_year between 1900 and 2100),
  check (
    birth_year is null or month <> 1 or day <> 29 or
    (birth_year % 4 = 0 and (birth_year % 100 <> 0 or birth_year % 400 = 0))
  )
);

create unique index entries_no_duplicate
  on public.entries (month, day, lower(name), coalesce(birth_year, 0));

alter table public.entries enable row level security;
grant usage on schema public to anon;
grant select, insert on public.entries to anon;

create policy "Anyone can read entries"
  on public.entries for select to anon using (true);

create policy "Anyone can add entries"
  on public.entries for insert to anon with check (true);
```

The app numbers months `0` through `11`, matching JavaScript’s `Date.getMonth()`. A missing birth year is stored as SQL `NULL`. The unique index blocks duplicate name/date/year entries, including entries with no year.

## 3. Copy the browser connection details

1. In the Supabase dashboard, open the project’s **Connect** dialog, or go to **Project Settings → API Keys**.
2. Copy the project URL and the **publishable key** (starts with `sb_publishable_`). Older projects may show an `anon` key; it is the low-privilege browser key.
3. Put these values in the HTML where the client is initialized:

```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
  const sbClient = window.supabase.createClient(
    'https://YOUR_PROJECT_REF.supabase.co',
    'YOUR_PUBLISHABLE_KEY'
  );
</script>
```

Replace both placeholders with your project’s values. Use the publishable/anon key only. **Never put a secret or `service_role` key in HTML, a public file, or a message.** The browser key is visible by design; Row Level Security is what limits its access.

## 4. Connect the page to the table

In the page’s existing JavaScript:

- Replace `makeDemo()` and the `localStorage` user-entry loading with an initial database read:

```js
const { data, error } = await sbClient
  .from('entries')
  .select('id,name,month,day,birth_year,avatar');
if (error) throw error;
```

- Convert each returned row to the page’s current shape: `m = month`, `d = day`, `year = birth_year`, `av = avatar`. Keep sample/demo entries out of the live calendar.
- In the form submit handler, replace the simulated delay and local save with an insert:

```js
const { data, error } = await sbClient
  .from('entries')
  .insert({
    name: v.name,
    month: v.month,
    day: v.day,
    birth_year: v.year,
    avatar: avChoice
  })
  .select('id,name,month,day,birth_year,avatar')
  .single();
if (error) throw error;
```

  Add the returned row to the page state and refresh the calendar. If the unique index rejects a duplicate, show a friendly “already on the sheet” message.
- Remove browser-only `localStorage` logic for saved entries and deletion. To remove an entry in this MVP, open **Table Editor → entries** in Supabase and delete its row.
- Keep the page’s public-visibility consent message. Consider removing the optional birth year if it is not needed.

## 5. Open the page through a local web server

Do not test by opening the HTML as a `file://` URL. In a terminal, change directory to the folder containing the HTML and run:

```sh
python3 -m http.server 8000
```

Open `http://localhost:8000/proj_Same_Day.html` (use the actual HTML filename). Add one test entry, reload, and check that it remains. Confirm that a duplicate is rejected and that the entry appears in the Supabase **Table Editor**.

## Before sharing or publishing

- Do not share the database password or secret/service-role key. Share the guide and HTML only; your friend should create their own Supabase project and use their own project URL/key.
- The current public insert rule allows anyone to submit entries, so it can attract spam. Add server-side rate limiting or a CAPTCHA before a large public launch.
- Enable RLS before putting the publishable key in the page. Review policies before changing table permissions.

## Official references

- [Create and manage tables](https://supabase.com/docs/guides/database/tables)
- [Supabase JavaScript installation](https://supabase.com/docs/reference/javascript/installing)
- [Supabase API keys](https://supabase.com/docs/guides/getting-started/api-keys)
- [Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
