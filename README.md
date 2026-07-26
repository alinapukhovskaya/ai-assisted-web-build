# Building a Production Website with Claude Code and Claude's Design Tool

A case study in directing AI tools to ship a real website end to end: brand and content decisions, an eleven-page static build, four interactive components rebuilt by hand, and the defects I caught and directed to a fix.

**Live site:** [coresocollaborative.com](https://coresocollaborative.com)

**My role:** direction, architecture, content, and quality control. Claude Code produced the code under my direction, and Claude's design tool produced the visual design. My work was deciding what the site had to be, specifying how to build it, and knowing when the output was wrong.

This piece sits alongside my knowledge-governance case studies for a reason. Those show how I structure institutional knowledge so AI can be trusted with it. This one shows the other half: I build with AI tools hands-on, and the site itself carries the same content-governance discipline, including consent behavior, a privacy and terms structure, and per-page form handling.

> This is a curated case study, not the deployable source. It documents the process and the judgment calls. Form endpoints, calendar links, and full page source are omitted.

![The Coreso Collaborative home page](assets/01-home.png)

---

## What This Repository Contains

- **This file:** the case study narrative: the brief, the working method, the build pipeline, the defects caught, and what the work demonstrates.
- **assets/:** screenshots of the live site.

---

## The Brief

Coreso Collaborative is my independent consulting practice, developed alongside my MBA. The site had to carry a real service offering across eleven pages, self-host its fonts and assets, run without a build server, and stay maintainable by one person. There was no agency and no developer, and nothing was to depend on a framework runtime once the site was live.

The constraint that shaped everything came from the starting material. The visual design left Claude's design tool as `.dc.html` files that depend on a React runtime loaded from a content delivery network at page load. That format is fine for previewing a design. It is not something you host as a real website, because it is slow, fragile, and dependent on a third party staying available. The design was right; the delivery format was not.

So the core problem was a translation problem: take a React-runtime design export and turn it into plain, self-contained HTML that behaves identically, without rebuilding eleven pages by hand.

---

## How I Worked with Claude Code

I treated Claude Code as an execution partner, not an oracle. The pattern on every step was the same:

1. **I specified the outcome and the constraints:** static output, no external dependencies at runtime, identical rendering, self-hosted fonts, one uploadable artifact.
2. **Claude Code proposed and built the pipeline.** I reviewed the approach before letting it run and redirected when the approach was wrong.
3. **I verified the result against reality, not against the claim.** Generated code that looks finished is not finished. Every page was checked in a real browser, and several outputs reported as complete were quietly broken in ways only inspection caught.

The value I added was not typing. It was framing the problem correctly, holding the quality bar, and having enough technical judgment to notice when a confident answer was wrong.

---

## The Build Pipeline

```mermaid
flowchart TD
    A["Claude design tool<br/>11 .dc.html exports<br/>(React runtime, CDN-loaded)"] --> B["Serve the design folder locally"]
    B --> C["Render each page in headless Chrome<br/>freeze the rendered DOM to static HTML"]
    C --> D["Strip the React runtime and design-tool scripts<br/>keep only what the page needs to run"]
    D --> E["Rewrite design-tool links<br/>to clean routes such as about.html and faq.html"]
    E --> F["Rebuild the 4 interactive components<br/>in plain JavaScript"]
    F --> G["Self-host fonts and assets<br/>correct the asset paths"]
    G --> H["Verify every page in a real browser<br/>drive the forms end to end"]
    H --> I["Static site: 11 pages, no build step,<br/>no runtime dependencies"]
```

In plain terms: I had the design rendered by a real browser, captured the finished HTML, then removed everything the page did not need to stand on its own. Anything interactive that could not survive that step was rebuilt by hand.

The result is eleven pages of plain HTML, five small scripts, self-hosted fonts, one favicon, and one file to upload. There is no server code, no framework, and no dependency on a content delivery network.

![One of the service pages built through the pipeline](assets/02-service-page.png)

---

## Defects I Caught and Directed to a Fix

This section is the point of the case study. Directing AI to generate code is straightforward. Knowing when it has produced something subtly broken is the real skill.

### 1. A dead cookie banner baked into all eleven pages

Freezing the rendered page captured the cookie-consent banner as it looked at that instant, as plain markup with no behavior attached. Every page shipped with a banner that looked real but did nothing: Accept and Decline were inert, it ignored any saved choice, and it reappeared on every page. On the privacy page it had been captured twice. Meanwhile the real script was rendering a second, working banner underneath it.

I diagnosed it and directed the fix: strip every baked-in banner block from the static HTML so the banner is only ever created by its script at runtime. A plain find-and-replace would not do it, because the blocks contain nested elements, so the fix had to match them structurally. I also set a standing check: after any future rebuild, confirm the count of baked-in banners in the static files is zero.

### 2. Self-hosted fonts failing without any visible error

The font stylesheet pointed at a path one level too deep. The fonts returned a not-found response, and every visitor silently fell back to a default serif. Nothing errored visibly. The site simply looked wrong to everyone. I caught it by checking the actual network requests rather than trusting the page to report a problem, and I corrected the paths.

### 3. The correct form key hidden behind a decoy

Each form page carried two candidate submission keys: a real one set as a component property, and a fallback in the code. The fallback was the obvious one to grab, and it was the wrong one. Sending real form submissions through it would have sent leads nowhere. I traced which key the design actually used on each page and wired the correct one per page, because the three forms each used a different key.

### 4. A consent banner nobody could read

The redesigned banner rendered as cream text on a cream background, which made it effectively invisible. I redirected it to a full-width dark bar with legible contrast, short plain wording, and a close control that also records the choice so it does not reappear on the next page.

![The redesigned consent banner: a dark full-width bar with legible Decline and Accept controls](assets/04-consent-banner.png)

The through-line is consistent: in every case the generated output reported success. The difference between a shipped defect and a working site was inspection and judgment, not the generation itself.

---

## The Four Interactive Components, Rebuilt by Hand

A frozen snapshot of a React application keeps how it looked, not how it worked. Four pieces had real logic that did not survive the freeze, so I had them rebuilt in plain JavaScript and tested each one end to end:

- **Contact form** with validated submission
- **Home page banner and popup**
- A short **two-minute reliability quiz**
- A longer **reliability-check intake** form

Each was driven through a full submission in an automated browser before launch, with the submission endpoint mocked so testing never sent real leads.

![The reliability quiz landing screen, one of the four rebuilt components](assets/03-interactive.png)

---

## What This Work Demonstrates

- **AI fluency with judgment.** I direct Claude Code and Claude's design tool to real output, and I catch what they get wrong. The value is in the second half of that sentence.
- **End-to-end ownership.** Brand and content decisions, architecture, build, debugging, and deployment ran through one accountable person.
- **Content governance applied to a live site.** Consent behavior, privacy and terms, per-page form handling, and a maintainable structure are the same content-governance discipline I bring to knowledge work.
- **A bias toward verifying against reality.** Every page was checked in a real browser, every form was driven end to end, and standing checks were written so the same class of defect cannot return.

---

## Tools

- **Claude Code** for the build pipeline, the static conversion, and the debugging
- **Claude's design tool** for the visual design and page layouts
- Headless Chrome to render and freeze the design output
- Plain HTML, CSS, and JavaScript in production, with no framework, no build step, and no runtime dependencies

---

*Coreso Collaborative is an independent consulting practice. This repository documents how the practice's website was built with AI tools. It is a curated case study, and sensitive configuration has been omitted.*
