# Azure DocumentDB in a Day

A hands-on, participant-first one-day workshop to help you get productive with Azure DocumentDB.

You will start with core NoSQL and DocumentDB concepts, then complete practical labs for cluster setup, migration, queries, search, RAG patterns, and basic performance troubleshooting. The day ends with an MCP demo and next steps you can apply to your own workloads.

---

## What You'll Learn

- What Azure DocumentDB is and when to use it
- Core NoSQL concepts: document model, schema flexibility, and data modeling basics
- How to deploy and connect to a DocumentDB cluster
- How to run a migration path from MongoDB-compatible source data
- How to run CRUD, filtering, and aggregation queries
- How keyword, full text, and vector search differ
- How to use explain output to spot indexing opportunities
- How MCP enables Copilot to work with live database context

---

## Prerequisites

- Azure subscription with access to create DocumentDB resources
- VS Code installed
- Access to workshop credentials/instructions from the facilitator
- Completion of prework checklist in Module 0

Prework starts here:
- [0-Prework-and-Prerequisites/README.md](0-Prework-and-Prerequisites/README.md)

---

## Full-Day Timeline

| Time | Duration | Session | Details |
|------|----------|---------|---------|
| 9:00 - 10:00 AM | 1 hour | Introduction + NoSQL Core Concepts | Azure DocumentDB overview, architecture, and core NoSQL concepts |
| 10:00 - 10:15 AM | 15 min | Break | |
| 10:15 - 12:15 PM | 2 hours | Lab 2: Cluster Setup and Migration | Provision cluster, configure access, and complete migration workflow |
| 12:15 - 1:00 PM | 45 min | Lunch | |
| 1:00 - 3:00 PM | 2 hours | Lab 3: Queries, Search, RAG, and Performance | CRUD, aggregation, text/vector search, and explain/index exercises |
| 3:00 - 3:30 PM | 30 min | Lab 4: MCP Server Demo | Instructor-led demo of Copilot + MCP with DocumentDB |
| 3:30 - 3:45 PM | 15 min | Lab 5: Wrap and Next Steps | Recap, Q&A, and next-action planning |

---

## Modules

| # | Module | What You'll Do |
|---|--------|----------------|
| 0 | [Prework and Prerequisites](0-Prework-and-Prerequisites/README.md) | Install tools, verify setup, and review MongoDB shell basics |
| 1 | [Introduction](1-Introduction) | Follow slides and discussion for DocumentDB and NoSQL fundamentals |
| 2 | [Cluster Setup and Migration](2-Cluster-Setup-and-Migration/README.md) | Create cluster, connect from tools, assess and run migration options |
| 3 | [Queries, Search, RAG, and Performance](3-Queries-Search-RAG-and-Performance/README.md) | Practice CRUD and aggregation, then run search and performance checks |
| 4 | [MCP Server Demo](4-MCP-Server-Demo/README.md) | See how MCP powers agent experiences over live database context |
| 5 | [Wrap and Next Steps](5-Wrap-and-Next-Steps/README.md) | Consolidate learning and define post-workshop action items |

---

## Participant Progress Checklist

- [ ] I completed prework and verified tools
- [ ] I can explain core DocumentDB and NoSQL concepts
- [ ] I created or connected to a working cluster
- [ ] I completed at least one migration method
- [ ] I ran CRUD, filtering, and aggregation queries
- [ ] I ran text and vector search examples
- [ ] I compared query plans before and after indexing
- [ ] I understand what MCP adds to Copilot workflows

---

## Sample Data

This workshop uses the shared movie dataset:

- [sample-data/movies_with_vectors.json](sample-data/movies_with_vectors.json)
- [sample-data/query_embeddings.json](sample-data/query_embeddings.json)
- [sample-data/README.md](sample-data/README.md)

---

## Notes for Participants

- Use the same dataset as your cohort for consistent results.
- Keep a record of command errors and outputs so support is faster.
- Do not store real production credentials in workshop files.

---

[Start with Module 0 ->](0-Prework-and-Prerequisites/README.md)
