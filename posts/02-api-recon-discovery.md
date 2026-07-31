# API Recon: Finding the Attack Surface Before You Attack Anything

In the last post, I talked about why APIs deserve their own testing mindset. This post covers the first real phase of that mindset: **recon** — figuring out what an API actually is, what it does, and where its edges are, before you send a single malicious request.

If you skip this phase, you're testing blind. Most of the interesting bugs in APIs live in functionality nobody told you existed — so the goal here is to map as much of that hidden surface as possible.

## Step 1: Look for Documentation First

Before doing any manual digging, check if the API is documented. Documentation exists so developers know how to integrate with an API, and it comes in two flavors:

- **Human-readable** — explanations, examples, usage scenarios, written for people
- **Machine-readable** — structured JSON/XML/YAML meant to be consumed by tooling (think OpenAPI/Swagger specs)

If an API is meant for external developers, its docs are often public — so this is always worth checking first. But even when docs aren't openly linked, they may still exist and just be poorly hidden. Common paths worth checking:

- `/api`
- `/swagger/index.html`
- `/openapi.json`

A useful trick: if you find one endpoint tied to documentation, walk back up the path. For example, finding `/api/swagger/v1/users/123` means it's worth also checking:
- `/api/swagger/v1`
- `/api/swagger`
- `/api`

Documentation is often left exposed unintentionally — sometimes with an "unpredictable" path that developers assumed was secure just because it wasn't linked anywhere. That assumption is exactly what a wordlist-based search (via Burp Intruder) is built to break.

**If you do find machine-readable docs**, don't just read them — feed them into tooling. Burp Scanner can crawl and audit OpenAPI documentation directly, and there are dedicated extensions for parsing OpenAPI specs. This turns a documentation file into a ready-made map of the entire API surface, params and all.

## Step 2: Map Endpoints by Browsing (Even If You Have Docs)

Documentation can be outdated or incomplete, so it's worth doing this step regardless. Two techniques matter here:

**Crawl and browse manually.** Look for URL patterns that hint at API structure — `/api/` being the obvious one — while browsing the app normally.

**Dig into JavaScript files.** Frontend JS often references API calls that never get triggered just by clicking around the UI. This is one of the highest-value recon steps for APIs specifically, because it surfaces endpoints the visible app never uses directly. Manual review works, but there are tools built for extracting these references at scale rather than eyeballing minified JS files line by line.

By this point you should have a working list of real endpoints — not guesses, actual observed API calls.

## Step 3: Interact With What You've Found

Once you know an endpoint exists, the next question is: what else can it do that isn't obvious from how the app uses it?

**Try different HTTP methods.** An endpoint you only ever see hit with `GET` might also respond to `POST`, `PATCH`, or `DELETE` — and each of those can expose entirely different functionality. A `/api/tasks` endpoint might quietly support creating and deleting tasks even if the UI only ever reads them. Burp Intruder has a built-in HTTP verbs list specifically for cycling through methods automatically.

One important caution here: when testing different methods, target low-priority objects. You don't want to accidentally delete or corrupt something real just to find out an endpoint accepts `DELETE`.

**Try different content types.** APIs often behave differently depending on what format you send data in. Switching the `Content-Type` header (and reformatting the body to match) can:
- Trigger error messages that leak useful internal details
- Slip past validation that was only built for one format
- Expose logic that's secure for JSON but not for XML, or vice versa

**Read every response closely — including the errors.** Error messages are frequently the richest source of information you'll get during recon. A verbose error can hand you field names, internal object structure, or validation logic you'd otherwise have to guess at.

## Step 4: Brute-Force for What's Still Hidden

Once you've exhausted what you can find by browsing and reading JS, the last recon move is systematic guessing. If you've found `PUT /api/user/update`, it's reasonable to assume siblings might exist at the same structural position — `/delete`, `/add`, and so on.

This is where Intruder earns its keep: feed it a wordlist of common API naming patterns and industry terms, but don't rely on generic lists alone — fold in terms specific to the application itself, based on everything you picked up during earlier recon steps. A generic wordlist finds generic endpoints. A tailored one finds the ones that actually matter for this specific target.

## Why This Order Matters

Notice the progression here: **read what's given → observe what's used → probe what's possible → brute-force what's hidden.** Each step narrows the guesswork for the next one. Jumping straight to brute-forcing without doing the documentation and browsing steps first means you're throwing wordlists at a target you don't understand yet — technically possible, but far less efficient than letting each recon step inform the next.

This is also the mindset shift from typical web app testing I mentioned in the last post: you're not just cataloguing what the app *shows* you, you're actively working out what the API is *willing to do*, regardless of whether the frontend ever asks it to.

**Next up:** once you've mapped the endpoints, the next question is what parameters they secretly accept — and what happens when one of those hidden parameters turns out to control something it really shouldn't.
