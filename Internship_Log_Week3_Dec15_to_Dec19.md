# 🚀 Internship Log | Week 3

> **Duration**: December 15, 2025 (Mon) - December 19, 2025 (Fri)  
> **Research Focus**: Multi-table Join Reasoning for Text-to-SQL in Large-Scale Schemas  
> **Role**: AI Algorithm Intern

---

## 📅 Week 3 Overview

### Dec 15 (Mon) | Schema Linking Multi-Strategy Comparison

**Tasks**

- Continued Schema Linking algorithm optimization research
- Designed and implemented table-level Schema extraction approach
- Compared differences between our database and academic datasets (Spider)
- Tested pure LLM direct table selection approach
- Explored query translation and multilingual embedding model optimization

---

**Dataset Comparison Analysis**

Comparison between netcaredb_ai and academic standard dataset Spider:

| Metric | netcaredb_ai | Spider 1.0 | Spider 2.0-Lite |
|--------|--------------|------------|-----------------|
| Tables | 344 | 5-10/db | Enterprise-scale |
| Columns | 3,353 | ~28/db | 803.6/db |
| Foreign Key Coverage | 17% | ~100% | Partial |
| SOTA Accuracy | TBD | 85-91% | **33-40%** |

**Conclusion**: Academic methods achieve only 33-40% on Spider 2.0-Lite, limited reference value for enterprise scenarios.

---

**Pure LLM Table Selection Test**

**Approach**: 344 tables ≈ 6k tokens, within LLM context, skip vector retrieval and let LLM select tables directly.

**Test Results (7 business questions)**:

| Metric | Result |
|--------|--------|
| Average Recall (Chinese desc) | **21.4%** |
| Customer table `t_bz_config_customer` | 100% recall ✅ |
| Device table `t_bz_config_ci_ne_root` | 0% recall ❌ |

**Analysis**: Device main table description not prominent enough, LLM selected other "device"-related tables (e.g., `sdw_dp_device`).

---

**Language Alignment Experiment**

**Hypothesis**: English question + English table description + English embedding model → better vector retrieval.

**Results**:

| Approach | Recall |
|----------|--------|
| Chinese question + Chinese desc | 0% |
| English question + English desc | 0% |

**Conclusion**: Language translation alone cannot solve vector retrieval issues; root cause is semantic expression in table descriptions, not language alignment.

---

### Dec 16 (Tue) | Comprehensive Table-Level Retrieval Evaluation

**Today's Work**

A very productive day focusing on **table-level retrieval** for LinkAlign. Completed full experimental cycle from baseline testing to deep optimization. Generated **7 technical reports**, organized **14 test scripts**, and reached important conclusions on retrieval strategy selection.

---

**Morning: Baseline Testing**

**BGE-M3 Vector Retrieval Optimization** (`docs/bge_m3_optimization_report.md`):
- Enhanced 3 core table descriptions, recall improved from **57.1% to 76.2%**
- Key changes:
  - `event_history`: Added "alert" keywords
  - `t_bz_config_ci_ne_root`: Marked as "device main table/core table"
  - `t_bz_config_customer`: Marked as "customer main table"

**Column-Level vs Table-Level Comparison** (`docs/column_vs_table_level_comparison.md`):
- Conclusion: Table-level (76.2%) outperforms column-level LinkAlign (52%)
- Discovered LinkAlign's "inherent flaw": core tables incorrectly filtered during LLM filtering stage

---

**Afternoon: Hybrid Retrieval Experiments**

**Dense vs Hybrid Comparison** (`docs/bge/dense_vs_hybrid_comparison.md`):
- Configuration: Dense:Sparse = 0.7:0.3 hybrid retrieval
- Result: No significant recall improvement (74% vs 74%)
- Analysis: Table descriptions lack keywords, Sparse vectors ineffective

---

**Evening: Deep Analysis & Code Refactoring**

**Schema Extraction Script Optimization (Important Change)**

Modified: `extract_table_level_schema.py`

Change: Modified `generate_embedding_text()` function, **removed `max_columns` parameter limit**

| Table | Before (20-col limit) | After (no limit) |
|-------|----------------------|------------------|
| `t_bz_config_ci_ne_root` | 20 columns | **75 columns** |
| `HOST_NAME` column | ❌ Truncated | ✅ Included |
| `embedding_text` length | ~500 chars | **1451 chars** |

**LLM Batch Table Description Enhancement (Systematic Approach)**

New file: `enhance_schema_with_llm.py`

Note:
- **No manual modifications**: LLM automatically generated descriptions for 344 tables, no special handling for test cases
- Comparison:

