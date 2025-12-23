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
