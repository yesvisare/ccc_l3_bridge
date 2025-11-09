# M11.2 BRIDGE: Tenant Customization → Resource Management
## Video Script — TVH Framework v2.0 (Production-Ready)

**Track:** CCC Level 3  
**Module:** 11 (Multi-Tenant SaaS Architecture)  
**Type:** Within-Module Bridge  
**Duration:** 8-10 minutes  
**Previous:** M11.2 Augmented (Tenant Customization Implementation)  
**Next:** M11.3 Concept (Resource Management & Throttling)

---

## [00:00-01:30] 1) RECAP: WHAT YOU BUILT

**Duration:** 1.5 minutes

[SLIDE 1: Title - "Bridge: M11.2 → M11.3"]

You just built the configuration layer that makes your multi-tenant SaaS flexible. Let's review.

[SLIDE 2: M11.2 Achievements Checklist]

Here's what you accomplished in M11.2 Tenant Customization:

✓ **Database-driven configuration system** — Add new tenant with custom model/prompts/parameters in <30 seconds via database INSERT, zero code deployments

✓ **Per-tenant model selection with fallback** — GPT-4 for legal tenants, GPT-3.5 for support, automatic fallback to cheaper model if primary unavailable

✓ **Safe prompt template validation** — Prevent prompt injection attacks with allowlisted variables and malicious pattern detection

✓ **Tier-based parameter limits** — Free tier capped at top_k=5, Pro at top_k=10, Enterprise at top_k=20 to prevent cost explosions

[Pause 3 seconds]

Now you can serve 100 tenants with different configurations—all managed through database, no code changes required.

---

## [01:30-03:30] 2) WHY NEXT: THE PROBLEM/TENSION

**Duration:** 2 minutes

[SLIDE 3: The Noisy Neighbor Problem]

Here's what happens with customization but no resource limits: One tenant ruins it for everyone.

**Monday morning production incident:**

[SLIDE 4: Validation-Driven Problem - Abuse Detection]

**9:00 AM:** All tenants performing well. Average response time: 2 seconds.

**3:00 PM:** Tenant A (Free tier) starts automated script
- Rate: 3 queries per second, non-stop
- Their quota: 100 requests/hour (should throttle at 1.67 req/sec)
- **Problem:** No per-tenant rate limiting implemented yet

**5:00 PM System Impact:**
- Response times: 2s → 30s (15x degradation)
- OpenAI bill: $500/month → $4,000/month (8x increase)
- Error rate: 0.1% → 12% (saturation)
- Support tickets: 23 from paying customers

**Each checkpoint failure cost:**

| Checkpoint | If Missing | Impact per Hour | Daily Cost |
|------------|-----------|-----------------|------------|
| Per-tenant rate limit | All tenants share global limit | $200 wasted API calls | $4,800/month |
| Query queue fairness | Heavy user starves others | 5 tenant churns | $2,500 MRR lost |
| Resource monitoring | Can't identify abuser | 4 hours debugging | $400 labor cost |
| Emergency throttle | Can't stop abuse quickly | 12 hours incident duration | Full day of poor UX |

**Total cost of missing resource management: $7,700+ per incident**

[Pause 3 seconds]

[SLIDE 4.5: Reality Check]

**Without per-tenant resource limits:**
- One tenant can monopolize all API tokens
- Paying customers get worse service than free users
- Can't enforce tier-based quotas
- No way to stop abuse without manual intervention

---

## [03:30-05:30] 3) READINESS CHECKLIST

**Duration:** 2 minutes

[SLIDE 5: Checklist - 4 Items]

Before moving to M11.3 Resource Management, verify:

☐ **Configuration caching working**
   → Check: Config lookup latency <10ms P95 (Redis cache hit rate >95%)
   → Impact: Prevents 15-25ms overhead per request from database queries

☐ **Prompt injection prevention tested**
   → Check: Submit malicious prompt with "ignore previous instructions"—should be rejected with clear error
   → Impact: Prevents data leakage attacks worth potential $100K+ in damages

