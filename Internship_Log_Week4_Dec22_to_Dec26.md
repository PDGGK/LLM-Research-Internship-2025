# 🚀 Internship Log | Week 4

> **Duration**: December 22, 2025 (Mon) - December 26, 2025 (Fri)  
> **Research Focus**: Multi-table Join Reasoning for Text-to-SQL in Large-Scale Schemas  
> **Role**: AI Algorithm Intern

---

## 📅 Week 4 Overview

### Dec 22 (Mon) | Column-Level Schema Enhancement Success 🎉

**Core Achievement**

**Column-level Schema Enhancement Experiment Succeeded**: Using Qwen 32B to optimize descriptions for 163 core columns, recall rate soared from 39.7% to **99.2%**.

---

## 1. Problem Diagnosis

### 1.1 Root Cause Discovery
Previous column-level retrieval performed poorly. Analysis revealed:
- Column-level Schema only had raw database comments (e.g., `EVENT_TYPE_NAME: Event type name`)
- **Table-level had LLM-optimized descriptions, column-level did not**
- Column descriptions too brief, lacking synonyms and business context

### 1.2 Misconception Clarified
Previously thought connecting to database to query real values was needed for column-level matching. Actually:
- Column-level retrieval relies on **column name + column description semantic matching**
- `sample_rows` is just supplementary, not core
- Key is making column descriptions contain rich business context and synonyms

---

## 2. Solution: LLM-Enhanced Column Descriptions

### 2.1 Implementation
Used Qwen2.5-Coder-32B-Instruct to generate enhanced descriptions for core table columns:

```python
# Generation prompt
prompt = f"""
Table: {table_name}
Column: {column_name}
Original Description: {original_desc}
Sample Values: {sample_rows}

Please generate a rich Chinese description (including synonyms, business meaning)
"""
```

### 2.2 Enhancement Examples

| Column | Original Description | Enhanced Description |
|--------|---------------------|---------------------|
| CUSTOMER_NAME | Parent ID (Wrong!) | Customer name, used to distinguish different customer entities. Synonyms: client name, user name... |
| EVENT_TYPE_NAME | Event type name | Alert/event type, e.g., Ping event, Trap event. Used for classification statistics... |

### 2.3 Output
- Processed **163 core columns** (6 core tables)
- Storage location: `spider2_dev/schemas_column_enhanced/netcaredb_ai/`

---

## 3. Test Results

### 3.1 100-Question Recall Test (Medium 50 + Hard 50)

| Method | Average Recall | Full Recall |
|--------|---------------|-------------|
| Table-level Only | 39.7% | 0/100 |
| **Table+Enhanced Column Fusion** | **99.2%** | **96/100** |

### 3.2 By Difficulty

| Difficulty | Table-level | Table+Enhanced Column Fusion |
|------------|-------------|------------------------------|
| Medium | 41% (0/50) | **100%** (50/50) |
| Hard | 39% (0/50) | **98%** (46/50) |

### 3.3 Key Findings
- **59.5 percentage point improvement!**
- Enhanced column-level retrieval became the main contributor
- Even hard questions achieved 98% full recall

---

## 4. Technical Details

### 4.1 Current Retrieval Flow

```
Question → Table-level Sparse Retrieval (TOP 10)
        → Enhanced Column-level Sparse Retrieval (TOP 200 → Aggregate to Table TOP 10)
        → Fusion (Union)
        → Return retrieved tables
```

### 4.2 Performance Bottleneck
Current testing of 100 questions takes ~15 minutes because:
- Each question requires recomputing schema embeddings
- Using `compute_score` instead of precomputed vectors

### 4.3 Embedding Precomputation Acceleration ✅

Implemented embedding precomputation:

| Metric | Before | After Precomputation |
|--------|--------|---------------------|
| 100 questions time | ~15 min | **4.3 sec** |
| Per question | ~9 sec | **43 ms** |
| Speedup | - | **207x** |

