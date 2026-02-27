---
name: email-best-practices
description: Use when building email features, emails going to spam, high bounce rates, setting up SPF/DKIM/DMARC authentication, implementing email capture, ensuring compliance (CAN-SPAM, GDPR, CASL), handling webhooks, retry logic, or deciding transactional vs marketing.
---

# Email Best Practices

Guidance for building deliverable, compliant, user-friendly emails.

## Architecture Overview

```
[User] -> [Email Form] -> [Validation] -> [Double Opt-In]
                                              |
                                    [Consent Recorded]
                                              |
[Suppression Check] <-----------------[Ready to Send]
        |
[Idempotent Send + Retry] --------> [Email API]
                                       |
                              [Webhook Events]
                                       |
              +--------+--------+-------------+
              |        |        |             |
         Delivered  Bounced  Complained  Opened/Clicked
                       |        |
              [Suppression List Updated]
                       |
              [List Hygiene Jobs]
```

## Quick Reference

| Need to... | See |
|------------|-----|
| Set up SPF/DKIM/DMARC, fix spam issues | [Deliverability](./official-deliverability.md) |
| Build password reset, OTP, confirmations | [Transactional Emails](./official-transactional-emails.md) |
| Plan which emails your app needs | [Transactional Email Catalog](./official-transactional-catalog.md) |
| Build newsletter signup, validate emails | [Email Capture](./official-email-capture.md) |
| Send newsletters, promotions | [Marketing Emails](./official-marketing-emails.md) |
| Ensure CAN-SPAM/GDPR/CASL compliance | [Compliance](./official-compliance.md) |
| Decide transactional vs marketing | [Email Types](./official-email-types.md) |
| Handle retries, idempotency, errors | [Sending Reliability](./official-sending-reliability.md) |
| Process delivery events, set up webhooks | [Webhooks & Events](./official-webhooks-events.md) |
| Manage bounces, complaints, suppression | [List Management](./official-list-management.md) |

## Start Here

**New app?**
Start with the [Catalog](./official-transactional-catalog.md) to plan which emails your app needs (password reset, verification, etc.), then set up [Deliverability](./official-deliverability.md) (DNS authentication) before sending your first email.

**Spam issues?**
Check [Deliverability](./official-deliverability.md) first - authentication problems are the most common cause. Gmail/Yahoo reject unauthenticated emails.

**Marketing emails?**
Follow this path: [Email Capture](./official-email-capture.md) (collect consent) -> [Compliance](./official-compliance.md) (legal requirements) -> [Marketing Emails](./official-marketing-emails.md) (best practices).

**Production-ready sending?**
Add reliability: [Sending Reliability](./official-sending-reliability.md) (retry + idempotency) -> [Webhooks & Events](./official-webhooks-events.md) (track delivery) -> [List Management](./official-list-management.md) (handle bounces).