| Table | Before (manual) | After (LLM) |
|-------|----------------|-------------|
| `event_history` | 【Alert/Event Main Table】Stores all device alerts... | Event history table, stores detailed system event info... |
| `t_bz_config_ci_ne_root` | 【Device Main Table/Core】... | Root NE config table, stores network device basic info... |

> ⚠️ **Note**: LLM-generated descriptions lack "alert", "main table/core" markers, causing recall drop from 76% to 74%.

---

**Recall Comparison Summary**

| Approach | Recall | Execution Time |
|----------|--------|----------------|
| Column-level LinkAlign | 52% | ~15 min |
| Table-level (manual desc) | **76.2%** | ~10 sec |
| Table-level (LLM desc) | 74% | ~10 sec |
| Table-level + Hybrid | 74% | ~10 sec |

---

### Dec 17 (Wed) | Major Breakthrough: Table+Column Fusion 🎉

**Major Achievement**

**Table-level + Column-level Fusion Retrieval achieved 95% recall!**

---

**Core Results**

After a full day of testing and validation, we found the optimal Schema Linking retrieval approach:

| Approach | Recall | Full Recall |
|----------|--------|-------------|
| Table-level Sparse | 74% | 4/7 |
| Column-level Sparse | 83% | 4/7 |
| **Table+Column Fusion (Union)** | **95%** | **6/7** ✅ |

**Key Findings**

1. **Sparse outperforms Dense**: In Chinese Schema retrieval, Sparse (lexical) 74% significantly outperforms Dense (semantic) 45%
2. **Table+Column are complementary**: Table-level finds some tables, column-level finds others, union greatly improves recall
3. **Cumulative scoring beats voting**: Avoids LinkAlign's original "one relevant column drowned by many irrelevant columns" problem

---

**Research Journey Retrospective**

**Why SteinerSQL's Graph Theory Approach Doesn't Fit Our Database**

While studying papers, I researched graph-based methods like SteinerSQL. Its core idea: model database Schema as a graph, use Steiner Tree algorithm to find the minimum subgraph connecting query-relevant tables.

Sounds impressive, but **doesn't fit our database**.

Our database characteristics:
- **Denormalized design**: Wide tables, not standard 3NF normalized design
- **Extremely low FK coverage**: 344 tables with only ~50 foreign keys, only **17%** coverage
- **80%+ tables unconnected**: Most tables are independent, no FK relationships

| Metric | Our Database | Academic Datasets |
|--------|--------------|-------------------|
| Tables | 344 | Dozens |
| FK Coverage | **17%** | 60-80% |
| Design Style | Denormalized/Wide | Normalized |
| Table Relations | 80%+ unconnected | Most have FKs |

**Graph methods require** rich FK relationships to build meaningful graphs. But our tables are mostly "isolated islands" that can't form a connected graph.

This made me realize: **Academic methods can't be blindly applied; you must consider data characteristics.**

---

**The Inspiration: Table+Column Union**

After multiple tests and discussions, I discovered table-level and column-level retrieval results are complementary:
- Test 1: Table-level finds event_history, column-level finds t_bz_config_ci_ne_root → Fusion: 100%
- Test 7: Table-level finds event_history, column-level finds two other tables → Fusion: 100%

**Take the union**, directly achieved 95% recall!

---

**Afternoon: Post-Recall Improvements**

**Column-Level Filtering Experiment**

**Goal**: After recalling enough tables, try filtering irrelevant columns to reduce info for LLM.

**Result**: ❌ **Failed**

| Threshold | Retained Cols | SQL Required Cols Recall |
|-----------|---------------|-------------------------|
| 0.05 | 12/155 | 0% |
| 0.02 | 45/155 | ~30% |

**Failure Reason**:
- SQL-required columns (like `ci_id`, `event_time`) have low semantic similarity to questions
- These columns have low scores but are crucial for SQL generation
- Filtering loses critical columns

**SQL Generation Quality Comparison**

**Core Finding**: Giving LLM more tables actually degrades SQL quality!

| Approach | Tables | Tokens | SQL Accuracy |
|----------|--------|--------|--------------|
| Original 3 tables | 1-3 | ~1000 | **71%** |
| Full format | 10-12 | ~3000 | 14% |
| Compact format | 10-12 | ~2000 | 14% |
| Minimal format | 10-12 | ~1200 | 14% |

**Conclusion**:
- **Table count is key**: 10+ tables causes 57% quality drop
- **Format compression can't compensate**: Quality doesn't improve with compressed format
- **Core issue isn't format, it's too many tables**

---

**Reflections**

Although we've only achieved "how to find needed tables from thousands"—this is already major progress.

From reading papers from scratch, to discovering algorithm flaws, to proposing improvements, to achieving 95% recall—this process taught me a lot.

