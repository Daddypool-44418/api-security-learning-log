# Server-Side Parameter Pollution: When the Server Can't Agree With Itself

The last few posts covered finding endpoints, finding hidden parameters, and abusing how the server binds request data to internal objects. This post covers a related but distinct problem: what happens when **the parameter you control gets reused internally**, in a context you never see, and the server ends up trusting your input somewhere it shouldn't.

That's server-side parameter pollution (SSPP) — and unlike most of what we've covered so far, it's less about finding a hidden field and more about understanding what your input turns into *after* it leaves your hands.

## The Core Idea

A lot of APIs don't just take your request and use it directly — they take pieces of it and build an entirely new internal request to some other system: a downstream API, an internal microservice, a database query, or another part of the same application. That internal request is constructed by the server, using your input as ingredients.

SSPP happens when the server builds that internal request by naively concatenating or reusing your input, without properly accounting for characters that have special meaning in the format it's building. If you can inject something that changes how the internal request gets parsed, you can potentially add parameters that were never meant to be attacker-controlled — because as far as the receiving system is concerned, it's just another parameter in a trusted, server-generated request.

The name makes sense once you see it this way: you're not polluting your own request. You're polluting a *second* request the server builds on your behalf, server-side.

## Where It Shows Up

**In the query string.** If your input gets inserted into a URL the server constructs for an internal call, and that input isn't properly encoded, characters like `&` can let you inject an entirely new parameter into that internal request. Something as simple as a value containing `&admin=true` could, if reflected unescaped into a server-built query string, result in a second parameter appearing that was never part of the original design.

**In the URL path.** Similarly, if part of your input gets inserted into a path segment of an internally constructed URL, path-traversal-style characters or unexpected path separators can shift what the internal system interprets as the actual endpoint being called — potentially routing your request to something you shouldn't be able to reach directly.

**In structured data formats.** The same core issue applies to JSON, XML, or other structured formats the server might build internally using your input. If a value you control gets inserted into a structured document without being properly escaped for that format, you may be able to break out of the intended field and inject an entirely new key/value pair, or even restructure the document. A string value that isn't sanitized for JSON, for instance, could close out the field it belongs in and open a new one that wasn't in the server's original template.

## Why This Is Different From Everything Else in This Series

Recon (Post 2) and hidden parameters/mass assignment (Post 3) are both about **what the API is willing to accept directly from you.** SSPP is about **what your input becomes after the server processes it and hands it somewhere else.** You're not exploiting a parameter the API exposes — you're exploiting a parameter-construction process the API performs internally, that you were never supposed to be able to see, let alone influence.

That also makes it harder to spot from the outside. There's no missing field to notice in a GET response, no extra HTTP method to test. The clues tend to be more indirect: unexpected behavior changes when your input contains characters like `&`, `=`, `"`, or path separators — especially if those changes suggest the server is treating your input as more than one value, or routing it somewhere unexpected.

## The General Approach

Without walking through a specific lab, the mindset for testing this looks like:

1. **Look for places where your input seems to trigger a second, server-side request** — response timing differences, behavior that implies a downstream call, or documentation/errors that reveal an internal architecture with multiple services talking to each other.
2. **Try injecting format-breaking characters relevant to the likely format** — `&` for query strings, `/` or `../` for paths, `"` and `{}` for JSON — one at a time, observing whether behavior changes at all.
3. **Watch for signs your injected value got interpreted as structure, not data** — an unexpected field appearing in a response, a different error message, or evidence that a second parameter was processed alongside the one you intended to send.
4. **Only once you've confirmed injection works, work out what parameter is actually worth adding** — same principle as mass assignment: confirm the mechanism first, then decide what's worth injecting.

## Why This Matters for API Testing Specifically

APIs are especially prone to this because they're often built as thin layers in front of other services — gateways, aggregators, backend-for-frontend patterns. Practically every API call is quietly triggering other calls behind the scenes, which means there are more opportunities for your input to get reused somewhere unsafe than in a traditional monolithic web app.

**Next up:** wrapping up the series by mapping everything covered — recon, hidden parameters, mass assignment, and SSPP — onto the OWASP API Security Top 10, so the whole series ties back to an industry-recognized framework.
