---
lastSync: Sat May 03 2025 15:05:07 GMT+0530 (India Standard Time)
title: "Relational Deep learning : Machine learning on Relational Databases"
draft: true
tags:
  - graph_nn
  - research_paper_blog
---

#### **Introduction**

Significant portion machine learning problems deals with tabular data. With problems ranging from churn prediction,  classification, forecasting etc.

- Hook: A real-world pain point or question the paper addresses
- Brief overview: What the research is about and why it matters

#### **Problem Statement**

How do Traditional Machine Learning Pipelines look like, how to traditional machine pipelines?
1. Creation of a feature store creating from joining multiple table extracting only relevant information.
2. 
Problems with existing approach?
2. Feature Engineering is Manual, slow and Labor intensive process.
3. Sub-optimal feature choices as entire spectrum of features doesn't get explored.
4. Re-computation and/or redesign of features in case of data distribution drifts. 

#### **Key Innovation / Contribution**

- What’s the new idea or breakthrough?
- Forgoes the need of hand engineering features.
- Avoid re-computation of features in case of data drifts
- Can model multi task learning tasks with a standalone graph representation by leveraging task specific head.
- Represent relational databases as a graph using primary key, foreign key relationships. 
- Learning to represent of linked entities as tables of features using GNNs.
- How does it work at a high level?
1. Training Table containing supervision labels constructed in task-specific manner based on historic data in the relational database.
2. Entity level features are extracted and encoded from each row in the table
3. Node Representations learned through inter entity linking message passing GNN that exchanges information  between entities with primary foreign key links
4. Task specific head to produces prediction for training data error are back-propagated through network.

#### **How It Works (Explained Simply)**

- Use diagrams, analogies, or examples
- Optional: insert short code snippets or illustrations

#### **Results & Implications**

- What did the experiments show?
- What’s impressive or surprising?

#### **Real-World Applicability**

- Where can this actually be used today?
- What industries or use cases are impacted?
- Are there startups or big tech companies working in this space?

#### **Challenges & Limitations**

- What’s still missing?
- Any caveats or assumptions?

#### **Your Take / Opinion**

- Do you think it’s promising?
- Is it practical today or 5 years from now?
- How does it compare with other techniques?


#### **Conclusion**

- Recap the impact in 1–2 lines
- Leave with a thought-provoking question or insight