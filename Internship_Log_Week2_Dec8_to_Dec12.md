# 🚀 Internship Log | Week 2

> **Duration**: December 8, 2025 (Mon) - December 12, 2025 (Fri)  
> **Research Focus**: Multi-table Join Reasoning for Text-to-SQL in Large-Scale Schemas  
> **Role**: AI Algorithm Intern

---

## 📅 Week 2 Overview

### Dec 8 (Mon) | LinkAlign Implementation Exploration

**Tasks**

- Started studying **LinkAlign** source code
- Set up LinkAlign local development environment
- Learned project architecture: Multi-round semantic enhanced retrieval + Schema Item Grounding
- Continued reading related papers

**Papers Read**

- **Text-to-SQL Multi-table Join Prediction and Schema**: Multi-table join prediction methods
- **(arXiv:2505.18744)**: Related retrieval enhancement methods

**Technical Architecture**

1. Draft Retrieval: Use vector search to get Top-K candidate tables
2. Schema Audit: LLM audits the completeness of retrieval results
3. Query Rewriting: Generate enhanced queries with Schema keywords
4. Final Retrieval: Second retrieval to fill in missed tables

**Reflections**

From today, I'm transitioning from "reading papers" to "implementing papers." This shift made me realize that academia and industry are different. Methods that look elegant in papers face various challenges during real-world implementation.

The experimental environments, datasets, and hyperparameter configurations in papers are different from our actual business scenarios. They use the Spider 2.0 dataset; we're connecting to a real business database with 344 tables and over 3,000 columns. This scale difference brings many challenges.

---

### Dec 9 (Tue) | Environment Setup & Data Preparation

**Tasks**

- Completed LinkAlign project local environment configuration
- Set up Python virtual environment, installed dependencies (LlamaIndex, sentence-transformers, etc.)
- Modified LlamaIndex source code to adapt to project requirements
- Wrote MySQL Schema extraction script, successfully extracted 344 tables and 3,353 columns from business database

**Technical Details**

- Using Qwen3-14B-awq as local LLM
- Using BGE-large-en-v1.5 as Embedding model
- Wrote `extract_mysql_schema.py` script to extract real business database Schema

**Reflections**

Setting up the environment is always the most frustrating part. Spent most of the day dealing with dependency conflicts, network issues, and model downloads. But seeing the Qwen API test pass and Schema successfully extracted was quite satisfying.

---

### Dec 10 (Wed) | Schema Linking Testing & Optimization

**Tasks**

- Ran LinkAlign Schema Linking tests
- Wrote `test_optimization.py`: Compare different modes (Agent/Pipeline) and parameters (top_k, turn_n)
- Wrote `test_query_expansion.py`: Implement query expansion mechanism to solve Chinese-English mixed retrieval issues
- Analyzed retrieval results, evaluated recall and precision

**Optimization Approaches**

- Implemented cross-lingual query expansion: Expand Chinese queries to include English keywords
- Compared various retrieval strategies

**Key Findings**

- Agent mode performs better on complex queries but takes longer
- Query expansion significantly improves retrieval recall for Chinese-English mixed Schemas
- similarity_top_k and turn_n parameter tuning greatly affects precision

**Reflections**

Today's testing gave me a deep appreciation of the difficulties in paper implementation. The metrics reported in papers are obtained on standard datasets, but our real business database has many "dirty" aspects: table names in English, comments in Chinese, inconsistent field naming, etc.

Users ask about "告警" (alerts), but the table is named `event_history`; users ask about "客户" (customers), but the table is `t_bz_config_customer`. This semantic gap isn't mentioned in papers, but it must be solved in real-world implementation.

So I wrote a query expansion module that has the LLM translate Chinese into various possible English variants before retrieval. The effect improved, but there's still a gap from ideal. This is probably the difference between research and engineering.

---

### Dec 11 (Thu) | Deeper Understanding of LinkAlign & Helping Colleague Debug

**Tasks**

- Continued testing LinkAlign, gaining deeper understanding of its core mechanism
- Helped colleague debug a voice dialogue project's human-agent handoff feature

**True Understanding of LinkAlign**

Today while continuing to test LinkAlign, I discovered that I hadn't truly understood what it was doing. I previously thought its operation involved two LLMs conversing with each other to derive the optimal SQL. That's completely wrong—it actually uses its own vector retriever for Schema Linking, then hands off to the LLM for SQL generation. I was stunned. It's my fault for not truly understanding the paper.

I used AI to help me read the paper, but Gemini 1.5 Pro might have higher hallucination rates and misled me about the paper's core mechanism. Reflecting on this: facing such professional, lengthy papers, if I don't rely on AI at all, research speed would be very slow; but if I don't thoroughly understand the problem before starting implementation, I'll end up spending even more time on corrections later. It's a dilemma.

Going forward, I need to study each paper's specific operations more carefully, not just look at the general architecture.

**Additional Task**

A colleague in the same office asked for help with a problem. Their project does sentiment recognition in voice dialogues—when negative emotions are detected, it automatically routes to human customer service. The project was entirely AI-written, but the human-agent handoff feature wasn't working properly.

I used AI to analyze the code step by step, write tests, and locate the problem. Finally discovered it was a JavaScript scope issue—a variable wasn't correctly binding `this` in a callback function.

I'm not very familiar with JavaScript myself. I self-taught the front-end basics a while ago, not very systematically. But with AI assistance, I managed to solve the problem.

**Reflections**

Two deep insights today:

**On Paper Reading:** AI-assisted paper reading is a double-edged sword. It definitely accelerates comprehension, but if the AI's interpretation is wrong (hallucination), you might waste a lot of time going in the wrong direction. Core technical details still need to be carefully extracted from the original paper—you can't completely rely on AI summaries.

