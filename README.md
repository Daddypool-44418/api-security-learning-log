# API Security Learning Log

A public log of my journey through [PortSwigger's API Testing module](https://portswigger.net/web-security/api-testing) on Web Security Academy — written to document what I'm learning and to help other beginners in offensive security follow the same path.

I'm a junior offensive security professional (web app pentesting, Active Directory exploitation) building out API-specific testing skills. This repo is where I break down each concept as I learn it — not lab walkthroughs, but the actual thought process behind each vulnerability class.

**No lab spoilers.** You won't find copy-paste payloads or step-by-step solutions to PortSwigger's labs here — solving them yourself is where the actual learning happens. What you will find is how I think through each concept, what I tried, what failed, and how it maps to real-world API assessments.

## Posts

Posts follow the PortSwigger API Testing module in order, so you can read along with the module section-by-section.

| # | Topic | Covers | Status |
|---|-------|--------|--------|
| 01 | [Why I'm Learning API Security](posts/01-why-api-security.md) | Intro — why APIs need their own testing mindset | ✅ Published |
| 02 | [API Recon: Finding the Attack Surface](posts/02-api-recon-discovery.md) | API recon, documentation discovery, identifying endpoints, interacting with endpoints (HTTP methods, content types, Intruder) | ✅ Published |
| 03 | Finding Hidden Parameters & Mass Assignment | Finding hidden parameters, mass assignment vulnerabilities | 🔜 Coming up |
| 04 | Server-Side Parameter Pollution | Query string, REST paths, structured data formats | 🔜 Coming up |
| 05 | OWASP API Top 10 Alignment | How this module maps to the OWASP API Security Top 10 2023 | 🔜 Coming up |

## About Me

Junior offensive security professional focused on web application pentesting and Active Directory exploitation. Currently expanding into API-specific security testing.

- LinkedIn: [add your profile link]
- Bugcrowd: [add your profile link]

## Why This Repo Exists

Most security learning happens in private — a lab gets solved, a certificate gets earned, and none of the actual thinking behind it is visible to anyone else. This repo is an attempt to make that thinking visible: partly to solidify my own understanding by writing it out, and partly because I wish more people documented their "still learning" phase instead of only their finished credentials.

If you're working through the same PortSwigger module, feel free to open an issue or reach out — happy to compare notes.