Implementation: `retrieval/precompute_embeddings.py`

Precomputed file: `spider2_dev/schema_embeddings.pkl` (0.9 MB)

---

## 5. Deep Dive: Why Does Embedding Precomputation Achieve 207x Speedup?

### 5.1 Understanding BGE-M3 Sparse Retrieval

BGE-M3's Sparse retrieval (lexical weight matching) fundamentally is:

```
Text → Tokenize → Transformer Encoding → Generate weight for each token → Sparse Vector
```

**Core operation**: Convert text into a **sparse vector** where each dimension corresponds to a token in the vocabulary, with value being that token's importance weight.

**Example**:
```
"Device alert statistics" → {device: 0.35, alert: 0.42, statistics: 0.28, ...}
```

### 5.2 Previous Approach: Why Slow?

Previously using `compute_score(sentence_pairs)`:

```
Each question:
  For each Schema (507):
    1. Tokenize question text          → ~1ms
    2. Tokenize Schema text            → ~1ms  
    3. Transformer encode question     → ~50ms ← Bottleneck!
    4. Transformer encode Schema       → ~50ms ← Bottleneck!
    5. Compute lexical matching score  → ~0.1ms
                                      ≈ 100ms × 507 = ~50sec
```

**Problem**:
- Each question re-encodes all 507 Schemas
- **Transformer encoding is O(n²) complexity**, very time-consuming
- 507 Schemas × 100ms/each = **~50 sec/question**

### 5.3 Precomputation Approach: Why Fast?

Core insight: **Schemas are fixed, no need to re-encode every time!**

```
【Precomputation Phase (one-time)】
507 Schemas → Transformer encoding → Save 507 sparse vectors → 5sec ← Only once!

【Retrieval Phase (per question)】
Question → Transformer encoding (1 time) → Question sparse vector → ~30ms
Question vector × 507 Schema vectors → Lexical matching scores → ~10ms ← Pure math!
                                                           Total ~40ms
```

**Why fast?**
1. **Schemas encoded only once**: 507 Schemas encoded once and saved, no repetition
2. **Only encode question at query time**: Each question goes through Transformer once (~30ms)
3. **Similarity computation is pure math**: Sparse vector dot product, CPU millisecond-level

### 5.4 Mathematical Perspective: Computation Comparison

| Operation | Before (per question) | After Precomputation (per question) |
|-----------|----------------------|-------------------------------------|
| Transformer Encoding | **508 times** (1 question + 507 Schemas) | **1 time** (only question) |
| Lexical Matching | 507 times | 507 times |

**Bottleneck Analysis**:
- Transformer encoding: ~50ms/time, GPU-intensive
- Lexical matching: ~0.02ms/time, pure CPU

```
Before: 508 × 50ms = 25,400ms ≈ 25sec
After:  1 × 50ms + 507 × 0.02ms = 50ms + 10ms = 60ms

Speedup ratio: 25,400 / 60 ≈ 423x (theoretical)
Actual: 207x (including I/O overhead)
```

### 5.5 Analogy

**Recipe Lookup Example**:

Imagine you're a chef answering 100 "how to cook this" questions, needing to check 500 cookbooks each time.

**Before (no precomputation)**:
```
Question 1 → Open 500 cookbooks → Read each → Find answer → Close all books
Question 2 → Open 500 cookbooks → Read each → Find answer → Close all books
...
100 questions = Open/close cookbooks 50,000 times
```

**With precomputation**:
```
[Preparation] Copy all 500 cookbook indexes and keywords onto a single card
Question 1 → Check card → Quick match → Find answer
Question 2 → Check card → Quick match → Find answer
...
100 questions = Check card 100 times
```

**Card = Precomputed embedding vectors**
**Opening cookbook = Transformer encoding**

### 5.6 Code Implementation Comparison

