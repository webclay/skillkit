---
name: domain-brainstormer
description: Domain name generator and availability checker. Trigger words - domain, domain name, find domain, check domain, domain available, domain ideas
---

# Domain Name Brainstormer

Generate creative domain name suggestions and check availability across popular TLDs.

## When to Use This Skill

- User asks to find a domain name
- User is launching a new project or product
- User mentions "find me a domain" or "check if X is available"
- User needs domain suggestions for their app idea

## Instructions

### Step 1: Understand the Project

Before brainstorming, gather context:
- What does the project/product do?
- Who is the target audience?
- What's the brand personality? (professional, playful, technical, etc.)
- Any keywords or themes to incorporate?

### Step 2: Generate Domain Ideas

Create 10-15 domain suggestions using these strategies:

**Naming Patterns:**
- Direct description: `taskflow`, `quicksend`
- Compound words: `mailbird`, `dropbox`
- Action + noun: `snapchat`, `slideshare`
- Prefix/suffix: `getflow`, `appify`, `usecanvas`
- Abbreviations: `npm`, `aws`
- Invented words: `spotify`, `hulu`

**Quality Criteria:**
- Under 15 characters (ideal)
- Easy to pronounce
- Easy to spell (no confusion when spoken)
- No hyphens (harder to share verbally)
- Memorable and brandable

### Step 3: Check Availability

For each suggestion, check availability across TLDs:
- `.com` - Best for established businesses, most trusted
- `.io` - Popular for tech startups
- `.dev` - Great for developer tools
- `.ai` - Perfect for AI products
- `.app` - Good for mobile apps
- `.co` - Alternative to .com

**How to check:**
```bash
# Using whois (if available)
whois example.com | grep -i "No match"

# Or recommend user check manually at:
# - namecheap.com
# - name.com
# - domains.google
```

### Step 4: Present Recommendations

Organize results by availability:

**Available:**
| Domain | TLD | Why It Works | Est. Price |
|--------|-----|--------------|------------|
| quicksend.io | .io | Action + noun, memorable | $30-40/yr |
| getflow.dev | .dev | Clear purpose, developer-friendly | $12-15/yr |

**Taken (with alternatives):**
| Domain | Status | Alternative |
|--------|--------|-------------|
| taskflow.com | Taken | taskflow.io, usetaskflow.com |

## TLD Recommendations

| TLD | Best For | Price Range |
|-----|----------|-------------|
| `.com` | Established businesses, broad audience | $10-15/yr |
| `.io` | Tech startups, SaaS products | $30-50/yr |
| `.dev` | Developer tools, open source | $12-15/yr |
| `.ai` | AI/ML products | $50-80/yr |
| `.app` | Mobile applications | $15-20/yr |
| `.co` | Startups, short names | $25-35/yr |

## Examples

**User:** "I'm building a tool that helps developers write better documentation"

**Response:**
Based on your documentation tool for developers, here are my recommendations:

**Top Picks (Available):**
1. `docwrite.dev` - Clear purpose, .dev signals developer focus
2. `beterdocs.io` - Memorable, tech-oriented
3. `writedocs.app` - Action-oriented, simple

**Creative Options:**
4. `docflow.io` - Implies smooth workflow
5. `autodocs.dev` - Suggests automation
6. `cleardocs.com` - Emphasizes clarity

**Premium (if budget allows):**
7. `docs.ai` - Short, memorable, premium

I'd recommend `docwrite.dev` as your primary choice - it's:
- Short (8 chars)
- Clearly describes what you do
- .dev TLD signals developer audience
- Available at ~$12/year

## Tips

- Check social media handle availability too (@docwrite on Twitter, etc.)
- Consider how the domain sounds when spoken aloud
- Avoid names that could be misspelled easily
- Register multiple TLDs to protect your brand (.com + .io)
- Premium domains can be worth it for the right brand

## How to Verify

### Quick Checks
- Use WHOIS lookup to confirm availability
- Check that suggested domain renders correctly in browser
- Verify no trademark conflicts (USPTO, EUIPO)

### Manual Verification
- Visit domain registrar (Namecheap, Google Domains)
- Confirm exact spelling and TLD is available
- Check social media handles match

### Common Issues
- Domain shows available but is "premium priced" - registrar markup
- Domain recently expired - may be in redemption period
- Trademark conflict - search USPTO before purchasing
