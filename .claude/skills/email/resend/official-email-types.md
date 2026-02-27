# Email Types: Transactional vs Marketing

Understanding the difference between transactional and marketing emails is crucial for compliance, deliverability, and user experience.

## Transactional vs Marketing: Key Differences

### Transactional Emails

**Definition:** Emails that facilitate or confirm a transaction the user initiated or expects. Directly related to an action the user took or are legal notices.

**Characteristics:**
- User-initiated or expected
- Time-sensitive and actionable
- Required for the user to complete an action
- Does not include promotional material
- Can be sent without explicit opt-in (with limitations)

**Examples:** Password reset links, order confirmations, account verification, OTP/2FA codes, shipping notifications.

### Marketing Emails

**Definition:** Emails sent for promotional, advertising, or informational purposes not directly related to a specific transaction.

**Characteristics:**
- Promotional or informational content
- Not time-sensitive to complete a transaction
- Require explicit opt-in (consent)
- Must include unsubscribe options
- Subject to stricter compliance requirements

**Examples:** Newsletters, abandoned cart, product announcements, promotional offers, company updates.

## Legal Distinctions

### CAN-SPAM Act (US)
- **Transactional:** Can send without opt-in if related to a transaction
- **Marketing:** Require opt-out mechanism, physical mailing address

### GDPR (EU)
- **Transactional:** Can send based on contract fulfillment or legitimate interest
- **Marketing:** Require explicit opt-in consent

### CASL (Canada)
- **Transactional:** Can send without consent if related to ongoing business relationship
- **Marketing:** Require express or implied consent

## Hybrid Emails: The Gray Area

Some emails mix transactional and marketing content. This isn't best practice and should be avoided.

**Best practice:** Keep transactional and marketing separate.

## Transactional Email Catalog

For a complete catalog of transactional emails and recommended combinations by app type, see [Transactional Email Catalog](./official-transactional-catalog.md).

**Quick reference - Essential emails for most apps:**
1. **Email verification** - Required for account creation
2. **Password reset** - Required for account recovery
3. **Welcome email** - Good user experience

The catalog includes detailed guidance for authentication-focused apps, newsletter/content platforms, e-commerce/marketplaces, SaaS/subscription services, financial/fintech apps, social/community platforms, developer tools/API platforms, and healthcare/HIPAA-compliant apps.

## Sending Infrastructure

### Separate subdomains

**Best practice:** Use separate sending subdomains for transactional and marketing emails.

**Implementation:**
- Use different subdomains (e.g., `t.yourdomain.com` for transactional, `m.yourdomain.com` for marketing)

## Related Topics

- [Transactional Emails](./official-transactional-emails.md) - Best practices for sending transactional emails
- [Marketing Emails](./official-marketing-emails.md) - Best practices for marketing emails
- [Compliance](./official-compliance.md) - Legal requirements for each email type
- [Deliverability](./official-deliverability.md) - Ensuring transactional emails are delivered