The afternoon work made me realize: **Recall is just step one; giving LLM the right amount of info is another challenge**. Too much info actually interferes with LLM; need to research how to select 2-3 core tables.

Most important insight: **Read papers critically**. arXiv papers aren't all perfect; some algorithms have inherent flaws. Only by testing hands-on and letting data speak can you find real problems and solutions.

---

### Dec 19 (Fri) | Building Test Dataset & Systematic Evaluation 📊

**Today's Work**

Focus on **building test dataset and systematic evaluation**, establishing complete testing benchmarks for the Text-to-SQL system.

---

**Generated 100 Test Questions Using DeepSeek**

**Background**: Previous sample size too small (only 7), cannot do systematic evaluation.

**Implementation**:
- Wrote `scripts/generate_test_questions.py` script
- Used DeepSeek API (`deepseek-chat` model)
- Read all 344 table Schemas, had LLM generate questions covering various scenarios

**Generation Results**:

| Difficulty | Count | Description |
|------------|-------|-------------|
| easy | 29 | Single-table simple stats/queries |
| medium | 52 | 2-3 table joins, condition filters |
| hard | 19 | Multi-table JOIN, complex aggregations |

**Core Table Usage**:
1. `t_bz_config_ci_ne_root` - 33 times
2. `t_bz_config_customer` - 29 times
3. `event_history` - 13 times

---

**Table Recall Test (100 samples)**

**Method**: Table+Column fusion retrieval (BGE-M3 Sparse mode)

**Overall Results**:

| Metric | Result |
|--------|--------|
| Full Recall | 65% |
| Partial Recall | 28% |
| No Recall | 7% |
| **Average Recall** | **78.8%** |

**By Difficulty**:

| Difficulty | Full Recall Rate |
|------------|------------------|
| easy | **93.1%** |
| medium | **69.2%** |
| hard | **52.6%** |

**Key Findings**:
- Easy questions perform excellently, single-table queries mostly fine
- Hard questions challenging, multi-table JOIN recall insufficient
- `t_bz_config_ci_ne_root` and `t_bz_config_customer` often missing in multi-table queries

---

**SQL Generation Test (Qwen2.5-Coder-32B-Instruct)**

**Test Config**:
- Model: Qwen2.5-Coder-32B-Instruct
- API: http://172.31.24.112:33080/v1
- Test Questions: 100

**Comparing Two Prompt Modes**:

| Mode | Correct | Accuracy |
|------|---------|----------|
| Without Thinking | 60/100 | **60%** |
| With Thinking | 66/100 | **66%** |

**Conclusion**: Enabling Thinking (let model think before answering) improved by **6 percentage points**

---

**Key Conclusions**

1. **Test dataset established**: 100 questions covering easy/medium/hard difficulties
2. **Recall bottleneck identified**: Hard questions only 52.6% full recall, need to optimize multi-table strategy
3. **Thinking mode effective**: Enabling Thinking improves accuracy by 6%
4. **Main error types**: Missing ORDER BY, SELECT field mismatch, missing JOIN

---

## 💭 Week 3 Reflections

### On Critical Paper Reading

The most important lesson this week: **Read papers critically**. arXiv papers aren't perfect—some algorithms have fundamental flaws. LinkAlign's voting mechanism is a prime example: one relevant column gets drowned by 49 irrelevant columns.

Only by hands-on testing can you discover real problems and devise better solutions.

### On AI-Assisted Research

AI helped tremendously this week:
- **Speed**: Verified dozens of ideas in a single day
- **Breadth**: Quickly tested multiple combinations (Dense/Sparse × Table/Column × Various scoring methods)
- **Efficiency**: What would take weeks manually completed in days

But AI alone isn't enough—**mentor guidance** is equally important. The questions my mentor asked directed my investigation toward the right paths.

### On Engineering vs Academia

Academic methods achieve 33-40% on Spider 2.0-Lite; our enterprise database is even more challenging. The gap between academia and industry is real:
- Enterprise data is messy
- FK coverage is low
- Mixed Chinese-English naming
- Denormalized designs

You can't blindly apply academic methods—you must adapt to data characteristics.

---

## 📚 Technical Output This Week

| Type | Count |
|------|-------|
| Technical Reports | 15+ |
| Test Scripts | 20+ |
| Test Questions Generated | 100 |
| Core Algorithms Tested | 10+ |

---

## 🎯 Next Week Plan

1. **Column-level description enhancement**: Use LLM to improve column descriptions
2. **Embedding precomputation**: Speed up retrieval with cached embeddings
3. **Full SQL generation testing**: Complete end-to-end evaluation

---

> *"Recall is just step one; giving LLM the right amount of info is another challenge."*
