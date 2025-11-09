# M11.1 BRIDGE: Tenant Isolation → Tenant Customization
## Video Script — TVH Framework v2.0 (Production-Ready)

**Track:** CCC Level 3  
**Module:** 11 (Multi-Tenant SaaS Architecture)  
**Type:** Within-Module Bridge  
**Duration:** 8-10 minutes  
**Previous:** M11.1 Augmented (Tenant Isolation Implementation)  
**Next:** M11.2 Concept (Tenant-Specific Customization)

---

## [00:00-01:30] 1) RECAP: WHAT YOU BUILT

**Duration:** 1.5 minutes

[SLIDE 1: Title - "Bridge: M11.1 → M11.2"]

You just crossed a major milestone in multi-tenant architecture. Let's review what you built.

[SLIDE 2: M11.1 Achievements Checklist]

Here's what you accomplished in M11.1 Tenant Isolation:

[Point to each item]

✓ **Multi-tenant isolation system** — PostgreSQL Row-Level Security preventing cross-tenant data leakage even with application bugs

✓ **Namespace-based data separation** — Each tenant's vectors isolated in dedicated Pinecone namespaces, supporting 100 tenants per index

✓ **Cost allocation engine** — Per-tenant cost tracking with monthly reconciliation (allocated costs within 5% of actual bills)

✓ **Auto-scaling logic** — Automatic provisioning of new Pinecone indexes when namespace capacity reaches 80% (90/100 namespaces)

[Pause 3 seconds]

This is production-grade isolation. You can onboard 100 customers with confidence that their data stays separate and you can track costs per tenant for accurate pricing.

---

## [01:30-03:30] 2) WHY NEXT: THE PROBLEM/TENSION

**Duration:** 2 minutes

[SLIDE 3: The Configuration Mismatch Problem]

Here's the problem you're about to hit: Every single tenant runs identical configuration.

**Real-world scenario from production:**

[SLIDE 4: Cost-Driven Problem - Customer Differentiation]

**Company A (Law Firm):**
- Needs: GPT-4, high accuracy, comprehensive retrieval
- Budget: $5,000/month
- Reality: Getting same GPT-3.5 as everyone else → **Frustrated**

**Company B (Customer Support):**
- Needs: Fast responses, basic accuracy sufficient
- Budget: $500/month  
- Reality: Paying for GPT-4 processing they don't need → **Hemorrhaging money**

[Pause 3 seconds]

**The burn rate:**

Company B configuration:
- Current: GPT-4 + top_k=10 + reranking = $0.045 per query
- Needed: GPT-3.5 + top_k=5 + no reranking = $0.008 per query
- Waste: $0.037 per query × 5,000 queries/month = **$185/month lost on ONE tenant**

Across 100 tenants with similar mismatches: **$18,500/month** in wasted API costs.

[SLIDE 4.5: Reality Check]

**Without per-tenant customization:**
- Can't offer tiered pricing (everyone gets same service)
- Overpay on API costs for light users
- Lose enterprise customers who need premium features
- Manual code changes for every custom request

---

## [03:30-05:30] 3) READINESS CHECKLIST

**Duration:** 2 minutes

[SLIDE 5: Checklist - 4 Items]

Before moving to M11.2 Tenant Customization, verify these four things:

[Read each item with clear pauses]

☐ **Row-Level Security policies verified**
   → Check: Query from Tenant A with wrong tenant_id returns zero results (not error, but empty set)
   → Impact: Saves 8 hours debugging data leakage in production

☐ **Cost allocation reconciliation passing**
   → Check: Run monthly report—allocated costs match actual bills within ±5%
   → Impact: Prevents $2,000-5,000/month pricing errors from untracked costs

☐ **Namespace capacity monitoring configured**
   → Check: Alert fires at 72/90 namespaces used (80% threshold)
   → Impact: Prevents hitting namespace limit and blocking new tenant signups

☐ **At least 3 test tenants provisioned**
   → Check: Can create tenant, upload documents, query successfully for 3 separate tenant IDs
   → Impact: Provides baseline for testing per-tenant configurations in M11.2

[SLIDE 6: Warning Visual]

