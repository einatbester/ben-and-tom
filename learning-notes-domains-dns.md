# Learning Notes: Git, GitHub, Domains & DNS

Compiled from our `ben-and-tom` practice project — the goal of this whole exercise is
to build real confidence for eventually switching **frooteria.de** from its current
live website to the new one, without breaking anything (especially email).

---

## 1. Project context

- **`ben-and-tom`** — a small practice website, used purely to learn the full
  pipeline: writing code → GitHub → live hosting → custom domain → updating a live site.
- **`frooteria-local-shine`** — the real replacement website built (via Lovable) for
  **Frooteria Juices & Bagels**, a real cafe in Munich, currently live at `frooteria.de`.
  Kept as a **private** GitHub repo on purpose, so the prototype isn't publicly visible
  under the real business's name before it's ready. This is normal, legitimate practice —
  not an issue — as long as it's never presented publicly as their live site without consent,
  and their copyrighted content/photos aren't redistributed publicly.
- The end goal: be the ones accountable for actually executing the real domain switch
  for Frooteria (not just handing them a repo and hoping they figure out DNS), while the
  **client keeps ownership** of the domain/registrar account itself.

---

## 2. Git & GitHub basics

- **Git repository ("repo")**: a folder whose history of changes is tracked.
- **Commit**: a saved snapshot of changes with a message.
- **Push**: sending your local commits up to GitHub.
- **Fork**: your own personal copy of *someone else's* repository. Editing your fork
  has zero effect on the original — it's like editing a photocopy.
- **Pull Request (PR)**: a formal request asking the owner of a repository to review and
  merge changes from a branch (often from your fork) into their main codebase. Nothing
  takes effect until it's merged.
- **Public vs Private repo**: a private repo is invisible to anyone without access; a
  public repo is visible to the world. GitHub Pages (free tier) **only works on public
  repos** — a private repo has no working live-link at all, not even a login-only one
  (that requires a paid GitHub plan, and even then it's collaborators-only, never a
  public link).

---

## 3. GitHub Pages — how our `ben-and-tom` site actually went live

GitHub Pages is a real, globally-distributed public web host — completely different
from running `http-server` locally (which only worked on our own machine). The moment
Pages was turned on (Settings → Pages → source: `main` branch) and the build finished,
the site became reachable by anyone on the internet at:

```
https://einatbester.github.io/ben-and-tom/
```

### Why a custom domain needs TWO layers, not one

1. **DNS layer**: a record (e.g. a CNAME) says "domain X → `einatbester.github.io`."
   On its own this is too generic — it doesn't say *which* of your repos to serve.
2. **GitHub Pages layer**: inside the specific repo's Settings → Pages → "Custom domain"
   field, you type the exact domain. This does two things:
   - Commits a file literally named `CNAME` into that repo, containing the domain name.
   - Registers that domain as claimed by **that repo only** — GitHub enforces this is
     exclusive per account. If you tried to also claim the same domain on another repo,
     GitHub would refuse it.

This is *how* you can be sure only `ben-and-tom` (and not `frooteria-local-shine`) is
linked to a given custom domain: check for the `CNAME` file inside the repo, and know
that GitHub won't let a second repo claim the same domain.

### Multiple free-domain sites from one GitHub account

Yes — the DNS side has no limit (you can create unlimited records pointing at
`einatbester.github.io` from as many domains as you like). The pattern for running
several independent live sites for free:

| Repo | Pages custom domain |
|---|---|
| `ben-and-tom` | `besterbros.is-a.dev` |
| some other repo | `something-else.is-a.dev` (or any other domain) |

Each repo/domain pair is fully independent.

---

## 4. DNS fundamentals

**A domain is not the same thing as DNS.**

- **Domain**: a name (e.g. `frooteria.de`) you register and hold for a period, like a
  lease, renewed yearly, bought from a **registrar**.
