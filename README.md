# 🌱 Grocefy

A live-sync shared grocery list app. Browse a categorized catalog, build a list, save it, share it with someone, and shop together in real time — no app store, no backend server, just a static page and Firebase.

**Live app:** https://habib-farhan.github.io/Grocefy/

---

## What it does

- Browse groceries by category (Produce, Bakery, Dairy, Meat, Pantry, Snacks, Frozen, Drinks, Household, Cat Supplies, plus any custom categories you create) and tap to add items to a list you're building.
- Pick a quantity and unit for each item (pcs, g, kg, ml, l, pack) from a quick popup.
- Save the list with an optional title. It shows up in **My Lists** — yours, or anyone's shared with you.
- Open a saved list to start shopping: check items off as you go, then finalize it.
- Finalized lists move to **History**, with a count and the option to delete old ones.
- **Share a list** with another person. They get a notification, and while you're both looking at the same list mid-shop, adding an item shows up for them instantly with a toast.
- Add an item that isn't in the catalog — it's auto-categorized by a lightweight keyword guess (editable before saving) and stays in the catalog permanently afterward, for everyone.
- Create entirely new categories on the fly with your own emoji and name.
- Simple name + color-icon login, remembered per device — no passwords.

---

## Tech stack

- **Frontend:** a single self-contained `index.html` — vanilla JS, no build step, no framework.
- **Backend:** Firebase Realtime Database (free Spark plan) for live sync, and Firebase Anonymous Authentication (silent, no UI) purely so the database rules can require "must be signed in" instead of being open to the internet.
- **Hosting:** GitHub Pages (static, free).

---

## Project structure

```
├── index.html   # the entire app
├── config.js    # Firebase project config (safe to be public — see note below)
└── README.md
```

> **Is it okay that `config.js` is public?** Yes. A Firebase web API key isn't a secret the way a server API key is — it just identifies which Firebase project the app talks to. The real access control lives in your **Realtime Database security rules**, not the key. This repo's rules require `auth != null` (see Setup below), which is what actually keeps the data safe.

---

## Setup (if you're forking this)

1. **Create a Firebase project** at [console.firebase.google.com](https://console.firebase.google.com).
2. **Enable Realtime Database** (Build → Realtime Database → Create Database).
3. **Enable Anonymous Authentication** (Build → Authentication → Get Started → Sign-in method → Anonymous → Enable).
4. **Set your database rules** (Realtime Database → Rules tab):
   ```json
   {
     "rules": {
       ".read": "auth != null",
       ".write": "auth != null"
     }
   }
   ```
5. **Get your web app config** (Project Settings → your app → Firebase SDK snippet → Config) and paste it into `config.js`:
   ```js
   const firebaseConfig = {
     apiKey: "...",
     authDomain: "...",
     databaseURL: "...",
     projectId: "...",
     storageBucket: "...",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
6. **Restrict the API key** (optional but recommended): Google Cloud Console → APIs & Services → Credentials → your key → API restrictions → allow **Identity Toolkit API** and **Firebase Realtime Database Management API** → Website restrictions → your GitHub Pages domain (`https://*.github.io/*`) and, if testing locally, `http://localhost:PORT/*`.
7. **Deploy:** push to a GitHub repo, enable GitHub Pages (Settings → Pages → deploy from branch), done.

### Testing locally without deploying

```bash
python3 -m http.server 8000
```
Then open `http://localhost:8000`. Remember to add `http://localhost:8000/*` to the API key's website restrictions first, or Firebase calls will be blocked.

---

## Data model

Everything lives under one Firebase node, `lists`:

```
lists/
  <listId>/
    title: string | null        // null → shown as "New List N"
    ownerName: string
    sharedWith: string[]         // names of users this list is shared with
    status: "draft" | "active" | "completed"
    createdAt: timestamp
    completedAt: timestamp | null
    items:
      <itemKey>/
        name: string
        en: string                // English translation, if any
        aisle: string              // category label, e.g. "1. 🥦 Produce (Obst & Gemüse)"
        checked: boolean
        qty: number
        unit: "pcs" | "g" | "kg" | "ml" | "l" | "pack"
        addedBy: string
```

A few supporting nodes:

- `users/<name>` — registered names/avatars, last login time (used for the login screen and the share picker).
- `customCatalogItems/<id>` — permanently-saved custom items, each with a `categoryId`.
- `customCategories/<id>` — user-created categories (name + emoji).
- `notifications/<name>/<id>` — in-app notifications (currently just list-sharing events).

A list only ever shows up for you if `ownerName` is you, or your name is in `sharedWith` — that filtering happens client-side, on top of the blanket `auth != null` rule. This is a reasonable model for a small trusted group; it is **not** a hard security boundary between users (anyone signed in can technically read any list via a direct API call). If that ever matters, the fix is to move the ownership check into the database rules themselves.

---

## Known limitations

- **No push notifications.** In-app only — you'll see a share notification or a live update only while the app is open. Adding real push (via Firebase Cloud Messaging) is possible later but requires upgrading to Firebase's Blaze plan and, on iPhone, installing the site to the home screen (iOS 16.4+).
- **Item icons are emoji**, not custom artwork — chosen deliberately to avoid hosting image assets or licensing concerns. A handful of items reuse a close-enough emoji since a perfect match doesn't exist (e.g. tofu, mustard).
- **Auto-categorization is a simple keyword match**, not AI — it gets obviously-related items right (e.g. "Shampoo" → Household) and defers anything unclear to an "Other" category you can recategorize by hand.
- **Sharing is owner-managed only** — the person who created a list is the only one who can add or remove people from it, or delete it.

---

## Version history

| Version | What it added |
|---|---|
| v1 | Locked down open database rules, fixed a key-sanitization bug, fixed a write race condition |
| v2 | Name + avatar login, remembered per device |
| v2.1 | Icon-based avatars, profile dropdown (last login, recent trips, switch user) |
| v3 | Multi-list data model — Catalog → draft → save → My Lists → Shopping Mode → History |
| v4 | ~40 new catalog items, new Cat Supplies category, permanent auto-categorized custom items |
| v5 | History count + delete |
| v6 | User-created custom categories |
| v7 | Per-item emoji icons |
| v8 | Quantity & unit selector |
| v9 | Multi-user sharing + in-app notifications |
