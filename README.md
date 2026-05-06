# EFR Field App

Offline-capable Progressive Web App for Edible Food Recovery field data entry.  
Works without internet — forms save locally and sync when connection is restored.

---

## What it does

- **Site Visit form** — generator, staff, documentation checkboxes, food types, notes
- **Inspection form** — full Section B/C/D with all compliance fields
- **Offline queue** — stores submissions in browser storage, auto-syncs when back online
- **Installable** — staff can add it to their phone home screen like a native app

---

## Deployment (GitHub Pages — free, 5 minutes)

1. Create a new GitHub repository (e.g. `efr-field-app`)
2. Upload these four files to the repo root:
   - `index.html`
   - `sw.js`
   - `manifest.json`
   - `icon-192.png` and `icon-512.png` (see Icons section below)
3. Go to **Settings → Pages → Source → Deploy from branch → main**
4. Your app will be live at `https://your-org.github.io/efr-field-app/`

---

## Icons

The manifest references `icon-192.png` and `icon-512.png`.  
Create simple icons using any tool, or generate them at https://favicon.io using a 🌿 emoji.

---

## First-run setup (staff side)

1. Open the URL in Safari (iPhone) or Chrome (Android)
2. On the setup screen, enter:
   - **Supabase URL**: your project URL (e.g. `https://xxxx.supabase.co`)
   - **Anon/Public Key**: from your Supabase project Settings → API
3. Tap **Save & Connect**
4. To install: tap Share → Add to Home Screen (iPhone) or the install prompt (Android)

Credentials are stored only on the device and never sent anywhere except your Supabase project.

---

## Required Supabase RLS Policies

Run this SQL in your Supabase SQL editor to allow the anon key to read and write:

```sql
-- READ access for lookup data
create policy "anon_read_generators" on edible_food_generators
  for select using (true);

create policy "anon_read_staff" on staff
  for select using (true);

create policy "anon_read_food_types" on food_types
  for select using (true);

-- WRITE access for field submissions
create policy "anon_insert_visits" on outreach_visits
  for insert with check (true);

create policy "anon_insert_inspections" on inspections
  for insert with check (true);

create policy "anon_upsert_compliance_status" on compliance_status
  for all using (true) with check (true);

create policy "anon_insert_food_types" on generator_food_types
  for insert with check (true);

create policy "anon_update_generators" on edible_food_generators
  for update using (true);
```

> **Note:** These are permissive policies suitable for an internal staff tool.  
> If you want stricter security, scope them to specific roles or use Supabase Auth.

---

## How offline sync works

1. Staff fill out a form — if online, it submits immediately to Supabase
2. If offline, the form data is saved to the browser's IndexedDB storage
3. When connectivity is restored, the app auto-detects it and syncs all pending records
4. Staff can also manually tap **Sync Now** from the Queue tab or Home screen
5. The queue badge on the Queue tab shows how many records are waiting

---

## Data written per form type

**Site Visit:**
- `outreach_visits` — one INSERT
- `compliance_status` — UPSERT (additive: only TRUE values update existing record)
- `generator_food_types` — additive INSERTs (ON CONFLICT DO NOTHING)

**Official Inspection:**
- `inspections` — one INSERT with full `inspection_data` JSON
- `generator_compliance` — UPSERT (only if compliant, never downgrades)
- `compliance_status` — UPSERT (additive)
- `edible_food_generators` — PATCH for container fields (only TRUE values)
- `generator_food_types` — additive INSERTs

**Educational Site Visit:**
- `outreach_visits` — INSERT with `visit_type = 'educational'` and full `inspection_data` JSON
- `compliance_status` — UPSERT (additive)
- `edible_food_generators` — PATCH for container fields
- `generator_food_types` — additive INSERTs

---

## Notes

- The generator list is cached for 4 hours. Staff can force-refresh from the Home screen.
- Photos are not supported in the offline version — staff should add them later in the main dashboard.
- The service worker caches the app shell, so the app loads even with no connection after first visit.
