# CMS Setup — Hand-Off Steps

This folder contains everything needed to let Samantha Cline (scline@seanc.org)
manage the retirees.seanc.org site through a web-based admin interface at
`retirees.seanc.org/admin`.

What's new in this bundle:

- `admin/index.html` + `admin/config.yml` — the Decap CMS admin interface
- `_data/forums.json` — the current forum list (what used to be hardcoded)
- `_data/newsletters.json` — the current Scoop + Retiree Report PDF links
- Updated `index.html` — now reads forums + newsletter links from the JSON files
  instead of having them hardcoded. Visually identical to before.

**None of this works until you do the one-time Netlify setup below.** The code
is ready, but the "login with password" piece (Netlify Identity) and the
"let the admin save changes" piece (Git Gateway) are dashboard toggles only
you can flip.

---

## Part 1 — One-time Netlify setup (~15 minutes)

### Step 1: Deploy the files

Drag the whole `retirees-site/` folder to your Netlify site like usual.
Deploy should succeed with no errors. **Don't test anything yet** — the admin
won't work until the next steps are done.

### Step 2: Enable Netlify Identity

1. In the Netlify dashboard, open the **retirees** site.
2. Click the **Integrations** tab (or **Site configuration** → **Identity** on
   some dashboard versions — Netlify shuffles this around).
3. Find **Identity** and click **Enable Identity**.
4. Scroll to **Registration preferences** and change it from "Open" to
   **"Invite only"**. This stops random people from signing up.
5. Scroll to **External providers** — leave all of these DISABLED (no Google,
   no GitHub, no GitLab). We want email-and-password only so there's nothing
   confusing on the login screen.

### Step 3: Enable Git Gateway

Still in the Identity section (or **Services** on some dashboard versions):

1. Find **Git Gateway** and click **Enable Git Gateway**.
2. That's it — Netlify authenticates to your Git repo on your behalf, so
   Samantha can save changes through the admin without needing a GitHub account.

### Step 4: Invite Samantha

1. In the Identity section, click **Invite users**.
2. Enter `scline@seanc.org` and click **Send**.
3. Netlify sends her an email with a sign-up link. She clicks it, sets her own
   password, and that's her login forever after.

### Step 5: Confirm the branch name in config.yml

Open `admin/config.yml` (just added) and look at the top. It says:

```yaml
backend:
  name: git-gateway
  branch: main
```

If your Netlify repo actually uses `master` as the main branch, change it to
`master`. Save and redeploy. (If you're not sure, check in GitHub — it's at
the top of the file list.)

---

## Part 2 — Test it yourself first

Before giving the URL to Samantha, run through it once:

1. Go to `retirees.seanc.org/admin` in your browser.
2. Click **Login** — you'll see the Identity widget. Log in with your own
   SEANC email (if Jonathan is already registered as a Netlify user) or
   invite yourself first.
3. You should see two collections in the sidebar: **Retiree Forums** and
   **Newsletter Links**.
4. Click **Retiree Forums** → see the 3 current forums. Click one to edit it.
5. Add a test forum — give it any date 2+ days in the future, fill in the
   fields, hit **Publish**.
6. Wait ~45 seconds, then open `retirees.seanc.org` in a different tab.
   The test forum should appear in chronological order.
7. Delete the test forum via the admin — confirm it disappears from the
   public site after another ~45 seconds.
8. If all of that works, send Samantha her onboarding email (template below).

---

## Part 3 — Hand-off email for Samantha

Something like:

> Subject: Managing retirees.seanc.org — your login + quick walkthrough
>
> Hi Samantha,
>
> You now have access to update the Retiree Resources site
> (retirees.seanc.org) directly, instead of asking me to do it. The two
> things you can manage are:
>
> 1. **The list of upcoming Retiree Forums** (add, edit, remove)
> 2. **The weekly Scoop PDF link and quarterly Retiree Report PDF link**
>
> **How to log in:**
>
> 1. Check your inbox for a message from Netlify with the subject "You've
>    been invited to join retirees" — click the link and set a password.
> 2. From then on, go to **retirees.seanc.org/admin** and log in with your
>    SEANC email + the password you set.
> 3. Bookmark that URL.
>
> **To add a forum:**
> Click "Retiree Forums" → "New Forum" → fill in the fields → "Publish".
> It appears on the public site in about 30-60 seconds.
>
> **To remove a past forum:**
> Open the forum, hit "Delete" at the bottom. (Past forums also auto-hide
> from the public site after their date passes, but deleting keeps your
> admin list tidy.)
>
> **To update the Scoop link each week:**
> Click "Newsletter Links" → paste the new PDF URL in "The SEANC Scoop"
> → "Publish".
>
> If you ever get stuck or something looks wrong, email or Slack me — I can
> see every change you make in Git history and can undo anything in seconds.
>
> — Jonathan

---

## How to add future features

If later you want Samantha (or anyone else) to edit other sections of the site
— podcast episode of the week, scholarship deadlines, pillar text — the pattern is:

1. Move that section's data from the hardcoded HTML into a new JSON file under
   `_data/`
2. Add a rendering loop to the dynamic script block in `index.html` (same
   pattern as `renderForums` and `renderNewsletters`)
3. Add a new collection to `admin/config.yml` with the right fields

Same recipe each time. The hardest part is already done.

---

## Troubleshooting

**"Site not found" when Samantha tries to log in at /admin.**
Netlify Identity isn't enabled yet. Go back to Part 1, Step 2.

**She can see the admin but gets an error saving.**
Git Gateway isn't enabled. Go back to Part 1, Step 3.

**The forums page shows "Loading upcoming forums…" forever.**
Something's wrong with the fetch. Open browser DevTools → Network tab,
refresh, look for `_data/forums.json`. If it's a 404, the deploy didn't
include the `_data/` folder. If it's blocked by Cloudflare, add
`/_data/*` as a cache-bypass rule in Cloudflare.

**Changes don't show up on the public site.**
Netlify needs ~30-60 seconds to rebuild after each publish. Also check that
`published: true` is checked on the forum, and that the date is today or
later (past forums are auto-hidden).

**Samantha accidentally broke something.**
In GitHub, every change she makes is a commit. Go to the repo → Commits →
find the one that broke things → "Revert" button. Takes 10 seconds.