**Warning:** If cost allocation reconciliation fails (allocated ≠ actual by >10%), stop here. Fix your cost tracking before adding per-tenant customization—otherwise you can't price tiers accurately.

---

## [05:30-07:30] 4) PRACTATHON CHECKPOINT

**Duration:** 2 minutes

[SLIDE 7: PractaThon Framework]

Your checkpoint exercise before M11.2 Tenant Customization.

**Learn (5 min):**
- Analyze your current tenant_configs table schema
- Review which configuration parameters each tenant tier will need (model, top_k, temperature)

**Build (10 min):**
- Add 2 more test tenants with different data characteristics:
  - Tenant D: 1,000 small documents (100-200 words each)
  - Tenant E: 50 large documents (5,000+ words each)

**Apply (10 min):**
- Query both tenants with same question
- Measure: Response time, tokens used, cost per query
- Document: Which tenant would benefit from different model/parameters?

**Ship (5 min):**
- Create comparison spreadsheet: Tenant D vs Tenant E (latency, cost, optimal config)
- This becomes your test data for M11.2 customization

[SLIDE 8: Expected Output Example]

Your comparison should show:

| Tenant | Doc Count | Avg Doc Size | Current Cost/Query | Optimal Model | Optimal top_k | Projected Cost |
|--------|-----------|--------------|-------------------|---------------|---------------|----------------|
| D | 1,000 | 150 words | $0.012 | gpt-3.5 | 5 | $0.006 |
| E | 50 | 5,200 words | $0.035 | gpt-4 | 10 | $0.045 |

This data drives M11.2 configuration decisions.

---

## [07:30-08:30] 5) CALL-FORWARD: NEXT FOCUS

**Duration:** 1 minute

[SLIDE 9: M11.2 Preview - Tenant-Specific Customization]

Tomorrow, in M11.2 Concept + Augmented, you'll add per-tenant configuration:

**1. Per-tenant model selection (GPT-4, GPT-3.5, Claude)**
   → Different tenants can use different LLMs based on their tier and needs

**2. Database-driven configuration system**
   → Add/modify tenant configs in seconds via database UPDATE, zero code deployments

**3. Safe prompt template injection**
   → Let tenants customize prompts while preventing prompt injection attacks

[SLIDE 10: The Question for M11.2]

**"How do you let each tenant customize their experience without turning your codebase into an unmaintainable mess?"**

[Pause 3 seconds]

You'll learn when per-tenant customization adds value versus when standardization is better for your product.

See you then.

[END - Total: 8 min 30 sec]

---

## PRODUCTION NOTES

**Recording Setup:**
- Bridge format: Fast-paced connector video
- Pause cues = 3 sec for warnings/reflection
- Tonal shift: Celebration (recap) → Urgency (problem) → Helpful (checklist) → Energized (call-forward)
- Source: Extracted from M11.1 Augmented and M11.2 Concept

**Slide Deck Requirements:**
- Total slides: 10 slides
- Naming: M11-1-Bridge-01.png through M11-1-Bridge-10.png
- Key visuals:
  - achievements_checklist.png (4 M11.1 accomplishments with checkmarks)
  - cost_mismatch_scenario.png (Company A vs B cost breakdown)
  - readiness_checklist.png (4 verification items)
  - tenant_comparison_table.png (Tenant D vs E analysis template)

**Visual Assets:**
- achievements_checklist.png (M11.1 accomplishments)
- cost_burn_calculation.png (API cost waste across tenants)
- readiness_checklist.png (4 checkpoints)
- next_preview.png (M11.2 capabilities diagram)

**References:**
- Previous: M11.1 Augmented - Tenant Isolation Implementation
- Next: M11.2 Concept - Tenant-Specific Customization
- Module: Module 11 - Multi-Tenant SaaS Architecture

**Editing Notes:**
- Celebration tone for RECAP
- Urgency tone for WHY NEXT (emphasize cost waste)
- Helpful tone for CHECKLIST
- Energized tone for CALL-FORWARD

---

## NO QUIZ (BRIDGE FORMAT)

Bridge videos do not include quiz questions. Learners proceed directly to M11.2 Concept after watching.
