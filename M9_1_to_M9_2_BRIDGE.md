# Module 9: Advanced Retrieval Techniques
## BRIDGE: M9.1 Query Decomposition → M9.2 Multi-Hop Retrieval
**Duration:** 8-10 minutes
**Format:** Bridge (Continuity between videos)
**Placement:** After M9.1 Augmented, before M9.2 Concept

---

## [00:00-01:30] 1) RECAP: WHAT YOU BUILT

[SLIDE 1: Title - "Bridge: Query Decomposition → Multi-Hop Retrieval"]

You just crossed a major milestone in Level 3. Let's review what you built.

[SLIDE 2: M9.1 Achievements Checklist]

Here's what you accomplished in M9.1 Query Decomposition & Planning:

[Point to each item]

✓ **Query decomposer with 95%+ accuracy** — Built LLM-powered system using GPT-4 that breaks complex queries into atomic sub-queries with dependency relationships

✓ **Dependency graph execution** — Implemented NetworkX-based DAG that determines optimal retrieval order, executing 3+ sub-queries in parallel with 60% latency reduction

✓ **Production-ready executor** — Created rate-limited async executor with max_concurrent=3, exponential backoff retry, and timeout handling to prevent API rate limit failures

✓ **Answer synthesis with conflict resolution** — Built synthesis system that combines sub-answers, detects contradictions using temporal metadata, and outputs confidence scores (threshold: 0.8+)

[Pause 3 seconds]

This is a huge leap. You went from single-shot retrieval (accuracy 2.1/5 for complex queries) to intelligent query planning (accuracy 4.0/5). Your system now handles 15-20% of production traffic—the complex multi-part questions that single-shot RAG couldn't touch.

---

## [01:30-03:30] 2) WHY NEXT: THE PROBLEM

[SLIDE 3: The Gap - Reference Chains]

But here's the problem you're about to hit. Your decomposition system works brilliantly for queries you can break down upfront. Questions like "Compare Q1 vs Q2 revenue" decompose into clear sub-queries you can plan in advance.

But what happens with this query:

**"What were the audit findings from last year's compliance review, and what corrective actions were implemented?"**

Your decomposition breaks it into:
1. Sub-query 1: "What were the audit findings?"
2. Sub-query 2: "What corrective actions were implemented?"

[SLIDE 4: The Retrieval Problem]

Here's where it fails:

**Sub-query 1** retrieves the audit report. Great. It mentions: *"Three critical findings: data breach in Q2, inadequate access controls, and missing encryption on legacy systems. Refer to documents CA-2024-001, CA-2024-002, and CA-2024-003 for corrective action details."*

**Sub-query 2** searches for "corrective actions" — but it doesn't find CA-2024-001, CA-2024-002, or CA-2024-003. Why? Because those document IDs weren't in the original query. They only appeared in the first retrieval's results.

Your system can't follow the reference trail. The answer is **incomplete**.

[SLIDE 5: The Scale of This Problem]

[Read metrics with emphasis]