- **DNS (Domain Name System)**: the internet's phonebook — the system that translates
  a domain name into the numeric address (IP) computers actually use to find each other.
- **DNS provider**: whoever holds the *actual record book* (the "zone") for a domain.
  By default this is usually the same company as the registrar, but it can be moved to
  a different company for free (commonly Cloudflare) without moving the domain itself.
- **Host**: the actual server that holds your website's files — what the DNS record
  ultimately points *to*. A separate role again from both of the above.

So there are three separable roles: **registrar → DNS provider → host**. Sometimes one
company does all three (e.g. GoDaddy can be registrar + DNS host + hosting), sometimes
they're split across three different companies.

### Key DNS record types

| Record | Purpose |
|---|---|
| **A** | name → IP address directly (e.g. `frooteria.de` → `81.169.202.53`) |
| **CNAME** | name → *another name*, which then gets looked up too (e.g. `besterbros.is-a.dev` → `einatbester.github.io`) |
| **MX** | which server handles **email** for the domain |
| **NS** | which company's servers hold the domain's real DNS records (the "nameservers") |

### TTL & propagation

**TTL (time-to-live)**: how long other computers are allowed to cache an old DNS answer
before re-checking. This is why DNS changes aren't instant everywhere — every resolver
that already cached the old answer keeps using it until its TTL expires. This is
something to plan around (lower the TTL in advance of a real switch) rather than something
you can force to update instantly everywhere.

---

## 5. is-a.dev — why it's good practice but not a perfect analogy

**What it is**: a free-subdomain giveaway service. `is-a.dev` itself is a normal
registered domain owned by a small open-source project; they run its DNS and let
anyone request a subdomain of it (e.g. `besterbros.is-a.dev`) by submitting a **pull
request** to their GitHub repo (`is-a-dev/register`) containing a JSON file describing
where the subdomain should point. A bot validates the PR's format, and a **human
maintainer manually reviews and merges it** — this can take hours to days.

### What DOES transfer to the real Frooteria switch
The core mechanic: a domain points somewhere via a DNS record; you change the record;
the world sees the new destination after propagation. That muscle memory is real and useful.

### What DOES NOT transfer
1. **The interface** — is-a.dev uses a slow, PR-and-wait-for-human-approval workflow.
   The real switch for `frooteria.de` happens **instantly**, self-service, inside
   whatever registrar/DNS dashboard they use (see below) — no one has to approve it.
2. **The real danger: email.** is-a.dev subdomains have no email attached, so this
   practice never exposes the #1 real risk of a domain switch. `frooteria.de`'s email
   is tied to the same domain as its website (see findings below) — carelessly
   repointing DNS, or changing nameservers instead of just a single record, can
   **silently break their email** along with the website. This risk is invisible in
   the is-a.dev exercise but very real for the actual client switch.

---

## 6. Hosting / domain / backend provider comparison

Quick reference — these aren't all the same category of thing:

**Registrars (sell you actual domain ownership)**
- **GoDaddy** — pay yearly per domain (registration + often pricier renewal); optional
  add-on hosting/email sold separately. This is where `frooteria.de` is currently
  registered, and where its DNS is currently managed too.

