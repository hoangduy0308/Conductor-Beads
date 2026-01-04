# Example Bead with Spike Learnings

This example shows how to embed spike learnings into a bead for worker context.

```markdown
# Implement Stripe webhook handler

## Context

Spike bd-12 validated: Stripe SDK works with our Node version.
See `.spikes/billing-spike/webhook-test/` for working example.

## Learnings from Spike

- Must use `stripe.webhooks.constructEvent()` for signature verification
- Webhook secret stored in `STRIPE_WEBHOOK_SECRET` env var
- Raw body required (not parsed JSON)

## Acceptance Criteria

- [ ] Webhook endpoint at `/api/webhooks/stripe`
- [ ] Signature verification implemented
- [ ] Events: `checkout.session.completed`, `invoice.paid`
```

## Key Points

1. **Reference spike code**: Link to `.spikes/` directory for working examples
2. **Embed learnings**: Include specific technical decisions from spike
3. **Clear acceptance criteria**: Checkboxes for verifiable outcomes
4. **File scope**: Specify which files this bead will touch (for track assignment)
