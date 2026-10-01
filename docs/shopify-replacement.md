# Replacement fulfillment (Shopify)

Status: planned. No node implements this yet.

## Flow

1. Reviewer presses Approve on a case with compensation above 0.
2. Workflow claims the case by setting status to `sending`.
3. Workflow creates a zero-value draft order.
4. The returned `invoice_url` replaces `{{INVOICE_URL}}` in the stored draft.
5. Workflow sends the email.
6. Workflow sets status to `approved` and updates the Slack card.

## API call

```
POST https://{shop}/admin/api/{version}/draft_orders.json
```

```json
{
  "draft_order": {
    "line_items": [
      { "variant_id": 0, "quantity": 1 }
    ],
    "applied_discount": {
      "value_type": "percentage",
      "value": "100.0",
      "title": "Replacement"
    },
    "email": "customer@example.com",
    "note": "Case CS-YYYYMMDD-XXXXXX"
  }
}
```

The response contains `invoice_url`, which goes into the email.

## Guards

- **Idempotency.** Store the draft order ID on the case row. If it exists,
  reuse it. A double click cannot create two orders.
- **Quantity cap.** Quantity is at most the units visible in the photo, or
  the quantity the customer stated, whichever is lower.
- **Level mapping.** The discount percentage follows the case compensation
  level, not a fixed 100.
- **Claim before send.** Status moves to `sending` before any external call.
  A retry on `sending` resumes, it does not restart.

## Failure handling

| Failure | Behavior |
|---|---|
| Order API fails | Status back to `pending_approval`, error in `last_error`, ephemeral Slack message |
| Email fails after order created | Keep order ID, status `send_failed`, retry sends only the email |
| Status write fails after send | Status stays `sending`, alert fires, no automatic retry |

## Open questions

- SKU to variant ID lookup: cached map or live product query.
- Whether non-marketplace orders get a discount link instead of a draft order.
- Whether the invoice URL needs an expiry note in the email.