**Hosting/deployment platforms (serve your site's files; free subdomain; can attach a domain you already own)**
- **Netlify** / **Vercel** — near-identical: git-based auto-deploy, free subdomain,
  generous free tier, attaching your own custom domain is **free** on both (you still
  need to have bought the domain itself elsewhere). Vercel has tighter Next.js
  integration; Netlify has slightly more mature form/serverless tooling for plain
  static sites. Otherwise largely interchangeable.
- **Firebase Hosting** — Google's equivalent; free subdomain, custom domain attachment
  free even on the free "Spark" tier.
- **Lovable** — same auto-deploy idea, free subdomain, but attaching a custom domain
  is **gated behind a paid plan** (the one exception in this group).

**Backend-as-a-service (not website hosting at all)**
- **Supabase** / **Firebase** (its non-Hosting half) — provide a database, auth, file
  storage, and server functions for an app's backend. They don't host visible web pages
  or offer domains for that — a frontend would still be deployed elsewhere (e.g.
  Netlify/Vercel) and just talk to Supabase/Firebase behind the scenes.

**Payment models, one line each**
- GoDaddy: pay per domain, per year.
- Netlify / Vercel / Firebase Hosting: free tier covers most personal projects; paid
  tiers add bandwidth/build minutes/team seats — custom domains stay free either way.
- Supabase / Firebase (backend): free tier with usage caps, then pay-as-you-go —
  unrelated to domains.
- Lovable: monthly subscription for AI generation credits; custom domain is a paid-tier feature.

---

## 7. Frooteria.de — actual current setup (looked up directly, not guessed)

| Item | Finding |
|---|---|
| Registrar / DNS | **GoDaddy** (`ns81.domaincontrol.com`, `ns82.domaincontrol.com`) |
| Hosting IP | `81.169.202.53` — a German shared-hosting range (**Strato**), not GitHub Pages or any platform above |
| Email | `MX` record points to `www.frooteria.de` **itself** — meaning their email is very likely hosted on the **same server** as their current website |

### Why this matters for the eventual real switch

> Is-a.dev will teach the general concept but not the real risk. The core mechanic is
> the same either way — a domain points somewhere via a DNS record, you change that
> record, the world sees the new site after propagation. That muscle memory transfers.
>
> But two things won't transfer from is-a.dev practice:
> 1. **The interface** — is-a.dev uses a GitHub-PR-and-wait-for-approval workflow; the
>    real switch happens instantly in GoDaddy's DNS control panel (self-service, no
>    approval needed).
> 2. **The actual danger** — is-a.dev subdomains have no email attached, so you'd never
>    encounter the #1 real risk in a domain switch: if you carelessly repoint
>    `frooteria.de`'s DNS (or change nameservers instead of just the A record), you can
>    silently break their email along with the website. That risk is invisible in the
>    is-a.dev exercise but very real here.
>
> Recommendation: finish the is-a.dev exercise for the DNS/propagation mechanics
> (cheap, safe, good practice), but treat "how not to break email when switching
> frooteria.de" as a **separate checklist item** to work through specifically,
> using their actual current setup.

### Starting point for that separate checklist (not yet done — for later)

- [ ] Confirm exactly which record(s) currently make email work (`MX` → `www.frooteria.de`
      → `81.169.202.53`) so the switch plan explicitly leaves email untouched
- [ ] Decide the new host in advance (GitHub Pages / Netlify / Vercel / etc.) and test the
      new site fully on its free subdomain before touching `frooteria.de`'s DNS at all
- [ ] Lower the domain's DNS TTL a day or two *before* the switch, so the real cutover
      propagates fast
- [ ] On switch day: change only the specific record(s) needed to point the domain at the
      new host (e.g. the `A`/`CNAME` for the site) — do **not** touch nameservers or the
      `MX` record
- [ ] Monitor both the new site and email delivery immediately after the change
- [ ] Keep the old hosting/account active for a rollback window in case anything breaks

---

## 8. Where things stand right now (practice project)

- ✅ `ben-and-tom` pushed to GitHub, live at `https://einatbester.github.io/ben-and-tom/`
- ✅ GitHub Pages enabled and confirmed working (HTTP 200)
- ⏳ `besterbros.is-a.dev` PR submitted (PR #52219 on `is-a-dev/register`) — template
  filled correctly, all automated checks passed, **awaiting human maintainer review/merge**
- ⏳ Once merged: still need to add `besterbros.is-a.dev` as the custom domain in
  `ben-and-tom`'s GitHub Pages settings (the "Layer 2" link described in section 3)
- ⏳ After that: practice actually *updating* the live site (the real point of this
  whole exercise) and separately work through the frooteria.de switch-safety checklist above
