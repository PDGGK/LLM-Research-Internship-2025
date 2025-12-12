# 🚀 Internship Log | Week 1

> **Duration**: December 2, 2024 (Mon) - December 11, 2024 (Thu)  
> **Research Focus**: Multi-table Join Reasoning for Text-to-SQL in Large-Scale Schemas  
> **Role**: AI Algorithm Intern

---

## 📅 Week 1 Overview

### Dec 2 (Mon) | Onboarding

**Tasks**
- Arrived at the company in the evening
- Initial meeting with mentor to understand research direction
- Received project materials and started exploring Text-to-SQL background

**Reflections**

First day at the company. Although my official start date is Thursday, I was eager to dive in. My mentor introduced the research focus—Multi-table Join Reasoning for Text-to-SQL, which converts natural language queries into SQL statements. Sounds fascinating!

---

### Dec 3 (Tue) | Paper Reading Begins

**Tasks**
- Started systematic reading of core Text-to-SQL papers
- Learned about benchmark datasets: Spider, BIRD
- Understood the "Scale Wall" problem: LLM accuracy drops from 86% to 5% when database tables exceed 100

**Papers Read**
1. **CHESS** (arXiv:2405.16755): Contextual Harnessing for Efficient SQL Synthesis - Multi-agent collaboration framework
2. **Using LLMs for Multi-hop Reasoning** (arXiv:2405.09593): Leveraging large language models for complex reasoning

**Reflections**

Started reading papers seriously today. This is my first time systematically reading arxiv papers. These are cutting-edge works—some haven't even been published in top venues yet. I genuinely feel I'm touching the frontier of technology.

After reading the first paper, I realized that with AI assistance, most papers are understandable. The core innovations are often combinations and improvements of existing algorithms, not ground-breaking inventions from scratch. This gives me a more realistic view of what doing a research-focused Master's or PhD would actually be like.

---

### Dec 4 (Wed) | Deep Dive into Graph-Based Approaches

**Tasks**
- In-depth study of **SteinerSQL**: Graph-constrained Text-to-SQL method
- Learned about Steiner Tree algorithm application in Schema Linking
- Read **SchemaGraphSQL**: Efficient Schema Linking with pathfinding algorithms
- Understood multi-dimensional cost functions and graph traversal integration

**Core Papers**
- **SteinerSQL** (arXiv:2509.19623): Graph-Guided Mathematical Reasoning for Text-to-SQL Generation
- **SchemaGraphSQL** (arXiv:2505.18363): Efficient Schema Linking with Pathfinding Graph Algorithms

**Technical Highlights**
- Schema Graph Construction: Model database schema as weighted undirected graph G=(V, E, C)
- Multi-dimensional Cost Function: C_total = α·C_connect + β·C_sem + γ·C_stat
- Terminal Node Mapping: Identify key tables through mathematical decomposition

**Reflections**

Today I encountered papers that use graph theory concepts to solve problems. SteinerSQL proposes using the **Steiner Tree** algorithm for Schema Linking—abstracting database tables as graph nodes, foreign key relationships as edges, then finding optimal paths connecting all key tables using minimum spanning tree-related algorithms.

This reminded me of my graph theory courses at the University of Melbourne. Although the specific algorithm in the paper (Steiner Tree) might be different, the underlying graph theory concepts—modeling complex relationships as graphs and using graph algorithms to find optimal solutions—are exactly what I learned in school.

Honestly, I used to complain that Melbourne's curriculum wasn't helpful for job hunting, as the university doesn't provide campus recruitment like Chinese universities do. But seeing these papers today, I realized that what I learned wasn't useless—I just didn't know where to apply it. Graph traversal, shortest paths, minimum spanning trees—these concepts actually matter in real AI algorithm implementations.

---

### Dec 5 (Thu) | Official Start & Team Meeting

**Tasks**
- **Official first day**: Completed onboarding procedures
- Continued reading: UNJOIN, LinkAlign
- Attended team meeting and presented Text-to-SQL research summary

**Papers Read**
- **UNJOIN** (arXiv:2505.18122): Enhancing Multi-Table Text-to-SQL Generation via Schema Simplification
- **LinkAlign** (arXiv:2503.18596): Scalable Schema Linking for Real-World Large-Scale Multi-Database Text-to-SQL

**Presentation Content**
- Introduced the "Scale Wall" problem and three core challenges: Semantic Retrieval Blindness, Topological Hallucination, Math-Schema Gap
- Detailed explanation of SteinerSQL's graph algorithm engine
- Overview of LinkAlign semantic alignment and UNJOIN schema simplification

**Reflections**

Officially started today! The afternoon presentation made me a bit nervous, but it went well overall. My mentor was satisfied with the research summary and suggested some directions for practical implementation.

Presenting these papers' core techniques helped me develop a more systematic understanding. I also realized that understanding a paper is one thing, but explaining it clearly is another skill entirely.

---

### Dec 6 (Fri) | LinkAlign Implementation Exploration

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

### Dec 9 (Mon) | Environment Setup & Data Preparation

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

### Dec 10 (Tue) | Schema Linking Testing & Optimization

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

### Dec 11 (Wed) | Deeper Understanding of LinkAlign & Helping Colleague Debug