**Previous slow code**:
```python
def retrieve(question, schemas):
    pairs = [[question, s["text"]] for s in schemas]  # 507 pairs
    scores = model.compute_score(pairs)  # Each pair encoded!
    return sorted(scores)
```

**Fast precomputed code**:
```python
# Precomputation phase (one-time)
schema_embeddings = model.encode(schema_texts, return_sparse=True)
save(schema_embeddings)

# Retrieval phase (per question)
def retrieve(question):
    q_vec = model.encode([question], return_sparse=True)  # Only encode question
    scores = [dot_product(q_vec, s_vec) for s_vec in schema_embeddings]
    return sorted(scores)
```

### 5.7 Summary

| Principle | Explanation |
|-----------|-------------|
| **Why precomputation works** | Schemas are fixed, embeddings don't change |
| **Why it's fast** | Avoids repeated Transformer encoding (most time-consuming) |
| **How much speedup** | From 508 encodings → 1 encoding, theoretical 500+ times |
| **Actual speedup** | 207x (including I/O, Python overhead) |
| **Applicable scenarios** | Any "fixed documents + dynamic queries" retrieval scenario |

**Core Insight**: **Trade compute for storage**. Use 0.9MB of storage space for 207x speed improvement.

---

## 6. Next Steps

- [ ] Expand column enhancement to more tables (currently only 6 core tables)
- [ ] Implement embedding precomputation acceleration
- [ ] Conduct complete SQL generation tests
- [ ] Prepare final report

---

## 7. Daily Summary

Today's work validated an important hypothesis: **Column description quality directly determines column-level retrieval effectiveness**. By LLM-enhancing column descriptions, we can significantly improve recall, fundamentally solving the previous "column-level retrieval doesn't work" problem.

This also demonstrates:
1. Vector retrieval effectiveness largely depends on embedding text quality
2. Simple database comments aren't enough; rich business context is needed
3. LLM is a good tool for generating such descriptions

---

*Report generated: 2024-12-22 15:22*

---

### Dec 23 (Tue) | Full-Scale Enhancement Comparison & Research Direction Reflection ⚠️

**Core Achievement**

**Column-level Enhancement Strategy Validated**: Comparative experiments proved that core table enhancement (163 columns) far outperforms full-scale enhancement (3353 columns), with full-scale enhancement providing zero unique advantages.

---

#### 1. Full-Scale Column Enhancement Experiment

**Experimental Design**
- Used Qwen 32B to generate enhanced descriptions for all 3353 columns across 344 tables
- Duration: 86.3 minutes (3190 successful, 0 failed)
- Re-precomputed embeddings (3697 Schemas, 5.4 MB)

**Results**

| Approach | Full Recall Rate | Average Recall Rate |
|----------|-----------------|---------------------|
| Core Column Enhancement (163 cols) | **97.4%** | **99.5%** |
| Full-Scale Enhancement (3353 cols) | 50.3% | 76.6% |

**Recall rate actually dropped by 47 percentage points!**

---

#### 2. Root Cause Analysis

**By Difficulty Comparison**

| Difficulty | Core Column Enhanced | Full-Scale Enhanced | Gap |
|------------|---------------------|---------------------|-----|
| Easy | 100% | 76% | -24% |
| Medium | 100% | 43.6% | -56.4% |
| Hard | 92% | 31.7% | -60.3% |

**Table-level vs Column-level Impact Analysis**

| Retrieval Method | Core Column Enhanced | Full-Scale Enhanced |
|-----------------|----------------------|---------------------|
| Table-level Only | ~40% | 52.3% ✅ Slightly up |
| Column-level Only | ~95% | 48.7% ❌ Significant drop |
| Table+Column Fusion | 97.4% | 50.3% |

**Finding**: Problem lies in column-level retrieval—95% of columns become noise in full-scale enhancement.

**Failed Question Cross-Comparison**

| Comparison Type | Count |
|-----------------|-------|
| Core column success + Full-scale fail | **142** |
| Full-scale success + Core column fail | **0** |