**In production systems with reference-heavy documents:**
- 15-25% of queries require following references across multiple documents
- Single-pass retrieval: 40% answer quality (misses referenced docs)
- Users manually follow references: 8-12 minutes per query
- Support ticket escalations: +45% (users can't find complete information)

**Cost of not solving this:**
- 1,000 reference-chain queries/day × 10 min manual work = 167 hours/day wasted
- At ₹500/hour support cost = ₹83,500/day = ₹25L/month in manual effort

**What you need:** Multi-hop retrieval that automatically follows document references.

[SLIDE 6: Reality Check]

**Reality Check:** Not all references need following.

If your documents are well-written and self-contained, you don't need multi-hop retrieval. This is specifically for:
- Enterprise knowledge bases with cross-referenced documents
- Legal/medical systems with citation chains
- Compliance systems with linked procedures
- Technical documentation with dependency trees

Check your retrieval logs: if <5% of queries mention "refer to document X" or have incomplete answers, stick with decomposition alone.

---

## [03:30-05:30] 3) READINESS CHECKLIST

[SLIDE 7: Readiness Checklist - 4 Items]

Before moving to M9.2 Multi-Hop & Recursive Retrieval, verify these four things:

[Read each item with clear pauses]

☐ **Query decomposer achieving 95%+ accuracy on test queries**
   → Check: Run 20 complex queries, verify JSON output format and dependency structure
   → Impact: Saves 4+ hours debugging malformed decompositions in M9.2

☐ **Dependency graph execution working with 3+ parallel sub-queries**
   → Check: Test case with 5 sub-queries (3 parallel, 2 sequential) completes in <2 seconds
   → Impact: Multi-hop builds on this—if parallel execution fails, multi-hop will be 3x slower

☐ **Rate limiting implemented to prevent API throttling**
   → Check: Fire 10 parallel queries, verify max_concurrent=3 batching, no 429 errors
   → Impact: Multi-hop adds 2-5 hops per query—without rate limiting, you'll hit limits immediately

☐ **Answer synthesis producing confidence scores ≥0.8 for 80% of test queries**
   → Check: Review synthesis_confidence metric in logs, flag queries with <0.8 score
   → Impact: Multi-hop synthesis is more complex (aggregating 5+ retrievals)—low confidence now means failures later

[SLIDE 8: Warning]

**Warning:** If your decomposer returns malformed JSON (missing "depends_on" field) more than 5% of the time, M9.2 will break. Fix prompt engineering first:
- Add explicit JSON schema examples
- Use GPT-4 (not GPT-3.5) for decomposition
- Test with edge cases (1-word queries, ambiguous questions)

Missing this checkpoint costs 2-3 days debugging cryptic NetworkX graph errors in M9.2.

---

## [05:30-07:30] 4) PRACTATHON CHECKPOINT

[SLIDE 9: PractaThon Checkpoint - 30 minutes]

Your checkpoint exercise before M9.2: **Dynamic Threshold Adjustment**

This bridges decomposition (M9.1) and multi-hop retrieval (M9.2) by teaching your system when to trigger multi-hop based on retrieval quality.

**Learn (5 min):**
- Analyze your current decomposition results for 10 queries
- Calculate retrieval completeness: How many queries have "refer to" or "see document" in sub-answers?
- Identify the threshold: What similarity score indicates incomplete retrieval?

**Build (10 min):**
- Implement a completeness detector that scans sub-answers for reference patterns:
  - Regex: `refer to|see document|as detailed in|reference #|doc ID`
  - If match found → trigger multi-hop flag
- Add similarity score analysis:
  - If top result score <0.75 → potentially incomplete retrieval
  - If score gap (1st to 2nd result) <0.1 → ambiguous retrieval

**Apply (10 min):**
- Test on 10 queries with known reference chains (from your audit/compliance docs)
- Measure: How many are correctly flagged for multi-hop?
- Benchmark: False positives (flagged but didn't need multi-hop) vs false negatives (missed)

**Ship (5 min):**
- Document trigger conditions in a decision table:
  ```
  Trigger Multi-Hop If:
  - Reference pattern detected in sub-answer (90% precision)
  - Top similarity score <0.75 (70% precision)
  - Score gap <0.1 AND contains "refer to" (95% precision)
  ```
- Commit to GitHub: `completeness_detector.py` with test results

[SLIDE 10: Expected Output]

Your completeness detector should look like this:

```python
def should_trigger_multihop(sub_answer, similarity_score, score_gap):
    """Decide if this sub-answer needs multi-hop retrieval."""
    
    reference_patterns = [
        "refer to", "see document", "as detailed in",
        "reference #", "doc ID", "document [A-Z]{2}-[0-9]+"
    ]
    
    has_reference = any(
        re.search(pattern, sub_answer, re.IGNORECASE) 
        for pattern in reference_patterns
    )
    
    low_confidence = similarity_score < 0.75
    ambiguous = score_gap < 0.1
    
    # Decision logic
    if has_reference and (low_confidence or ambiguous):
        return True, "Reference detected + low confidence"
    elif has_reference:
        return True, "Reference detected"
    else:
        return False, "Complete answer, no multi-hop needed"
```

**Success metric:** 85%+ precision (correctly identifies queries needing multi-hop).

---

## [07:30-08:30] 5) CALL-FORWARD: NEXT FOCUS

[SLIDE 11: M9.2 Multi-Hop & Recursive Retrieval Preview]

Tomorrow, in M9.2 Multi-Hop & Recursive Retrieval, you'll add three critical capabilities:

**1. Automatic Reference Extraction**
   → Parse sub-answers for document IDs, URLs, section references
   → Build a knowledge graph of document relationships
   → Extract "follow-up queries" from retrieved content

**2. Recursive Retrieval with Stop Conditions**
   → Implement 2-5 hop depth limit (prevent infinite loops)
   → Add relevance pruning (don't follow every reference)
   → Track visited documents (avoid re-retrieving same content)

**3. Knowledge Graph Traversal Optimization**
   → Use graph algorithms (BFS vs DFS) to explore reference chains
   → Prune irrelevant paths early (save API calls)
   → Aggregate answers from multiple hops into coherent narrative

[SLIDE 12: The Question for M9.2]

**"Your decomposition handles multi-part questions. But what happens when the answer is hidden across documents that only reference each other?"**

[Pause 3 seconds]

Multi-hop retrieval follows the reference trail automatically—extracting document IDs from initial results, retrieving referenced documents, and recursively exploring until you have the complete answer.

It's the difference between:
- **Single-pass:** "The audit found 3 issues. See CA-2024-001 for details." ❌ Incomplete
- **Multi-hop:** "The audit found 3 issues: data breach (fixed via encryption upgrade in CA-2024-001), inadequate access controls (fixed via RBAC implementation in CA-2024-002), missing encryption (fixed via TLS 1.3 rollout in CA-2024-003)." ✓ Complete

See you then.

[END - Total: 8 min 30 sec]

---

## PRODUCTION NOTES

**Recording Setup:**
- Bridge format: Fast-paced connector video
- Pause cues = 3-5 sec for warnings/reflection
- Tonal shifts: Celebration (RECAP) → Urgency (WHY NEXT) → Helpful (CHECKLIST) → Energized (CALL-FORWARD)
- Source: Extracted from M9.1 Augmented and M9.2 Concept

**Slide Deck Requirements:**
- Total slides: 12
- Naming: M9-1-to-M9-2-Bridge-01.png through M9-1-to-M9-2-Bridge-12.png
- Key visuals: 
  - Achievements checklist (slide 2)
  - Reference chain problem diagram (slide 4)
  - Cost calculation (slide 5)
  - Readiness checklist (slide 7)
  - Completeness detector code snippet (slide 10)

**Visual Assets:**
- achievements_checklist.png (4 items with ✓)
- reference_chain_problem.png (showing missing CA-2024-XXX docs)
- cost_quantification.png (167 hours/day calculation)
- readiness_checklist.png (4 checkpoints with ☐)
- completeness_detector_example.png (code sample)
- multihop_preview.png (3 capabilities diagram)

**References:**
- Previous: M9.1 Augmented (Query Decomposition & Planning)
- Next: M9.2 Concept (Multi-Hop & Recursive Retrieval)
- Module: Module 9 - Advanced Retrieval Techniques

**Editing Notes:**
- Emphasize "167 hours/day wasted" visually with cost overlay
- Use animation for reference chain problem (show missing links)
- Highlight completeness detector code with syntax highlighting
- **Quantification pattern:** Time-driven (manual effort cost)

---

## NO QUIZ (BRIDGE FORMAT)

Bridge videos do not include quiz questions. Learners proceed directly to M9.2 Concept (Multi-Hop & Recursive Retrieval) after watching.
