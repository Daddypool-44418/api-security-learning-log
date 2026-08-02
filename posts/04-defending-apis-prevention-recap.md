# Defending APIs: What All of This Actually Prevents

The last three posts covered the offense side — recon, hidden parameters, mass assignment. Before moving on to server-side parameter pollution, it's worth pausing on the flip side: **if you were building an API instead of testing one, what would you actually do differently?**

This post is a recap through that lens — tying together prevention advice for everything covered so far, rather than treating defense as an afterthought tacked onto a single vuln class.

## Don't Let Documentation Become Recon Material

Post 2 was largely about how much documentation hands an attacker for free — endpoint lists, parameter names, expected data shapes. The fix isn't "never document your API" (that just pushes the cost onto every developer who has to integrate with it). The fix is being deliberate about *where* documentation lives:

- If docs are meant for internal or partner use only, keep them behind authentication — not just an unlinked, "hard to guess" path. Obscurity isn't access control.
- If docs are meant to be public, treat them as a public-facing asset: keep them accurate and don't let outdated versions linger at old paths.
- Audit what your own documentation reveals. If it lists a field or endpoint your team forgot was still live, an attacker running the same recon steps from Post 2 will find it too.

## Lock Down What Each Endpoint Actually Accepts

A lot of what we covered in Post 2 — testing extra HTTP methods, switching content types — only works because APIs are often more permissive than they need to be. The fix is narrowing that surface deliberately:

- **Allowlist HTTP methods per endpoint.** If a route is only ever meant to be read, don't let the framework's defaults quietly leave `PUT`/`DELETE`/`PATCH` open just because no one explicitly closed them.
- **Validate content types strictly.** If an endpoint expects JSON, reject anything else outright rather than trying to gracefully handle whatever format shows up. Permissive parsing is exactly what let us probe for different behavior across content types in Post 2.
- **Keep error messages generic.** Detailed stack traces and verbose validation errors are convenient for developers and just as convenient for attackers doing recon. Log the detailed version server-side; return something minimal to the client.

## Fix Mass Assignment at the Root, Not the Symptom

Post 3 covered this one directly, but it's worth restating as a principle rather than a one-off fix: **allowlist what's writable, don't blocklist what isn't.**

Concretely, that means:
- Define an explicit schema or DTO (data transfer object) for what a given endpoint accepts, separate from your internal data model. Never bind a request body directly onto the same object your database uses.
- Any time a new sensitive field gets added to an internal model, it should be excluded from client input *by default* — not something a developer has to remember to block manually.
- Apply this consistently across every write endpoint, not just the ones that felt sensitive at the time. Mass assignment bugs often show up on the endpoint nobody thought to double-check.

## Treat Every API Version and Endpoint as Still Live

One thing that came up implicitly in Post 2 but is worth naming directly: attackers don't respect your idea of which endpoints are "current." An old `/api/v1/` route sitting next to a shiny new `/api/v2/` is still a fully working attack surface if it's still deployed — even if nothing in your frontend calls it anymore.

- Deprecate deliberately: if an old version is no longer supported, actually take it offline rather than leaving it reachable indefinitely.
- Apply the same authentication, authorization, and input validation standards to old versions as new ones, for as long as they're live.
- Don't assume "nothing links to it" means "nothing can reach it" — that assumption is exactly what recon in Post 2 was built to break.

## The Common Thread

Every fix above traces back to the same root cause: **APIs tend to trust the client more than they should, by default.** Frameworks are built for developer convenience — permissive method handling, flexible content-type parsing, auto-binding request bodies — and convenience defaults are rarely secure defaults.

None of this requires exotic tooling or a security team of ten. It requires treating "what does this endpoint actually need to accept" as a deliberate design question, rather than letting the framework's defaults answer it for you.

**Next up:** server-side parameter pollution — what happens when the *same* parameter shows up more than once, or in more than one place, and the server can't agree with itself about which one to trust.
