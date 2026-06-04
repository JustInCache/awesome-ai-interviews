# Data Scientist Interview Question

## Topic: Data Quality & Governance

---

### Question

> Your company's data lake has grown to petabytes, but data quality issues are causing model failures and business decisions based on bad data. How do you approach data quality and governance at scale?

---

### Answer

Data quality at scale isn't about perfection—it's about **systematic identification and mitigation** of quality issues before they break downstream systems. The goal is reliable data that enables confident decision-making.

---

#### Data Quality Framework

**1. Define Quality Dimensions**

Quality isn't one thing—it's multiple dimensions:

- **Accuracy**: Data correctly represents real-world facts
- **Completeness**: All required data is present
- **Consistency**: Data is consistent across sources/systems
- **Timeliness**: Data is available when needed
- **Validity**: Data conforms to defined rules/schemas
- **Uniqueness**: No unintended duplicates

**2. Quality Assessment**

- **Automated profiling**: Statistical summaries, missing rates, distribution checks
- **Rule-based validation**: Schema validation, range checks, cross-field consistency
- **Sampling validation**: Manual review of representative samples
- **User feedback**: Reports from data consumers about issues

**3. Quality Monitoring**

- **Real-time monitoring**: Streaming validation of incoming data
- **Batch monitoring**: Periodic quality scans of historical data
- **Trend analysis**: Quality metrics over time
- **Alerting**: Automated alerts for quality degradation

---

#### Governance Structure

**1. Data Ownership**

- **Data stewards**: Domain experts responsible for data quality
- **Data owners**: Business leaders accountable for data assets
- **Data custodians**: Technical teams managing infrastructure

**2. Policies & Standards**

- **Data classification**: Public, internal, confidential, PII
- **Retention policies**: How long to keep different data types
- **Access controls**: Who can access what data
- **Usage guidelines**: Appropriate uses for different data types

**3. Quality SLAs**

- **Service level agreements**: Guaranteed data freshness, completeness, accuracy
- **Escalation procedures**: What happens when SLAs are violated
- **Remediation timelines**: How quickly issues must be fixed

---

#### Technical Implementation

**1. Data Quality Platform**

Centralized quality monitoring:

- **Great Expectations**: Declarative data quality tests
- **Deequ**: AWS data quality library
- **Custom frameworks**: Domain-specific quality checks

**2. Data Lineage**

Track data from source to consumption:

- **Lineage tracking**: Understand data dependencies and transformations
- **Impact analysis**: Which systems are affected by data changes
- **Root cause analysis**: Trace quality issues to their source

**3. Automated Remediation**

- **Data cleansing**: Automated fixes for common issues
- **Fallback mechanisms**: Use backup data sources when primary fails
- **Circuit breakers**: Stop processing when quality drops below threshold

---

#### Example: E-commerce Data Quality

**Data Sources**: User events, product catalog, transaction logs, third-party feeds

**Quality Issues Identified**:

| Issue | Detection | Impact | Mitigation |
|-------|-----------|--------|------------|
| Missing product prices | Schema validation | Broken pricing models | Default to category average |
| Duplicate user sessions | Uniqueness checks | Inflated engagement metrics | Deduplication pipeline |
| Stale inventory data | Freshness monitoring | Overselling | Real-time inventory sync |
| Inconsistent category taxonomy | Cross-source validation | Poor product recommendations | Taxonomy standardization |

**Governance Structure**:

- **Product team**: Owns product catalog quality
- **Engineering**: Manages data pipeline reliability
- **Data science**: Monitors model performance impacts
- **Business**: Defines quality requirements and SLAs

---

#### Scaling Challenges & Solutions

**Challenge: Volume**

- **Solution**: Distributed quality checks, sampling for expensive validations

**Challenge: Velocity**

- **Solution**: Streaming validation, incremental quality assessment

**Challenge: Variety**

- **Solution**: Modular quality frameworks, domain-specific validators

**Challenge: Veracity**

- **Solution**: Multi-source validation, consensus mechanisms

---

#### Organizational Change Management

**1. Culture Shift**

- **Quality mindset**: Everyone responsible for data quality
- **Training**: Data literacy programs
- **Recognition**: Reward teams that maintain high-quality data

**2. Process Integration**

- **CI/CD for data**: Data pipeline testing and validation
- **Code reviews**: Include data quality considerations
- **Incident response**: Data quality incidents treated like system outages

**3. Metrics & Incentives**

- **Quality KPIs**: Track quality metrics alongside business metrics
- **Accountability**: Clear ownership and consequences
- **Celebration**: Public recognition for quality improvements

---

#### Measuring Success

**1. Technical Metrics**

- **Data quality score**: Composite metric across dimensions
- **Pipeline reliability**: % of time pipelines deliver quality data
- **Issue resolution time**: How quickly quality issues are fixed

**2. Business Impact**

- **Decision confidence**: % of decisions made with high-quality data
- **Model performance**: Improvement in ML model accuracy/reliability
- **Cost savings**: Reduced time spent on data cleaning and debugging

**3. Cultural Indicators**

- **Self-reporting**: Teams proactively reporting and fixing issues
- **Tool adoption**: Widespread use of quality monitoring tools
- **Feedback loops**: Regular improvement based on lessons learned

---

### What This Question Tests

- Data governance and quality management
- Large-scale data operations
- Organizational change management
- Business impact of technical decisions
