# Mosa's Baby Gift List

A one-page baby gift registry for baby Mosa. Guests tap the gift they are
bringing so nobody buys the same thing twice. Claimer names are hidden from
other guests and only shown in the passcode-protected **Family view**.

Claims are **shared across every guest's phone** through a free Supabase
database, so the "still needed / covered" list is the same for everyone.

| Path | What it is |
| --- | --- |
| `index.html` | The whole site — markup, styling and logic in one file. No frameworks, no build step. |
| `images/` | The illustrations, cropped from the original hand-made gift list. |
| `README.md` | This file. |

---

## 1. Set up the shared database (Supabase — free, ~10 minutes)

1. Go to **[supabase.com](https://supabase.com)** → sign in → **New project**.
   Pick any name, set a database password (you won't need it again), choose the
   region closest to you. Wait ~2 minutes for it to finish.
2. In the left sidebar open **SQL Editor** → **New query**, paste this in and
   click **Run**:

   ```sql
   create table public.claims (
     id          bigint generated always as identity primary key,
     item_id     text not null,
     guest_id    text not null,
     guest_name  text not null,
     note        text default '',
     created_at  timestamptz default now()
   );

   alter table public.claims enable row level security;

   create policy "anyone can read"   on public.claims for select using (true);
   create policy "anyone can add"    on public.claims for insert with check (true);
   create policy "anyone can remove" on public.claims for delete using (true);
   ```

3. In the left sidebar open **Project Settings** (the gear) → **API**. Copy:
   - **Project URL** — looks like `https://abcdefgh.supabase.co`
   - **Project API keys → `anon` `public`** — a long string starting `eyJ…`

4. Open `index.html`, find the `CFG` block near the top of the `<script>` and
   paste the two values in:

   ```js
   var CFG = {
     SUPABASE_URL:      "https://abcdefgh.supabase.co",
     SUPABASE_ANON_KEY: "eyJhbGciOi…the-long-anon-key…",
     ...
   ```

   Save. That's it — the page now reads and writes claims from Supabase.

> The `anon` key is **meant** to live in the page; it only allows what the
> policies above allow (read / add / remove rows in this one table). It is not
> a secret. Do not paste the `service_role` key — that one *is* secret.

> **Heads-up:** a free Supabase project is paused after ~1 week with no
> traffic. Set this up in the few days before the event, or just open the
> Supabase dashboard on the morning of and hit **Restore** if it's asleep.

If you leave `SUPABASE_URL` blank the page still works, but claims are only
saved on that one phone — fine for a quick preview, not for the real thing.

---

## 2. Edit the content

Everything is in the `CFG` block at the top of the `<script>` in `index.html`:

| Field | What it controls |
| --- | --- |
| `babyName` | Name in the title and footer |
| `tagline` | The line under the title |
| `contactName`, `contactPhone` | The "Ask the family" card |
| `dropOff` | The paragraph in the "Ask the family" card |
| `passcode` | Family-view gate. **It is visible in the page source** — it just keeps casual guests out of the names list, it is not real security. Currently `mosa2026`. |

**The gift list** is the `CATS` array just below `CFG`. Each category is
`{ name, band, ink, img, items }`. Each item is `["Item name"]`, or
`["Item name", 1]` to let several people bring it (nappies, clothing,
vouchers…).

**Date & venue** is the "Coming soon" card in the markup — replace it with the
real date, venue and an RSVP when they're confirmed.

---

## 3. Preview it locally

```bash
python -m http.server 8000
# then open http://localhost:8000
```

(Opening `index.html` by double-clicking won't load the images — a local
server is needed.)

---

## 4. Publish it (GitHub Pages — free)

The files are already in this repo. To go live:

1. Push any edits to the `main` branch.
2. Repo **Settings → Pages**: *Source* = **Deploy from a branch**,
   *Branch* = `main`, folder = `/ (root)`. **Save**.
3. Wait ~1 minute. The registry is live at
   **`https://muzisitsha.github.io/Mosa-registary/`** — that's the link to
   send out. The "Copy the registry link" button on the page copies it too.

Any later edit: commit, push, wait a minute, done.

---

## How it works

- Every claim is one row in the Supabase `claims` table
  (`item_id`, `guest_id`, `guest_name`, `note`).
- On load and every 6 seconds the page fetches all rows and re-renders, so a
  claim made on one phone shows up on the others within a few seconds.
- `guest_id` is a random id kept in that phone's `localStorage` — it's what
  lets a guest undo *their own* claim. Clearing browser data makes a phone
  "forget" which claims were its own (the claims themselves stay).
- Family view removals and "Copy a summary" read straight from the same table.

### Privacy note

The `anon` key can read the `claims` table, so a determined person could read
claimer names without the passcode. For a family baby shower that's fine. If
you need real privacy, claimer names would have to move behind a login — out
of scope for a one-day page.
