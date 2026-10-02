# Going live

Switch one integration at a time. Run the demo after each step.

## Order

1. **Classification.** Create the Azure OpenAI credential (header auth). Set the URL and deployment in Config. Set `mock_classify` to `false`. Add `proposed_compensation_level` to the schema or remove it from the rules code.
2. **Mailbox.** Create the Microsoft Graph credential. Set `mailbox` and `own_domain`. Set `test_mode` to `false`. Add a date filter and a page size to the Graph list call.
3. **Orders.** Set `shopify_shop` and a current `shopify_api_version`. Add the Shopify credential to the order search.
4. **Evidence storage.** Set `sp_site_id` and the Graph credential on the upload node.
5. **Slack cards.** Create the Slack header auth credential (`Authorization: Bearer <token>`). Set `slack_channel`. Post one card.
6. **Slack actions.** Expose the webhook, set the Slack interactivity URL, add the signing secret to the Crypto credential, then build out the scaffold.
7. **Watchdog and errors.** Enable the disabled nodes one by one. Set the intake workflow ID on the Execute Workflow nodes. Set the error workflow in intake settings.

## Required before the watchdog is trustworthy

- **Early case row.** Intake writes the case row only after drafting. A run that fails earlier leaves no row, so the watchdog cannot see it. Write a `processing` row at the start of intake.
- **Intake entry point.** Add an Execute Workflow Trigger to intake so retries and backfills can call it with `case_id_override` or `cutoff_override`.
- **Success checkpoint.** `last_intake_success_iso` in Config is static. Store it in a data table and update it at the end of each intake run.
- **Column.** `sharepoint_uploaded` must be written by intake after the upload.
- **Retry ownership.** Only the watchdog retries, capped by `max_retries`. The error handler logs and alerts. Two retry loops without a shared counter create a retry storm.

## Safety rules that do not change

- Nothing is sent to a customer without a button press.
- Junk folder mail is never processed automatically.
- Code decides compensation, not the model.