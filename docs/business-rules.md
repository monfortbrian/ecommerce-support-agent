# Business rules

All money decisions are made in code. The model supplies signals only.

## Precedence

Evaluated top to bottom. The first match wins.

| # | Condition | Compensation | Draft |
|---|---|---|---|
| 1 | Classification parse failure | none | manual review |
| 2 | Rude or gaming | 0% | polite decline |
| 3 | Cause not determined | none | request evidence |
| 4 | Marketplace channel | 100% | replacement |
| 5 | Everything else | `max(model proposal, 30)` | replacement |

Rule 5 currently always returns 30. The model proposal field is missing
from the classification schema.

## Cause determined

True only when all of these hold:

- the email reports a product fault
- the customer did not say a photo is difficult
- classification did not fall back to manual review
- a root cause ID was returned

## Evidence rule

Photos are not requested by default. They are requested only when the
description is not enough to find the root cause.

Vision rules:

- `fault_visible` is true only if the exact spot and defect can be named.
- A receipt identifies the product and proves nothing about damage.
- A detached part with no body in frame returns low confidence, no SKU.
- Low confidence sets manual review on the card.

## Attachments

Only `image/*` is analyzed. Other types are listed as unsupported, treated
as no photo evidence, and flagged on the card.

## Controlled terminology

| Rule | Value |
|---|---|
| Banned phrases | `free of charge`, `defective` |
| Required instruction phrase | set in config, used in root cause text |
| Instruction video | one canonical URL from config |
| Link allowlist | domains in config, others warn |

## Reply constraints

- Greeting and sign-off follow the fixed template.
- No absolute guarantees.
- Replies go out in the customer's language. The English version is always
  stored next to the native one for review.

## Prompt injection

Email text sits inside a nonce-delimited fence and is labeled as data. The
prompt tells the model to ignore any instruction inside it.
`manipulation_attempt` is true only for a real takeover attempt. An angry
customer is not manipulation.