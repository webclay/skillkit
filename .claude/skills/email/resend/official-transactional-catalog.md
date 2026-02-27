# Transactional Email Catalog

A comprehensive catalog of transactional emails organized by category, plus recommended email combinations for different app types.

## Email Combinations by App Type

### Authentication-Focused App
**Essential:** Email verification, Password reset, OTP/2FA codes, Security alerts, Account update notifications
**Optional:** Welcome email, Account deletion confirmation

### Newsletter / Content Platform
**Essential:** Email verification, Password reset, Welcome email, Subscription confirmation
**Optional:** OTP/2FA codes, Account update notifications

### E-commerce / Marketplace
**Essential:** Email verification, Password reset, Welcome email, Order confirmation, Shipping notifications, Invoice/receipt, Payment failed notices
**Optional:** OTP/2FA codes, Security alerts, Subscription confirmations

### SaaS / Subscription Service
**Essential:** Email verification, Password reset, Welcome email, OTP/2FA codes, Security alerts, Subscription confirmation, Subscription renewal notice, Payment failed notices, Invoice/receipt
**Optional:** Account update notifications, Feature change notifications

### Financial / Fintech App
**Essential:** Email verification, Password reset, OTP/2FA codes (required), Security alerts (all types), Account update notifications, Transaction confirmations, Invoice/receipt, Payment failed notices
**Optional:** Welcome email, Compliance notices

### Social / Community Platform
**Essential:** Email verification, Password reset, Welcome email, Security alerts
**Optional:** OTP/2FA codes, Account update notifications, Activity notifications

### Developer Tools / API Platform
**Essential:** Email verification, Password reset, OTP/2FA codes, Security alerts, API key notifications, Subscription confirmation, Payment failed notices
**Optional:** Welcome email, Usage alerts, Feature change notifications

### Healthcare / HIPAA-Compliant App
**Essential:** Email verification, Password reset, OTP/2FA codes (required), Security alerts (all types), Account update notifications, Appointment confirmations
**Optional:** Welcome email, Compliance notices

**Note:** Healthcare apps have strict requirements. Emails should contain minimal PHI and link to secure portals for sensitive information.

## Full Email Catalog

### Authentication & Security
- **Email Verification:** Send immediately after signup or email change. Include verification link/code, expiration (24-48h), resend option.
- **OTP / 2FA Codes:** Send immediately. Large, easy-to-read code, expiration (5-10 min), security warnings.
- **Password Reset:** Send immediately. Reset link with token, expiration (1 hour), "I didn't request this" link.
- **Security Alerts:** Send immediately on security events. What happened, when, location/IP, action steps.

### Account Management
- **Welcome Email:** After successful verification. Next steps, key features, support contact. Must not be promotional.
- **Account Update Notifications:** After settings change. What changed, when, revert option, security notice.

### E-commerce & Transactions
- **Order Confirmations:** Immediately after order. Order number, items, pricing, shipping address, tracking link.
- **Shipping Notifications:** When order ships. Tracking number, carrier info, expected delivery.
- **Invoices and Receipts:** After payment. Invoice number, amount, payment method, downloadable PDF.

### Subscriptions & Billing
- **Subscription Confirmations:** After subscribe/change. Plan details, billing amount/frequency, next billing date.
- **Subscription Renewal Notices:** 3-7 days before renewal. Renewal date, amount, update payment link, cancel option.
- **Payment Failed Notices:** Immediately after failure. What happened, resolution steps, consequences, update payment link.

## Related Topics

- [Email Types](./official-email-types.md) - Understanding transactional vs marketing
- [Transactional Emails](./official-transactional-emails.md) - Best practices for sending transactional emails
- [Compliance](./official-compliance.md) - Legal requirements for each email type
