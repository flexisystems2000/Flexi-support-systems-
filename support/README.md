# Flexi Educational Consult Support Ticket Centre

This is a single-page support portal. Firebase Authentication is the identity provider, matching the configuration used by `login.php`; Supabase Postgres stores tickets, replies, assignments, and statuses.

## Included
- Existing Flexi logo URL and brand blue `#003366` / green `#2E8B57`.
- Royal blue `#4169E1` accent and animated blue–royal-blue–green gradient.
- Email/password and Google sign-in through the existing Firebase project.
- Ticket creation, ticket numbers, status tracking, conversation replies, search and filters.
- Staff status/priority/assignment controls and realtime updates.
- Responsive mobile layout and royal-blue footer with a small selection of Flexi links.

## Important: enable Firebase as Supabase's third-party auth provider
The database is Supabase, but users must continue signing in through Firebase. Supabase must be configured to validate Firebase ID tokens before its RLS policies can identify users.

1. Open the Supabase project `ryvauylmymcvbvvlaceb`.
2. Go to **Authentication → Sign In / Providers** (the exact dashboard label may vary) and locate **Third-Party Auth / Firebase**.
3. Enable Firebase and enter Firebase project ID `waec2026jamb2027`. Save the settings.
4. Keep Firebase Authentication enabled for Email/Password and Google. The configured auth domain in `login.php` is `auth.flexieduconsult.com.ng`.
5. The portal passes the signed-in Firebase ID token to Supabase; do not replace this with a Supabase Auth login or expose a service-role key in the browser.

## Database
The matching migration is in `supabase-schema.sql`. It changes only the `support_*` tables to use Firebase UID text identifiers, preserves existing support ticket/message data, and enables RLS policies based on the verified Firebase JWT subject.

## Grant the first support administrator
After the third-party integration is enabled, sign in to the portal once with the account that should manage support. In Firebase Console → Authentication → Users, copy that user's UID. Then run the following in Supabase SQL Editor, replacing the three example values:

```sql
insert into public.support_staff (user_id, email, display_name, role)
values ('FIREBASE_UID_HERE', 'your-email@example.com', 'Flexi Support Admin', 'admin')
on conflict (user_id) do update
set email = excluded.email, display_name = excluded.display_name,
    role = 'admin', active = true;
```

Only active staff records can access the staff workspace. Do not grant staff access to ordinary customer accounts.

## Deploy
- Upload this `support` folder to the `flexieduconsults` repository.
- For a separate Vercel deployment, import the repository and set **Root Directory** to `support`.
- Add `support.flexieduconsult.com.ng` as the deployment domain and apply the DNS record Vercel gives you. Do not remove or replace existing email/MX records.
- In Firebase Console → Authentication → Settings → Authorized domains, ensure `support.flexieduconsult.com.ng` is allowed. Keep the existing auth domain too.

The portal is a single HTML page and does not require a build step. The Supabase publishable key is intended for browser use; access is constrained by RLS. Never place a Supabase service-role key in this folder.