**Conclusion**: Full-scale enhancement has **zero unique advantages**.

---

#### 3. Technical Documentation

**Embedding Precomputation Principles Deep Dive**

Completed detailed technical documentation (~1100 lines), including:
- Sparse vs Dense vectors
- Transformer working principles
- Attention mechanism explained
- Q/K/V vector computation process
- Encoder vs Decoder differences
- Precomputation acceleration principles (207x speedup)

---

#### 4. Output Files

| File | Description |
|------|-------------|
| `column_enhancement_comparison_report.md` | Core column vs Full-scale comparison report |
| `embedding_precomputation_deep_dive.md` | Technical principles deep dive |
| `recall_test_300_full_report.md` | 302-question recall test report |

---

#### 5. 💭 Deep Reflection: Fundamental Error in Research Direction

> This reflection was written on Dec 24 about Dec 23's work, after discussing with my mentor and realizing the severity of the problem.

**We made a fundamental error: wrong control variables.**

**Incorrect Research Setup**

Initially, by observing the data, I formed a hypothesis: some tables in our database are more important (like device types, usage records, fault records). I naively assumed that if I could enhance retrieval for these core tables, the overall algorithm accuracy would improve.

Then I had DeepSeek generate test questions based on all 344 tables to test "whether we can retrieve these core tables each time."

**This was completely wrong!**

The correct approach should be:
- **Have DeepSeek generate questions targeting the 3 core tables**
- Then test whether we can retrieve these 3 tables from the 344 tables

Not:
- Have DeepSeek look at 344 tables and generate random questions
- Hope the retrieval results happen to be core tables

**This is classic "programming toward results"**

We were misled by the test set's peculiarities:
1. Case questions and DeepSeek-generated questions happened to involve core tables a lot
2. So when I optimized core table retrieval, recall rate did go up
3. But this was just **coincidence**, not actually solving the problem

The real situation was completely different.

**Wrong from the start, wrong all the way through**

If we're researching a topic and haven't even defined the problem correctly, how can we claim research success?

This experience made me realize:
- **Problem definition is more important than the solution**
- I used lots of AI assistance in research, but AI won't help you correct fundamental research setup errors
- **Experts are still experts, PhDs are still PhDs**—I felt kind of stupid

**Dec 24 Direction Adjustment**

Based on this reflection, Dec 24 needed a major direction change:
1. **Redefine the problem**: Clarify correct variable control
2. **Redesign experiments**: Generate test questions targeting specific tables
3. **Avoid programming toward results**: Decouple test sets from optimization targets

This was a big lesson.

---

*Report generated: 2024-12-24*

---

### Dec 24 (Tue) | Milestone Success: 95.7% Recall Rate 🎉

**Core Achievement**

After discussing with my mentor last night, I discovered a fundamental error in my previous testing methodology. After redesigning the experimental approach, I finally improved retrieval recall rate from ~60% to **95.7%**. This is a very satisfying result.

---

#### 1. Retrospective: Yesterday's Struggles

Yesterday's test results were terrible:

| Difficulty | Recall Rate |
|------------|-------------|
| simple | ~80% |
| medium | ~50% |
| hard | **~20%** |

I started feeling anxious—does my algorithm have no generalizability at all? Is this just "programming toward results" in specific scenarios?

Even more frustrating, I looked at code implementations from many top conference papers (EMNLP, ACL), and they didn't perform well either. I began to wonder: is this direction a dead end?

---

#### 2. Turning Point: Discussion with Mentor

After talking with my mentor last night, I suddenly realized where the problem was:

**Previous Testing Method (Wrong)**:
- Had DeepSeek randomly select tables from 344 tables to generate questions
- Couldn't control variables, question quality was inconsistent
- Many questions themselves were unreasonable

**Correct Testing Method**:
- Fix 3 core tables as the "answer"
- Generate 300 questions using only these 3 tables (100 simple + 100 medium + 100 hard)
- Test whether retrieval algorithm can find these 3 tables from 344 tables

