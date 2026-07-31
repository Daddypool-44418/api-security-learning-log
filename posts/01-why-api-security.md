# Why I'm Learning API Security (And Why You Should Too)

If you've spent time doing web app pentesting, you've probably noticed something: most of the internet has quietly shifted from "websites" to "APIs with a UI bolted on top." Your banking app, your food delivery app, your favorite SaaS tool — under the hood, they're all just a frontend talking to a bunch of API endpoints.

And here's the uncomfortable part: **a lot of these APIs are tested far less rigorously than the UI ever is.**

That gap is exactly why I've started PortSwigger's **API Testing** module on Web Security Academy, and I'm going to document the whole journey here — lab by lab, concept by concept. If you're a fellow beginner in offensive security, or you're comfortable with the OWASP Top 10 for web apps but haven't touched APIs specifically, this series is for you. We're learning this together.

## Why APIs Deserve Their Own Attention

When I was doing standard web app assessments, I'd naturally gravitate toward the pages I could see — login forms, search bars, file uploads. But every one of those actions is quietly firing off API calls behind the scenes, and those calls often accept far more than the UI ever shows you.

A few reasons APIs are a goldmine for vulnerabilities:

- **They're often less visible.** No UI element points to every parameter an endpoint accepts — you have to go dig for it.
- **They assume trust between client and server** in ways that are easy to abuse. If the frontend only *shows* you an "edit your own profile" button, that doesn't mean the backend is actually checking whose profile you're editing.
- **Documentation is a double-edged sword.** Swagger/OpenAPI specs are great for developers, but they're also a roadmap for attackers if they're exposed.
- **They evolve faster than security testing does.** New endpoints get shipped constantly, and older, undocumented ones often get forgotten — but stay live.

If you're coming from a "find XSS and SQLi in a login form" mindset, API testing forces a shift: you're now thinking about *what the server trusts the client to say*, not just what the client can visually do.

## What the Module Actually Covers

PortSwigger's API Testing module is broken into a few core areas, and I'll be writing a dedicated post for each as I work through it:

1. **Discovering API endpoints** — finding documentation, hunting through JS files, using Burp's engagement tools, and identifying undocumented or "hidden" endpoints
2. **Understanding API structures** — REST vs GraphQL, how OpenAPI/Swagger specs work, and reading them like an attacker
3. **Finding hidden parameters** — endpoints often accept fields the docs never mention
4. **Mass assignment vulnerabilities** — what happens when an API blindly trusts client-supplied JSON fields
5. **Server-side parameter pollution** — manipulating how the server builds internal requests from your input
6. **Broken object-level authorization (BOLA/IDOR in APIs)** — the API-flavored version of "can I access data that isn't mine?"

I'll take each of these one at a time rather than trying to cram the whole module into one post — easier to actually absorb, and easier for you to follow along at your own pace.

## How I'll Be Writing This Series

A quick note on approach, since it matters for anyone following along: I won't be posting direct lab solutions or step-by-step "paste this payload and win" walkthroughs. Two reasons —

1. PortSwigger's labs are genuinely worth solving yourself. If I hand you the answer, you skip the part where the learning actually happens.
2. It's just not useful content. Anyone can copy a payload. What's actually valuable is understanding *why* it works.

So instead, each post will focus on:
- The vulnerability concept, explained plainly
- My own thought process — what I tried, what failed, what finally clicked
- Sanitized screenshots of my Burp Suite requests/responses where relevant
- How the concept maps to real-world API assessments, not just the lab environment

## Where This Is Going

By the end of this series, the goal isn't just "I finished a PortSwigger module." It's building a genuine mental model for how to approach an unfamiliar API during an assessment — how to map its attack surface, what assumptions to question, and where the interesting bugs tend to hide.

If you're learning alongside me, drop your own findings or questions — half the value of writing this stuff publicly is the conversation it starts.

**Next up:** API endpoint discovery — how to map out an API's attack surface before you've even sent your first malicious request.