☐ **Cost limits enforced per tier**
   → Check: Free tier tenant attempts top_k=100—should be capped at top_k=5
   → Impact: Prevents $500-2,000/month in unauthorized API usage per tenant

☐ **At least 3 tenants with different configs**
   → Check: Tenant A (GPT-4), Tenant B (GPT-3.5), Tenant C (Claude) all working
   → Impact: Provides test coverage for multi-model routing in production

[SLIDE 6: Warning Visual]

**Warning:** If config caching not working (cache hit rate <90%), fix before M11.3. Resource throttling relies on fast config lookup—slow lookups add 50-100ms per request.

---

## [05:30-07:30] 4) PRACTATHON CHECKPOINT

**Duration:** 2 minutes

[SLIDE 7: PractaThon Framework]

Your checkpoint exercise before M11.3.

**Learn (5 min):**
- Calculate your current "fair share" per tenant:
  - Total OpenAI tokens/month budget: $500
  - Number of tenants: 50
  - Fair share: $10/tenant/month = 5,000 tokens/tenant

**Build (10 min):**
- Simulate noisy neighbor in test environment:
  - Create script sending 100 requests from one tenant
  - Measure: Impact on other tenant's response times
  - Document: At what request rate does system degrade?

**Apply (10 min):**
- Test current system behavior without throttling:
  - Run simulation: Tenant A sends 5 req/sec
  - Measure: How long until Tenant B's queries start failing?
  - Calculate: Cost impact if this ran for 1 hour

**Ship (5 min):**
- Create "noisy neighbor test report":
  - Baseline performance: 100ms response time
  - With heavy load: XXXms response time
  - Breaking point: Y requests/second
  - Estimated monthly cost without limits: $Z

[SLIDE 8: Expected Output Example]

Your report should show:

**Noisy Neighbor Impact Analysis**
- Baseline (normal load): 120ms P95, $500/month cost
- Heavy load (1 tenant at 5 req/sec): 2,100ms P95, $4,200/month cost
- Breaking point: 3 req/sec sustained → system saturation
- **Conclusion:** Need per-tenant limit of 1.67 req/sec (100/hour)

This quantifies the need for M11.3 resource management.

---

## [07:30-08:30] 5) CALL-FORWARD: NEXT FOCUS

**Duration:** 1 minute

[SLIDE 9: M11.3 Preview - Resource Management & Throttling]

Tomorrow, in M11.3 Concept + Augmented, you'll add resource limits:

**1. Per-tenant rate limiting (100 queries/hour)**
   → Token bucket algorithm preventing any single tenant from monopolizing resources

**2. Fair query queue with priority tiers**
   → Enterprise customers processed first, but Free tier still gets service (no starvation)

**3. Emergency quota overrides**
   → Support team can grant temporary 10x quota increase in <60 seconds without code deployment

[SLIDE 10: The Question for M11.3]

**"How do you prevent one tenant from ruining the experience for everyone else without building a full billing system?"**

[Pause 3 seconds]

You'll learn when quotas are premature (<50 tenants don't need this) and what to use instead.

See you then.

[END - Total: 8 min 30 sec]

---

## PRODUCTION NOTES

**Recording Setup:**
- Fast-paced bridge format
- Strong emphasis on quantified incident cost ($7,700+ per abuse incident)
- Tonal shift: Celebration → Urgency → Helpful → Energized
- Source: M11.2 Augmented and M11.3 Concept

**Slide Deck:**
- 10 slides total
- Key visuals: Achievements checklist, incident timeline, cost breakdown table, noisy neighbor impact

**Visual Assets:**
- achievements_checklist.png, incident_timeline.png, cost_breakdown_table.png, noisy_neighbor_test_report.png

**References:**
- Previous: M11.2 Augmented
- Next: M11.3 Concept
- Module: Module 11

---

## NO QUIZ (BRIDGE FORMAT)

Bridge videos do not include quiz questions. Learners proceed directly to M11.3 Concept after watching.
