# E-commerce Support Agent

Human-in-the-loop support automation. It reads customer emails, diagnoses
the issue, drafts a reply and posts a Slack card. Nothing is sent to a
customer without explicit approval.

## Design principles

- AI classifies and drafts. Code decides compensation.
- No automatic customer emails or orders.
- Photos are requested only when the description is not enough.
- Parse failures fail closed to manual review.
- Email content is treated as data, not instructions.

## Workflows

| File | Purpose | Status |
|---|---|---|
| intake-diagnosis | Inbox to Slack card | Working in test mode |
| slack-actions | Approve, edit, park, forward | Scaffold |
| error handler | Failure alerts | Planned |
| Watchdog | Dropped-email checks | Planned |

## Business rules

- Marketplace orders override goodwill rules (100% replacement).
- Direct orders: floor of 30%, higher on model proposal.
- Rude or gaming behaviour: 0%.
- Unknown cause: request evidence instead of compensating.
- Fixed terminology. "free of charge" and "defective" are banned.

## Replacement fulfillment (Shopify)

On approval, a zero-value draft order is created through the Admin API
and its invoice URL replaces `{{INVOICE_URL}}` in the email.

## Setup

1. Import both workflows into n8n.
2. Create credentials: Slack, Microsoft Graph, Azure OpenAI, Shopify.
3. Copy `config/config.example.json` values into the Config node.
4. Set `test_mode` to `true` and run against `tests/fixtures.json`.
5. Set the Slack interactivity URL to the actions webhook.