This is proper controlled variable experimentation!

---

#### 3. Today's Work

##### 3.1 Regenerated 300 Test Questions

- **Target tables**: t_bz_config_ci_ne_root, t_bz_config_customer, event_history
- **Question distribution**: 100 simple + 100 medium + 100 hard
- **Verification passed**: All questions only use these 3 tables

##### 3.2 Discovered and Corrected Algorithm Understanding Error

During testing, I found my previous understanding of "fusion strategy" was wrong:

| Strategy | Description | Recall Rate |
|----------|-------------|-------------|
| Summed Score TOP-10 (Wrong understanding) | Table+Column scores summed, then TOP 10 | 81.7% |
| **Union TOP-20 (Correct understanding)** | Table TOP-10 ∪ Column TOP-10 | **95.7%** |

This discovery directly improved recall rate by **14 percentage points**!

##### 3.3 Schema Version Comparison

| Schema | Union Recall Rate |
|--------|------------------|
| **V2 Table-level Enhanced** | **95.7%** |
| V4 LLM Full Enhancement | 70.7% |

V4 actually performed worse, showing **longer descriptions aren't necessarily better**—they may introduce noise.

##### 3.4 Code Organization

Created independent `our_algorithm/` directory, decoupled from top conference paper code frameworks, making code structure clearer.

---

#### 4. Final Results

| Difficulty | ✅Passed | Pass Rate |
|------------|---------|-----------|
| simple | 100/100 | **100%** |
| medium | 96/100 | **96%** |
| hard | 91/100 | **91%** |
| **Total** | **287/300** | **95.7%** |

Very satisfied with these results.

---

#### 5. Personal Reflections

##### On "Working in Isolation"

This experience made me deeply realize the **importance of communication**.

Even with AI assistants like Claude, GPT, and Gemini, I still made a fundamental methodological error—poor test design. AI can help me write code, debug, and optimize algorithms, but it cannot help me escape my own cognitive blind spots.

These "metacognitive" level issues still need experienced people (like mentors) to point out. A single sentence can enlighten me and save potentially days of detours.

##### On "Milestone Success"

95.7% recall rate means the **retrieval stage can be concluded for now**. Next, I can focus on:
- SQL generation optimization
- End-to-end complete testing

Learning this field from scratch and reaching this stage in three weeks is pretty good.

##### Still Much to Learn

Text-to-SQL, Schema Linking, Sparse/Dense retrieval... these are all completely new fields for me. Getting this far is largely due to continuous trial-and-error and timely consultation.

There's still a long road ahead.

---

#### 6. Output Files

| File | Description |
|------|-------------|
| `docs/milestone_20241224/test_questions_3tables_300.json` | 300 test questions |
| `docs/milestone_20241224/retrieval_final_report.md` | Final test report |
| `docs/milestone_20241224/schema_versions.md` | Schema version comparison |
| `our_algorithm/README.md` | Algorithm usage instructions |
| `our_algorithm/retriever.py` | Core retriever (union strategy) |

---

#### 7. Next Steps

| Priority | Task |
|----------|------|
| P0 | Conduct SQL generation test based on 95.7% recall rate |
| P1 | Analyze 13 failed cases, see if further optimization is possible |

---

*Log date: 2024-12-24*
*Core achievement: Recall rate improved from ~60% to 95.7%, retrieval stage milestone completed*

---

### Dec 25 (Wed) | SQL Generation Approach Design & Code Implementation

**Core Achievement**

After completing the retrieval stage optimization, moved into the SQL generation stage. Designed two approaches and completed code implementation with 150-question batch testing.

---

#### 1. Background

On Dec 24, using 3 tables to generate 300 questions for large-scale testing showed good retrieval performance (95.7% recall rate). Now tackling the SQL generation phase.

#### 2. Two SQL Generation Approaches

