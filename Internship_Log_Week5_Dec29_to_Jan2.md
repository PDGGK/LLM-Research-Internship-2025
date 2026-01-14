# 🚀 Internship Log | Week 5

> **Duration**: December 29, 2024 (Sun) - January 2, 2025 (Thu)
> **Research Focus**: Multi-table Join Reasoning for Text-to-SQL in Large-Scale Schemas
> **Role**: AI Algorithm Intern

---

## 📅 Week 5 Overview

### Dec 30 (Mon) | Schema Enum Value Enhancement & Model Comparison Testing

**Core Achievement**

Based on mentor's suggestions, implemented Schema enum value enhancement. Qwen2.5-32B + Schema enhancement achieved **100% accuracy on Simple questions**! Also completed code reorganization into an independent module.

---

#### 1. Model Comparison Test (Unified Prompts)

Discovered that previous comparison tests had **inconsistent prompts**. Re-ran 150-question test with unified prompts:

| Model | Accuracy |
|:------|:--------:|
| Qwen2.5-32B | 80/150 (53.3%) |
| Qwen3 MoE | 94/150 (62.7%) |

**Finding**: For alert-related queries, 32B frequently misselected `event_sdn` table instead of the correct `event_history` table.

#### 2. Mentor's Suggestion - Schema Enum Value Enhancement

**Problem**: Model couldn't determine which table to use for "alerts" from Schema descriptions

**Solution**: Write TOP10 enum values into Schema descriptions

```
# Before enhancement
EVENT_NAME (varchar): Event name

# After enhancement
EVENT_NAME (varchar): Event name. Common values: IP SLA State, Temperature State, Bgp peer state, Device state, CPU Utilization...
```

**Enhanced fields**:
- EVENT_NAME
- EVENT_TYPE_NAME
- EVENT_STATUS_NAME
- SEVERITY_NAME

#### 3. Post-Enhancement Verification

**7-Question Test**:

| Test | 32B | MoE |
|:-----|:---:|:---:|
| Before Enhancement | 6/7 (85.7%) | 5.5/7 (78.6%) |
| After Enhancement | **7/7 (100%)** | **7/7 (100%)** |

**150-Question Test**:

| Difficulty | 32B Before | 32B After | Improvement |
|:-----------|:----------:|:---------:|:-----------:|
| Simple | 36/50 (72%) | **50/50 (100%)** | +14 |
| Medium | 30/50 (60%) | 29/50 (58%) | -1 |
| Hard | 14/50 (28%) | 16/50 (32%) | +2 |
| **Total** | 80/150 (53.3%) | **95/150 (63.3%)** | **+15** |

**Key Achievement**: Qwen2.5-32B + Schema enhancement, **Simple questions reached 100% accuracy**!

#### 4. Retrieval Success Rate Verification

Confirmed Schema enhancement doesn't affect retrieval performance:

| Test | Before | After | Change |
|:-----|:------:|:-----:|:------:|
| 300 questions | 287/300 (95.7%) | 286/300 (95.3%) | -1 |

**Conclusion**: Retrieval success rate remains stable.

#### 5. Code Reorganization into Independent Module

Per mentor's requirements, reorganized the full pipeline code into an independently runnable module:

**New files**:
- `our_algorithm/__init__.py` - Package entry
- `our_algorithm/pipeline.py` - Unified Pipeline entry
- `our_algorithm/config.py` - Independent config file
- `our_algorithm/llm/qwen.py` - LLM wrapper

**Usage**:
```python
from our_algorithm import TextToSQL

pipeline = TextToSQL()
sql = pipeline.run("Query how many devices the test customer has")
```

---

#### 6. Output Files

| File | Description |
|:-----|:------------|
| `docs/milestone_20241230_qwen3_moe/150q_enhanced_full_review.md` | 150-question manual review report |
| `docs/milestone_20241230_qwen3_moe/150q_enhanced_schema_test.json` | Post-enhancement test data |
| `spider2_dev/schemas_table_level_enhanced/netcaredb_ai/event_history.json` | Enhanced Schema |
| `our_algorithm/pipeline.py` | Unified Pipeline entry |
| `our_algorithm/README.md` | Algorithm module documentation |

---

#### 7. Key Conclusions

1. **Schema enum value enhancement is effective**: Writing TOP10 enum values into Schema significantly improves SQL accuracy
2. **32B + Schema enhancement works best**: Simple questions improved from 72% to 100%
3. **Code reorganized into independent module**: `our_algorithm` can be used as a standalone package

#### 8. Next Steps

| Priority | Task |
|----------|------|
| P0 | Continue optimizing Medium/Hard question accuracy |
| P1 | Explore more Schema enhancement strategies |
| P2 | Complete top conference paper reading (ACL, EMNLP related work) |

---

*Log date: 2024-12-30*
*Core achievement: Schema enum value enhancement brought Simple question accuracy to 100%, code reorganized into independent module*

