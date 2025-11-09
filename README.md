# BRIDGE: M12.2 Billing → M12.3 Self-Service Onboarding

**Validation & Readiness Notebook**

## Purpose

This bridge validation notebook ensures your system is ready to transition from manual billing (M12.2) to automated self-service onboarding (M12.3). It validates critical infrastructure components without building new features.

## What This Validates

The notebook performs 4 essential readiness checks based on the M12.2 → M12.3 bridge requirements:

1. **Stripe Subscriptions** - Verify subscriptions can be created with payment methods attached
2. **Tenant Database Schema** - Confirm database includes status tracking fields for provisioning
3. **Pinecone Namespaces** - Validate programmatic namespace creation for multi-tenant isolation
4. **Email Service** - Check automated welcome email capability

## How to Run

### Prerequisites

Set up environment variables for the services you want to validate:

```bash
# Stripe (Required for Check #1)
export STRIPE_SECRET_KEY=sk_test_...
export STRIPE_WEBHOOK_SECRET=whsec_...

# Database (Required for Check #2)
export DATABASE_URL=postgresql://...

# Pinecone (Required for Check #3)
export PINECONE_API_KEY=...
export PINECONE_ENVIRONMENT=...

# Email Service (Required for Check #4 - one of these)
export SMTP_HOST=smtp.example.com
# OR
export SENDGRID_API_KEY=SG...
# OR
export AWS_SES_REGION=us-east-1
# OR
export MAILGUN_API_KEY=...
```

### Install Dependencies

```bash
pip install stripe pinecone-client
# Add email library as needed (sendgrid, boto3, etc.)
```

### Run the Notebook

```bash
jupyter notebook Bridge_L3_M12_2_to_M12_3_Readiness.ipynb
```

Or use JupyterLab:

```bash
jupyter lab Bridge_L3_M12_2_to_M12_3_Readiness.ipynb
```

Execute all cells sequentially (Shift + Enter or Run All).

## Pass Criteria

**PASS** if:
- All configured services show `✓` status
- No `✗ Error` messages appear
- `⚠️ Skipping` messages are acceptable for unconfigured optional services

**ATTENTION NEEDED** if:
- Any `✗ Error` messages appear
- Stripe or Database checks fail (these are critical for M12.3)

**OPTIONAL** checks:
- Pinecone and Email can be configured later, but will block self-service onboarding in M12.3

## What Happens If Checks Fail?

| Check | Failure Impact | Fix Required |
|-------|----------------|--------------|
| Stripe | Can't auto-create subscriptions → billing breaks | Configure Stripe API keys, verify webhook handlers |
| Database | Can't track provisioning state → incomplete tenant setup | Add status, config, metadata fields to tenants table |
| Pinecone | Can't isolate tenant data → security breach risk | Configure Pinecone credentials, test namespace API |
| Email | Can't send credentials → customers can't login → 30% drop-off | Configure email service (SMTP/SendGrid/SES/Mailgun) |

## Next Module

**M12.3 Self-Service Tenant Onboarding** builds:
- 60-second automated signup flow
- Interactive setup wizard
- Sample data auto-loading
- Activation metric tracking

**Time saved:** 10 hours/month at 50 customers (2.5 workweeks/year)

**The driving question:** "How do you onboard 10 customers while you sleep—completely self-service, from signup to first successful query in under 5 minutes?"

## Bridge Reference

Source: [M12_2_to_M12_3_BRIDGE.md](./M12_2_to_M12_3_BRIDGE.md)

Duration: 8 minutes 30 seconds
Track: CCC Level 3
Type: Within-Module Transition
