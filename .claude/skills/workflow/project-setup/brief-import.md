# External Brief Import

## Overview

Users often create project briefs in Claude Projects (using uploaded docs, graphics, design files, etc.) before starting a SkillKit project. This flow lets them paste that brief into the setup wizard so it enriches the internal `projectbrief.md` rather than starting from scratch.

## When to Ask

**After environment detection (Step 1), before the project type question.**

Ask this question:

```
Do you have a project brief you'd like to share? For example, from Claude Projects, a Google Doc, or any description of what you're building.

A. Yes, let me paste it
B. No, let's start from scratch
```

**If yes:** Ask them to paste it:

```
Paste your project brief below. I'll use it to set up your project faster -
you'll only need to answer a few remaining questions about deployment and tooling.
```

**If no:** Continue with the normal questionnaire flow (no changes).

## How to Process the Pasted Brief

### Step 1: Read and Understand

Read the entire pasted brief. Identify what information is present and what's missing for the setup.

### Step 2: Extract and Map

Look for these categories and map them to the internal projectbrief format:

| Look for in the pasted brief | Maps to internal section | Skips question |
|------------------------------|--------------------------|----------------|
| Project description, what it does, purpose | `## Project Overview` -> description | Web app Q1, Content Q4 |
| Target users, audience, who it's for | `## Project Overview` -> users | Web app Q2 |
| Application type (web, mobile, extension) | `## Tech Stack` -> framework | Web app Q3 |
| Feature list, functionality requirements | `## Core Features (MVP)` | Web app Q4 |
| Technology preferences, stack mentions | `## Tech Stack` | Web app Q6 (partially) |
| Timeline or launch date | `## Project Overview` -> timeline | Web app Q7 |
| Design system (colors, fonts, spacing, tokens) | `## Design System` | - |
| Brand guidelines (logo, voice, tone) | `## Brand` | - |
| Content structure (pages, sections, navigation) | `## Content Structure` | - |
| Wireframes or layout descriptions | `## Pages & Layouts` | - |
| SEO requirements | `## SEO` | - |
| Content types, collections, taxonomies | `## Content Model` | - |
| Competitor references, inspiration sites | `## References` | - |

### Step 3: Confirm What Was Extracted

Show the user a summary of what you found:

```
I found the following in your brief:

- Project: [extracted description]
- Target users: [extracted users]
- Key features: [extracted features list]
- Design system: [colors, fonts, etc. if found]
- [any other extracted sections]

I still need to ask you about:
- [list of remaining questions - typically deployment, package manager, code review]

Does this look right? I'll ask the remaining questions next.
```

**Wait for confirmation before proceeding.** If the user corrects something, update accordingly.

### Step 4: Ask Only Remaining Questions

Continue with the appropriate questionnaire (web app or content website), but **skip questions that were already answered** by the external brief.

**Questions that are always asked** (never pre-filled from external brief):
- Package manager (SkillKit-specific)
- Deployment platform(s) (SkillKit-specific)
- Code review / Greptile (SkillKit-specific)
- Experience level (SkillKit-specific)

**Questions that can be skipped** if covered by the brief:
- Project description (Q1 / Content Q4)
- Target users (Q2)
- Application type (Q3)
- Key features (Q4)
- Technology preferences (Q6)
- Timeline (Q7)

### Step 5: Determine Project Type from Brief

If the brief clearly describes one of these, skip the project type question:

| Brief mentions... | Project type |
|-------------------|-------------|
| CMS, blog, marketing site, portfolio, content pages | Content website |
| User accounts, dashboard, SaaS, payments, login | Web application |
| Astro, Payload, static site | Content website |
| Database, API, authentication, real-time | Web application |

If unclear, still ask the project type question.

## Merge Rules

When generating the final `projectbrief.md`:

1. **Use the internal template as the skeleton** - section headings, structure, and format come from our template
2. **Fill sections with external brief content** - rich descriptions, design tokens, content structure from the pasted brief
3. **Add custom sections for extra info** - if the brief contains info that doesn't fit any template section (brand guidelines, competitor analysis, etc.), add it as a new section at the end
4. **Preserve the pasted brief's detail level** - if the external brief has detailed design tokens (exact hex colors, font sizes, spacing scale), keep all that detail. Don't summarize it away.
5. **Mark unfilled sections as TBD** - sections not covered by the brief or questionnaire get `[TBD - discuss before implementation]`
6. **Never contradict the external brief** - if the brief says "use blue #2563EB as primary color", don't override it with a default

## Content Website Special Case

For content websites, there may be THREE sources of information:

1. **Template projectbrief** - pre-filled by the Astro + Payload template (collections, routes, components)
2. **External brief** - pasted by the user (design, content, brand)
3. **Questionnaire answers** - deployment, code review

**Merge priority:** External brief > Template projectbrief > Questionnaire answers

- Template provides the technical foundation (collection schemas, routes, component library)
- External brief provides the project-specific context (what the website is about, design, content)
- Questionnaire fills in the remaining gaps (deployment, tooling)

## Example Flow

```
Claude: "Do you have a project brief you'd like to share?"
User: "Yes"
Claude: "Paste your project brief below."
User: [pastes 2-page brief about a design agency website with colors, fonts, page structure]

Claude: "I found the following in your brief:

- Project: Portfolio website for a design agency
- Target audience: Potential clients and creative professionals
- Pages: Home, Work (portfolio), About, Team, Blog, Contact
- Design system: Primary blue #2563EB, Inter font, 8px spacing grid
- Content types: Projects, Team Members, Blog Posts

I still need to ask you about:
- Where to deploy (hosting platform)
- Package manager preference
- Code review setup

Does this look right?"

User: "Yes!"
Claude: "Where do you want to deploy the backend..."
[continues with only the remaining questions]
```