**On Helping Colleague Debug:** As an intern, why was I able to solve a problem that a full-time employee couldn't? First, I think I'm better at using AI—having AI analyze problems step by step, rather than throwing it one big problem. First have AI read and understand the code architecture, then have it write test cases for specific functions, finally narrow down the problem range based on test results. Second is mindset—often it's not about how hard the problem is, but whether you have the patience to troubleshoot step by step.

---

### Dec 12 (Fri) | LinkAlign Deep Analysis & Optimization Attempts

**Tasks**

- Conducted deep analysis and optimization attempts on LinkAlign
- SQL Comparison Analysis: Analyzed 7 test cases, compared LinkAlign-generated SQL with correct SQL
- Root Cause Investigation: Discovered 50% of Schema metadata (column descriptions) were missing
- Designed and implemented Schema Auto-Enhancement solution
- Modified SQL generation Prompt template
- Validation testing and identified deeper algorithm issues

**SQL Comparison Analysis**

Detailed comparison of SQL generation results for each test case:
- SQL Accuracy: 0/7 (0%)
- Error causes:
  - Wrong table selection (e.g., using `report_new_dev` instead of `t_bz_config_ci_ne_root`)
  - Forced JOIN causing meaningless associations
  - LLM output containing excessive `<think>` tags

**Root Cause Investigation**

Found two major issues when examining Schema files:

1. **Schema Metadata Missing**:
   - Examined `extract_mysql_schema.py` source code
   - Confirmed schema is extracted from MySQL database's `COLUMN_COMMENT`
   - Statistics: 1,679 out of 3,353 columns (50%) have null descriptions
   - LinkAlign doesn't include a schema generation tool; papers use high-quality public datasets (Spider/BIRD)

2. **Unreasonable Prompt Template**:
   - Original Prompt forced JOIN for multi-table queries
   - Simple queries don't need forced JOINs

**Schema Auto-Enhancement Solution**

Design: If database columns lack comments, use LLM to auto-generate descriptions based on column name, type, and sample data

Implementation:
- Created `extract_mysql_schema_enhanced.py` - full enhancement version
- Created `enhance_core_tables.py` - quick core table enhancement version
- Optimized Prompt to explicitly prohibit thinking process
- Added response cleaning function to remove `<think>` tags and noise

Test results:
- 3 core tables: `t_bz_config_ci_ne_root`, `t_bz_config_customer`, `event_history`
- Processed 155 columns, LLM generated 34 descriptions

**Validation Testing & Key Discovery**

Test case 5: "How many new devices came online last month"

| Stage | Result |
|:------|:-------|
| Vector Retrieval | ✅ Found `t_bz_config_ci_ne_root` |
| Reserve Protection | ⚠️ Not in protection list |
| LLM Filter Round 1 | ❌ Filtered out (142 tables → 53 tables) |
| Table Recall | 0% |

**Unexpected Discovery: Not a Schema description issue, but a LinkAlign filtering algorithm flaw!**

LinkAlign uses column voting to determine table relevance:
```
t_bz_config_ci_ne_root (75 columns):
  - ONLINE_TIME: relevant ✅
  - Other 74 columns: not relevant ❌
  
Vote result: 1/75 = 1.3%
Decision: Filter out entire table ❌
```

Correct logic should be: If any key column is relevant → Keep entire table

**Technical Output**

- `extract_mysql_schema_enhanced.py` - Full enhancement version
- `enhance_core_tables.py` - Quick core table enhancement version
- `quick_validate.py` - Single case quick validation script
- `sql_analysis_report.md` - SQL comparison analysis report
- `validation_report.md` - Schema enhancement validation report

**Reflections**

Today's work demonstrated a complete problem-solving process:

1. **Problem-oriented Research Method**: Observation → Initial Analysis → Hypothesis → Validation Experiment → Refute Hypothesis → Deep Investigation → New Hypothesis

2. **Academia vs Engineering Gap**: Papers focus on algorithm innovation under ideal conditions; engineering needs to handle real-world complexity and imperfect data. Bridging this gap requires extensive adaptation and optimization work.

3. **Schema Enhancement Didn't Solve the Problem**: I thought it was caused by missing Schema descriptions, spent 2 hours implementing the enhancement solution, only to discover the problem lies in the deeper filtering algorithm logic. A good lesson—confirm root cause before implementing solutions.

---

## 💭 Week 2 Reflections

### On AI and the Future

Compared to my internship last year, AI has become much more powerful. Chatting with my previous internship mentor, he said: "If AI development continues at this pace for two more years, developers might become obsolete. Currently 2-3 developers with AI can complete a project; in the future, one developer with AI could handle 3 projects."

AI development has made learning computer science much easier—it's like having a 24/7 tutor. But at the same time, AI development is also squeezing programmers' future employment space.

I originally planned to pursue backend development or full-stack development. But with AI's advancement, I think I need to pivot toward AI algorithm-related directions—not abandoning development, but doing AI project development and AI algorithm engineering implementation.

I'm very grateful to have this algorithm internship opportunity as an undergraduate. Determining this direction is very important for me.

---

## 📚 Papers Read This Week

| # | Paper | Core Contribution |
|:---:|:---|:---|
| 1 | **Text-to-SQL Multi-table Join Prediction** | Multi-table join prediction and Schema analysis |
| 2 | **(arXiv:2505.18744)** | Retrieval enhancement methods |

---

## 🎯 Next Week Plan

1. Continue optimizing LinkAlign retrieval accuracy
2. Try incorporating graph algorithm concepts for multi-table join reasoning
3. Organize experimental results and prepare for next team meeting

---

> *"Academia and industry are different—papers look elegant, but the real challenges begin when you try to implement them."*
