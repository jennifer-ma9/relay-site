# Relay — Backend & Database Design Sketch

## What the current site can't do yet
The GitHub Pages + Formspree setup is a static site with a mailbox attached. It can't: store listings so they update the Browse page automatically, let two kinds of users log in, track an application from a successor to a founder, or track a handoff through its stages. That's what a real backend adds.

## Recommended stack for where you're at
**Supabase** — a hosted Postgres database with built-in authentication and an auto-generated API, callable directly from plain JavaScript. No server code to write or host yourself.

Why this fits you specifically: it keeps your existing static HTML/CSS/JS files exactly as they are (still deployable on GitHub Pages) — you just add API calls. It teaches you real relational SQL, which is directly useful for an econ major working with data later. Free tier is generous enough for this project's scale for a long time.

The alternative most people suggest — Firebase — uses a NoSQL document model that's actually a worse fit here, because your data (projects → contacts → applications → handoffs) is inherently relational: things reference other things by ID, and you'll constantly want to ask questions like "show me all open applications for projects in the Education category." That's what SQL is built for.

## Core entities

**users**
| field | type | notes |
|---|---|---|
| id | uuid, pk | |
| email | text, unique | |
| name | text | |
| school | text | optional |
| created_at | timestamp | |

A single user can be a founder on one project and an applicant on another — don't hardcode "role" onto the user; it's contextual per-project (see below).

**projects** — the continuity record
| field | type | notes |
|---|---|---|
| id | uuid, pk | |
| name | text | |
| cause_area | text/enum | Education / Food / Clothing / Other |
| location | text | |
| founded_year | int | |
| status | enum | `draft`, `pending_review`, `open`, `in_handoff`, `active`, `archived` |
| weekly_rhythm | text | |
| budget_notes | text | |
| handoff_reason | text | |
| successor_requirements | text | |
| current_leader_id | uuid, fk → users.id | |
| created_at / updated_at | timestamp | |

**contacts** — kept separate from `projects` on purpose
| field | type | notes |
|---|---|---|
| id | uuid, pk | |
| project_id | uuid, fk | |
| name | text | |
| role | text | e.g. "on-site coordinator" |
| contact_info | text | **not publicly readable — see access control below** |

**applications**
| field | type | notes |
|---|---|---|
| id | uuid, pk | |
| project_id | uuid, fk | |
| applicant_id | uuid, fk → users.id | |
| message | text | why they want it, their availability/commitment |
| status | enum | `pending`, `accepted`, `rejected`, `withdrawn` |
| created_at | timestamp | |

**handoffs**
| field | type | notes |
|---|---|---|
| id | uuid, pk | |
| project_id | uuid, fk | |
| outgoing_leader_id | uuid, fk | |
| incoming_leader_id | uuid, fk | |
| shadow_start / shadow_end | date | the overlap window |
| status | enum | `in_progress`, `completed` |
| notes | text | |

## Status workflow (this is the actual product logic)
```
draft → pending_review → open ──(application accepted)──> in_handoff → active
                ↑                                                          │
           (admin rejects,                                        (later, leader
            needs more info)                                       graduates again)
                                                                          │
                                                                          ▼
                                                                    open again
```
`active` looping back to `open` is the whole point — a project should be able to cycle through multiple leaders over years without dying, which is the thing you're trying to fix.

## The access-control decision that matters most
Because several of your example projects involve **minors being served** (the tutoring program) and the people submitting are often **minors themselves**, don't make `contacts.contact_info` publicly readable. Recommended rule: contact details only become visible to an applicant *after* their application is accepted by the current leader — and even then, consider requiring a short admin/moderator review step before that unlock, given no one's verifying who these applicants really are otherwise. Supabase's Row Level Security policies are built exactly for rules like "user X can see this row only if condition Y" — this is a natural first real use of them.

## Suggested build order
1. **Projects table + Browse page reads from it** instead of hardcoded cards — your first real win, and doesn't require auth yet.
2. **Auth + the submission form writes to `projects`** (status `pending_review`) instead of just emailing you via Formspree.
3. **A simple admin view** (even just you, querying Supabase's dashboard directly) to flip `pending_review` → `open`.
4. **Applications table** + a form for incoming leaders to apply to a specific project.
5. **Contacts unlock logic** once an application is accepted — this is where you'll actually write your first Row Level Security policy.
6. **Handoffs table** to track the shadow period and eventually mark a project `active` again.

Each step is shippable on its own — you don't need steps 4–6 built before step 1 is useful.
