# Arabele

A daily ballet guessing game in French, inspired by Wordle. Each day, players receive up to five progressive clues about a classical ballet (or, as a bonus round, a Paris Opéra étoile) and try to identify it in as few attempts as possible. Streaks are tracked across sessions via Google login.

Live at: **[arabele.vercel.app](https://arabele.vercel.app)**

---

## How It Works

- One new ballet puzzle per day, rotating through a list of 100 ballets
- One bonus dancer puzzle per day, rotating through Paris Opéra étoiles only
- Five progressive clues revealed one by one on each wrong guess (Origine, Époque, Chorégraphie, Synopsis, Indice final)
- Previous clues can be swiped/scrolled through after they are revealed
- Autocomplete suggestions as you type
- Streak tracking for logged-in users (current streak + personal best)
- Hidden easter egg: type "elea" in the answer box

---

## Tech Stack

### Frontend
- Pure HTML, CSS, JavaScript — single `index.html` file, no build step
- Fonts: [Cormorant Garamond](https://fonts.google.com/specimen/Cormorant+Garamond) via Google Fonts
- SVG stage background with teal curtains, ballet shoes, and spotlight effects

### Hosting — Vercel
- Deployed via [Vercel](https://vercel.com) connected to this GitHub repository
- Every push to `main` triggers an automatic redeploy
- Free tier, custom subdomain: `arabele.vercel.app`
- To redeploy: simply push updated `index.html` to `main`

### Authentication — Google OAuth 2.0 via Supabase Auth
- Users sign in with Google via [Supabase Auth](https://supabase.com/docs/guides/auth)
- OAuth is handled entirely by Supabase — no separate backend needed
- Google OAuth credentials are registered in [Google Cloud Console](https://console.cloud.google.com)
- Authorised redirect URI: `https://qkgaqcopmwclwsdtyudm.supabase.co/auth/v1/callback`
- After login, Supabase redirects back to `https://arabele.vercel.app`
- Auth state is managed client-side via `supabase.auth.onAuthStateChange`

### Database — Supabase (PostgreSQL)
- Project: **Arabele** on [supabase.com](https://supabase.com)
- Project URL: `https://qkgaqcopmwclwsdtyudm.supabase.co`
- One table: `streaks`

#### `streaks` table schema

| Column | Type | Description |
|---|---|---|
| `id` | uuid | Primary key |
| `user_id` | uuid | References `auth.users(id)` |
| `current_streak` | int | Days played consecutively |
| `longest_streak` | int | Personal best streak |
| `last_played` | date | Last date the user completed a puzzle |
| `created_at` | timestamptz | Account creation timestamp |

Row Level Security (RLS) is enabled. Users can only read and write their own row.

#### SQL to recreate the table

```sql
create table streaks (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id) on delete cascade not null unique,
  current_streak int default 0,
  longest_streak int default 0,
  last_played date,
  created_at timestamptz default now()
);

alter table streaks enable row level security;

create policy "Users can read own streak"
  on streaks for select
  using (auth.uid() = user_id);

create policy "Users can upsert own streak"
  on streaks for insert
  with check (auth.uid() = user_id);

create policy "Users can update own streak"
  on streaks for update
  using (auth.uid() = user_id);
```

---

## Adding New Content

Ballets and dancers are stored as plain JavaScript arrays inside `index.html`. To add entries, find the `const B=[...]` (ballets) or `const D=[...]` (dancers) arrays and append new objects following the existing format:

```js
{n:"Nom du Ballet", c:[
  {t:"Origine",   x:"Pays · nationalité du compositeur"},
  {t:"Époque",    x:"Créé en XXXX"},
  {t:"Chorégraphie", x:"Nom du chorégraphe"},
  {t:"Synopsis",  x:"Résumé vague de l'histoire"},
  {t:"Indice final", x:"Résumé plus précis"}
]}
```

The daily puzzle is selected by: `dayOfYear % arrayLength`. To pin a specific ballet to a specific date, count the day of year and place the entry at `dayOfYear % arrayLength`.

---

## Updating the Site

1. Edit `index.html` locally
2. `git add index.html && git commit -m "your message" && git push`
3. Vercel redeploys automatically within ~30 seconds
