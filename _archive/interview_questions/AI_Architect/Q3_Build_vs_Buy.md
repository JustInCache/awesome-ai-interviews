# AI Architect Interview Question

## Topic: Build vs. Buy Decisions

---

### Question

> Your team wants to build a custom vector database for your AI application instead of using an existing solution. How do you evaluate this decision?

---

### Answer

Build vs. buy decisions are some of the highest-leverage choices an architect makes. Building the wrong thing wastes years of effort; buying the wrong thing creates lock-in and technical debt. The framework I use focuses on strategic fit, not just technical capability.

---

#### The Core Question

"Is this component a source of competitive advantage, or is it infrastructure?"

- **Competitive advantage**: Build. Your differentiation lives here.
- **Infrastructure**: Buy. Don't reinvent solved problems.

For most companies, a vector database is infrastructure. The differentiation is what you do *with* it, not the database itself.

---

#### My Evaluation Framework

**Dimension 1: Strategic Fit**

| Question | Build Signal | Buy Signal |
|----------|--------------|------------|
| Is this core to our product? | Yes, it's our moat | No, it's enabling infrastructure |
| Do we have unique requirements? | Yes, nothing else fits | Standard use case |
| Will we invest in this long-term? | Yes, dedicated team | No, want to minimize attention |
| Does this create switching costs for customers? | Yes, lock-in advantage | No, commodity functionality |

**For vector databases specifically**:

Most companies should buy. Unless you're building a search/AI platform where vector storage *is* the product, you're better off using existing solutions and focusing engineering effort on your actual differentiators.

---

**Dimension 2: Technical Requirements**

Do existing solutions actually meet your needs?

| Requirement | Existing Solutions | Custom Build Needed? |
|-------------|-------------------|---------------------|
| Scale (vectors, QPS) | Most handle billions+ | Only at extreme scale |
| Latency | Sub-10ms available | Only for ultra-low latency |
| Filtering | Metadata filtering supported | Complex query patterns might |
| Multi-tenancy | Many support it | Custom isolation needs might |
| Hybrid search | Sparse+dense supported | Novel retrieval approaches might |

**The honest assessment**: Modern vector databases (Pinecone, Weaviate, Milvus, Qdrant, pgvector) cover 95%+ of use cases. Custom builds are rarely justified technically.

---

**Dimension 3: Total Cost of Ownership**

Build costs are always underestimated. Include:

**Build costs**:
- Initial development (typically 2-4 engineer-years for production-grade)
- Ongoing maintenance (bugs, security, updates)
- Operations (deployment, monitoring, on-call)
- Opportunity cost (what else could that team build?)
- Hiring/retention (specialists are expensive and scarce)

**Buy costs**:
- License/usage fees
- Integration effort
- Vendor risk (pricing changes, service discontinuation)
- Customization limitations

**Rule of thumb**: If build cost estimates are within 3x of buy cost, you're underestimating build. Multiply by 3 and re-evaluate.

---

**Dimension 4: Risk Assessment**

| Risk | Build | Buy |
|------|-------|-----|
| Delivery timeline | High—novel development is unpredictable | Low—proven solution |
| Quality/reliability | High initially—needs hardening | Low—battle-tested |
| Maintenance burden | High—your responsibility forever | Medium—vendor handles core |
| Vendor dependency | None | Medium-High |
| Flexibility | High | Medium—within vendor capabilities |

---

#### The Conversation with the Team

When engineers want to build, they often have legitimate concerns that should be addressed:

**"Existing solutions don't do exactly what we need"**

Response: "Let's list the specific gaps. Can we work around them? Can we influence the vendor's roadmap? Is the gap worth 2 years of building?"

**"We can build something better"**

Response: "Probably true for our specific use case. But better enough to justify the cost? What's the marginal value of 'better' here?"

**"Vendor lock-in is risky"**

Response: "Real concern. Let's design an abstraction layer so we can switch vendors. That's 10% of the work of building from scratch."

**"It'll be fun/good learning"**

Response: "I appreciate that, but we're optimizing for company outcomes. Let's find a project where learning aligns with strategic value."

---

#### My Recommendation Framework

```
Is this your core competency?
├── Yes → Investigate building
│   ├── Do you have the team? → Yes → Consider building
│   │                         → No → Can you hire? Worth it?
│   └── 
└── No → Buy
    ├── Do existing solutions meet requirements?
    │   ├── Yes → Buy, standardize on best fit
    │   └── No → Re-examine requirements (are they real?)
    │       ├── Requirements are valid → Hybrid (buy + custom extensions)
    │       └── Requirements are inflated → Buy, adapt workflow
```

**For the vector database question specifically**:

Unless you're building a database company, buy. The technology is mature, competition is fierce (good for buyers), and your engineering effort is better spent on your actual product.

---

#### Exception: When Building Makes Sense

- You're operating at scale that breaks existing solutions
- You have truly unique requirements (novel embedding types, exotic query patterns)
- This becomes a product you can sell/license
- You have excess engineering capacity with relevant expertise

Even then, start with an existing solution, hit its limits, then build.

---

### What This Question Tests

- Strategic thinking beyond technical features
- Total cost of ownership awareness
- Ability to push back on engineer preferences
- Framework-driven decision making