| Approach | Name | Description |
|----------|------|-------------|
| A | Direct Generation | Pass the complete Schema of 10-20 retrieved tables directly to LLM, let LLM select needed tables and generate SQL |
| B | Filter-Then-Generate | First use LLM to filter 3-5 most relevant tables from 10-20, then generate SQL with refined Schema |

#### 3. Main Work

1. **Improved `table_filter.py`** - Implemented Approach B's table filtering
   - Load table-level Schema (with column info)
   - Use LLM for table filtering

2. **Improved `sql_generator.py`** - Implemented SQL generation
   - Build complete table Schema text (table name, description, column name, column type, column description)
   - Call LLM to generate SQL

3. **150-Question Batch Test**
   - Generated `scheme_a_150_results.json` and `scheme_b_150_results.json`
   - Produced detailed SQL Review reports

---

*Log date: 2024-12-25*
*Core achievement: Two SQL generation approaches designed and implemented, 150-question batch test completed*

---

### Dec 26 (Thu) | End-to-End Testing & Approach Comparison

**Core Achievement**

Completed 7-question end-to-end testing with original questions. Approach A achieved **85.7%** accuracy, significantly outperforming Approach B (28.6%).

---

#### 1. Main Work

1. **7-Question End-to-End Test with Original Questions**
   - Used 7 human-calibrated questions from `cc_result.csv`
   - Complete pipeline: Retrieval → Filtering (Approach B) → SQL Generation
   - Strict comparison with manually verified correct answers

2. **LLM Input Statistics**
   
   | Metric | Value |
   |--------|-------|
   | Average Schema per question | ~9,400 tokens |
   | Percentage of 32B model context | ~29% |
   | Confirmed: Only 10 tables passed | ✅ (not 344) |

3. **Final Test Results**

   | Approach | Accuracy |
   |:--------:|:--------:|
   | **A** | **85.7%** (6/7) |
   | **B** | **28.6%** (2/7) |

4. **Approach B Failure Analysis**
   - Filtering errors: Wrong table selection or missing critical tables (e.g., customer table)
   - SQL generation errors: Missing filter conditions, wrong field names

#### 2. Output Files

| File | Description |
|------|-------------|
| `docs/milestone_20241226/original_7q_full_report.md` | Complete end-to-end report |
| `docs/milestone_20241226/original_7q_results_v2.json` | Test result data |
| `cc_result.csv` | Updated with Approach A/B SQL and evaluation results |

---

#### 3. Personal Reflections

1. **Step by Step, Steady Progress**
   - Any project requires solid, incremental progress
   - The hardest part of this project was the earlier retrieval optimization stage

2. **Post-Retrieval Still Requires Careful Attention**
   - After retrieval, the SQL generation stage also needs careful handling
   - Approach B's filtering logic still has room for optimization

3. **Future Directions**
   - New Year approaching, the hardest retrieval part has been conquered
   - Can continue studying top conference papers (ACL, EMNLP, NeurIPS Text-to-SQL work)
   - Explore better table filtering strategies to improve Approach B accuracy

---

#### 4. Weekly Summary

| Date | Main Work |
|------|-----------|
| 12/22 | Column-level Schema enhancement experiment, 99.2% recall rate |
| 12/23 | Full-scale enhancement comparison, proved core table enhancement beats full-scale |
| 12/24 | Corrected test methodology, 3-table 300-question retrieval test, 95.7% recall |
| 12/25 | SQL generation approach design, code improvements, 150-question batch test |
| 12/26 | 7-question end-to-end test, Approach A/B comparison, report generation |

**Key Conclusion**: Approach A (directly using 10 tables) currently outperforms Approach B (filtering to 3-5 tables first), mainly because Approach B's filtering step is prone to missing critical tables or using wrong fields.

---

*Log date: 2024-12-26*
*Core achievement: End-to-end testing completed, Approach A accuracy 85.7%, SQL generation stage initial milestone reached*