**Tasks**
- Continued testing LinkAlign, gaining deeper understanding of its core mechanism
- Helped colleague debug a voice dialogue project's human-agent handoff feature

**True Understanding of LinkAlign**

Today while continuing to test LinkAlign, I discovered that I hadn't truly understood what it was doing. I previously thought its operation involved two LLMs conversing with each other to derive the optimal SQL. That's completely wrong—it actually uses its own vector retriever for Schema Linking, then hands off to the LLM for SQL generation. I was stunned. It's my fault for not truly understanding the paper.

I used AI to help me read the paper, but Gemini 1.5 Pro might have higher hallucination rates and misled me about the paper's core mechanism. Reflecting on this: facing such professional, lengthy papers, if I don't rely on AI at all, research speed would be very slow; but if I don't thoroughly understand the problem before starting implementation, I'll end up spending even more time on corrections later. It's a dilemma.

Going forward, I need to study each paper's specific operations more carefully, not just look at the general architecture. Continue trying more test data.

**Additional Task**

A colleague in the same office asked for help with a problem. Their project does sentiment recognition in voice dialogues—when negative emotions are detected, it automatically routes to human customer service. The project was entirely AI-written, but the human-agent handoff feature wasn't working properly.

I used AI to analyze the code step by step, write tests, and locate the problem. Finally discovered it was a JavaScript scope issue—a variable wasn't correctly binding `this` in a callback function.

I'm not very familiar with JavaScript myself. I self-taught the front-end basics a while ago, not very systematically. But with AI assistance, I managed to solve the problem.

**Reflections**

Two deep insights today:

**On Paper Reading:** AI-assisted paper reading is a double-edged sword. It definitely accelerates comprehension, but if the AI's interpretation is wrong (hallucination), you might waste a lot of time going in the wrong direction. Core technical details still need to be carefully extracted from the original paper—you can't completely rely on AI summaries.

**On Helping Colleague Debug:** As an intern, why was I able to solve a problem that a full-time employee couldn't? First, I think I'm better at using AI—having AI analyze problems step by step, rather than throwing it one big problem. First have AI read and understand the code architecture, then have it write test cases for specific functions, finally narrow down the problem range based on test results. Second is mindset—often it's not about how hard the problem is, but whether you have the patience to troubleshoot step by step.

---

## 💭 Week 1 Reflections

### On Research

This is my first algorithm internship and my first time systematically reading arxiv papers. I feel I'm genuinely touching the cutting edge of technology—these papers are freshly published, some haven't even made it to conferences or journals yet.

Many papers are impressive, but after understanding them, I found they're mostly combinations and improvements of existing algorithms, not ground-breaking inventions from scratch. This gives me a more realistic understanding of what pursuing a research Master's or PhD would actually involve—research isn't as unreachable as I previously imagined.

### On University Curriculum

I used to complain that Melbourne's curriculum wasn't helpful for job hunting, as the university doesn't provide campus recruitment. But several papers I read during this internship—especially SteinerSQL and SchemaGraphSQL—use graph theory-related algorithmic thinking. Although the specific algorithms might differ, the underlying concept—modeling complex problems as graphs and solving with graph algorithms—is exactly what Melbourne's graph theory courses taught.

What I learned in school wasn't useless—I just didn't know where to apply it back then.

### On AI and the Future

Compared to my internship last year, AI has become much more powerful. Chatting with my previous internship mentor, he said: "If AI development continues at this pace for two more years, developers might become obsolete. Currently 2-3 developers with AI can complete a project; in the future, one developer with AI could handle 3 projects."

AI development has made learning computer science much easier—it's like having a 24/7 tutor. But at the same time, AI development is also squeezing programmers' future employment space.

I originally planned to pursue backend development or full-stack development. But with AI's advancement, I think I need to pivot toward AI algorithm-related directions—not abandoning development, but doing AI project development and AI algorithm engineering implementation.

I'm very grateful to have this algorithm internship opportunity as an undergraduate. Determining this direction is very important for me.

---

## 📚 Papers Read This Week

| # | Paper | Core Contribution |
|:---:|:---|:---|
| 1 | **SteinerSQL** (arXiv:2509.19623) | Models Schema Linking as Steiner Tree problem, 40.04% SOTA |
| 2 | **LinkAlign** (arXiv:2503.18596) | Multi-round semantic enhanced retrieval + Query Rewriting |
| 3 | **UNJOIN** (arXiv:2505.18122) | Schema simplification, flattens multi-table to virtual wide table |
| 4 | **SchemaGraphSQL** (arXiv:2505.18363) | Efficient Schema Linking with pathfinding graph algorithms |
| 5 | **CHESS** (arXiv:2405.16755) | Multi-agent framework, context enhancement |
| 6 | **Multi-hop Reasoning with LLMs** (arXiv:2405.09593) | Leveraging LLMs for complex reasoning |
| 7 | **Text-to-SQL Multi-table Join Prediction** | Multi-table join prediction and Schema analysis |
| 8 | **(arXiv:2505.18744)** | Retrieval enhancement methods |

---

## 🎯 Next Week Plan

1. Continue optimizing LinkAlign retrieval accuracy
2. Try incorporating graph algorithm concepts for multi-table join reasoning
3. Organize experimental results and prepare for next team meeting

---

> *"Academia and industry are different—papers look elegant, but the real challenges begin when you try to implement them."*
