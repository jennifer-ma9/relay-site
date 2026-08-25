# Adding a real project listing to Relay

Your site is static — there's no database, so "listing a project" is a manual step:
read the submission → decide to approve it → paste in a small card → commit to GitHub.

## Step 1 — Read the submission
Check your Formspree dashboard or email for the fields the founder filled in
(project name, cause area, location, weekly rhythm, etc.).

## Step 2 — Copy this template
Paste this block into `index.html`, inside the `<div class="wrap">` of the
`#browse` section — right *above* the `<div class="empty-state">` line.
If you add at least one real card, you may want to remove the empty-state
div entirely (or keep both if you expect more listings soon).

```html
<div class="listings">
  <div class="listing-card">
    <span class="tag">Education</span>
    <h3>Wednesday Math Circle</h3>
    <span class="meta">Cape Town, South Africa · founded 2022</span>
    <p class="desc">Weekly math &amp; science tutoring for 10–14 girls over a shared video call. Founder graduating this spring.</p>
  </div>
  <!-- Add another <div class="listing-card">...</div> for each additional approved project -->
</div>
```

## Step 3 — Fill in the real details
Swap in the actual project name, cause area (Education / Food insecurity /
Clothing insecurity / Other), location, founded year, and a 1–2 sentence
description pulled from their "what does this project do week to week"
answer.

## Step 4 — Commit
Same as always: edit `index.html` on GitHub, paste your updated version in,
commit. Give it a minute, then refresh your site.

## A note on privacy
Don't put a founder's personal email, phone number, or the specific contact
info of the community they serve directly into a public card — that
information should stay private until a real applicant is vetted and
approved. The public card is just a teaser (name, cause, general location,
brief description); the sensitive details (contacts, documents, budget)
stay in your inbox until you're ready to hand them to a specific person.

## Longer term
Once you have more than a handful of real listings, doing this by hand will
get tedious — that's exactly the point where the Supabase/database version
from earlier (or trying it again once Supabase is stable) starts to pay off,
since it would let the Browse page update itself automatically instead of
you manually editing HTML each time.
