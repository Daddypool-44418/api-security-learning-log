# Hidden Parameters and Mass Assignment: When the API Trusts You Too Much

In Post 2, we mapped out an API's attack surface — documentation, endpoints, HTTP methods, content types. That gets you a list of *what exists*. This post is about the next question: **what does each endpoint secretly accept that nobody meant for you to send?**

That's the world of hidden parameters — and its most common, most exploitable consequence: mass assignment.

## What "Hidden Parameters" Actually Means

During recon, you'll often come across parameters the API supports but the documentation never mentions and the frontend never sends. These aren't necessarily secrets someone deliberately buried — more often they're leftovers: internal fields, deprecated options, admin-only toggles, or fields a developer added for convenience and never bothered to lock down.

The interesting part isn't just finding these parameters — it's testing whether the server actually *does* something different when you supply them.

Burp has a few tools purpose-built for this hunt:

- **Intruder** — cycles through a wordlist of common parameter names, either replacing existing params or appending new ones to see how the server reacts
- **Param Miner** — a Burp extension that can guess tens of thousands of parameter names automatically, using context clues from the target itself rather than a purely generic list
- **Content discovery** — surfaces content (including parameters) that isn't linked anywhere you could browse to normally

The common thread: don't rely on generic wordlists alone. Fold in terms specific to the application — field names, naming conventions, and vocabulary you picked up during recon in Post 2. A tailored guess list finds far more than a stock one.

## Where Mass Assignment Comes From

Mass assignment (sometimes called auto-binding) happens when a framework automatically maps whatever fields you send in a request straight onto an internal object — without checking whether you *should* be allowed to set those fields.

Here's the shape of it, using a generic example rather than any specific lab:

Say there's an endpoint like `PATCH /api/profile` meant to let a user update their own display name and bio. The intended request looks like:

```json
{
  "displayName": "alex",
  "bio": "security enthusiast"
}
```

Now imagine a separate `GET /api/profile/me` request — used elsewhere in the app to *read* your profile — returns a fuller object:

```json
{
  "id": 4821,
  "displayName": "alex",
  "bio": "security enthusiast",
  "accountTier": "free"
}
```

That extra `accountTier` field you never sent is a strong hint: it's a real field on the internal object, just not one the frontend normally lets you *write* to. If the backend is naively binding whatever JSON it receives to that same internal object, there's a reasonable chance the PATCH endpoint will happily accept `"accountTier": "premium"` too — even though no button in the UI ever offered you that option.

This is exactly the kind of discrepancy worth hunting for: **compare what a GET response reveals against what a PATCH/POST/PUT request is willing to accept.** Any field visible in a read operation but absent from the documented write operation is a candidate worth testing.

## How You'd Go About Testing It (Conceptually)

Without walking through a specific lab step-by-step, the general method looks like this:

1. **Spot the discrepancy** — compare a full object (from a GET) against the fields a write endpoint documents or the frontend actually sends.
2. **Add the missing field back in** — include the suspected hidden field in your write request and observe whether the response or app behavior changes at all.
3. **Test both a valid and an invalid value** — if a field genuinely controls something server-side, an invalid value (wrong type, out-of-range number, malformed string) often triggers different behavior than a valid one. A different error, or no error at all, tells you the server is actually parsing and validating that field — which means it's live and worth pushing further.
4. **Only then push toward the value that actually matters** — once you've confirmed the parameter is real and writable, the exploit itself is often just supplying the value you weren't supposed to be able to set.

The classic version of this in real-world writeups involves a hidden `isAdmin`-style boolean quietly bound to a user's own profile-update endpoint. I won't walk through the exact lab solution — but if you understand the four steps above, you already understand *why* that class of bug works, regardless of what the field happens to be named.

## Why This Keeps Happening

The root cause is almost always the same: developers optimize for convenience. Auto-binding request bodies to internal objects is fast to write and easy to maintain — you don't have to manually map every field. But that convenience quietly hands the client control over anything the object contains, unless the developer explicitly restricts it.

Which is also the fix: **allowlist, don't blocklist.** Explicitly define which fields a client is allowed to set, and reject everything else by default — rather than trying to remember every sensitive field to exclude. A blocklist only works until someone adds a new sensitive field to the model and forgets to update the exclusion list. An allowlist fails safe by default.

## Why This Fits Right After Recon

This is why Post 2 and this post belong in sequence: recon gives you the raw material — the endpoints, the observed request/response pairs, the vocabulary of the app — and hidden-parameter hunting is what you *do* with that material. Without solid recon, you're guessing at random field names. With it, you're testing specific, informed hypotheses about what the server might be blindly trusting.

**Next up:** server-side parameter pollution — what happens when the *same* parameter shows up more than once, or in more than one place, and the server can't agree with itself about which one to trust.
