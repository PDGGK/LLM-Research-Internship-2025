# 🚀 Internship Log | Week 1

> **Duration**: December 2, 2025 (Tue) - December 5, 2025 (Fri)  
> **Research Focus**: Multi-table Join Reasoning for Text-to-SQL in Large-Scale Schemas  
> **Role**: AI Algorithm Intern

---

## 📅 Week 1 Overview

### Dec 2 (Tue) | Onboarding

**Tasks**

- Arrived at the company, started familiarizing with the work environment
- Initial meeting with mentor to understand research direction
- Received project materials and started exploring Text-to-SQL background

**Reflections**

First day at the company. Although my official start date is Friday, I was eager to dive in. My mentor introduced the research focus—Multi-table Join Reasoning for Text-to-SQL, which converts natural language queries into SQL statements. Sounds fascinating!

---

### Dec 3 (Wed) | Paper Reading Begins

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

### Dec 4 (Thu) | Deep Dive into Graph-Based Approaches

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

Honestly, I used to complain that Melbourne's curriculum wasn't helpful for job hunting, as the university doesn't provide campus recruitment like Chinese universities do. But seeing these papers today, I realized that what I learned wasn't useless—I just didn't know where to apply it back then.

---

### Dec 5 (Fri) | Official Start & Team Meeting

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

## 📚 Papers Read This Week

| # | Paper | Core Contribution |
|:---:|:---|:---|
| 1 | **SteinerSQL** (arXiv:2509.19623) | Models Schema Linking as Steiner Tree problem, 40.04% SOTA |
| 2 | **LinkAlign** (arXiv:2503.18596) | Multi-round semantic enhanced retrieval + Query Rewriting |
| 3 | **UNJOIN** (arXiv:2505.18122) | Schema simplification, flattens multi-table to virtual wide table |
| 4 | **SchemaGraphSQL** (arXiv:2505.18363) | Efficient Schema Linking with pathfinding graph algorithms |
| 5 | **CHESS** (arXiv:2405.16755) | Multi-agent framework, context enhancement |
| 6 | **Multi-hop Reasoning with LLMs** (arXiv:2405.09593) | Leveraging LLMs for complex reasoning |

---

## 💭 Week 1 Reflections

### On Research

This is my first algorithm internship and my first time systematically reading arxiv papers. I feel I'm genuinely touching the cutting edge of technology—these papers are freshly published, some haven't even made it to conferences or journals yet.

Many papers are impressive, but after understanding them, I found they're mostly combinations and improvements of existing algorithms, not ground-breaking inventions from scratch. This gives me a more realistic understanding of what pursuing a research Master's or PhD would actually involve.

### On University Curriculum

I used to complain that Melbourne's curriculum wasn't helpful for job hunting. But several papers I read during this week—especially SteinerSQL and SchemaGraphSQL—use graph theory-related algorithmic thinking.

What I learned in school wasn't useless—I just didn't know where to apply it back then.

---

> *"Academic papers point the direction, but engineering implementation is the real challenge."*
