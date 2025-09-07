# SRE Learning Summaries

## 📚 Key Concepts

### Site Reliability Engineering (SRE) Fundamentals
**Source:** [Google SRE Book - Introduction](https://sre.google/sre-book/introduction/)  
**Date Learned:** 2025-01-07

SRE is what happens when you ask a software engineer to design an operations team. It's a discipline that incorporates aspects of software engineering and applies them to infrastructure and operations problems.

**Key Points:**
- SRE focuses on creating scalable and highly reliable software systems
- Balances the velocity of development with the reliability of service
- Uses software engineering approaches to solve operational problems
- Emphasizes automation over manual processes
- Implements error budgets to balance innovation and stability

**Related Concepts:** DevOps, Operations, Software Engineering

---

### Service Level Objectives (SLOs) and Service Level Indicators (SLIs)
**Source:** [Google SRE Book - SLOs](https://sre.google/sre-book/service-level-objectives/)  
**Date Learned:** 2025-01-07

SLIs are metrics that measure the level of service provided. SLOs are target values or ranges for SLIs that represent the desired level of reliability.

**Key Points:**
- SLIs should be meaningful to users (availability, latency, throughput)
- SLOs should be achievable but challenging targets
- Error budgets are derived from SLOs (100% - SLO = Error Budget)
- SLOs help balance feature velocity with reliability
- Regular review and adjustment of SLOs is essential

**Related Concepts:** Error Budgets, Monitoring, Alerting

---

### The Four Golden Signals
**Source:** [Google SRE Book - Monitoring](https://sre.google/sre-book/monitoring-distributed-systems/)  
**Date Learned:** 2025-01-07

The four golden signals of monitoring are latency, traffic, errors, and saturation. These represent the most important metrics to monitor for any system.

**Key Points:**
- **Latency:** Time to process requests (distinguish between successful and failed requests)
- **Traffic:** Demand on the system (requests per second, transactions per second)
- **Errors:** Rate of failed requests (explicit failures, implicit failures, policy failures)
- **Saturation:** How "full" the service is (CPU, memory, I/O, network utilization)

**Related Concepts:** Monitoring, Alerting, Performance

---

## 🔗 Concept Relationships

```
SRE Principles → SLOs/SLIs → Error Budgets → Monitoring (Four Golden Signals)
     ↓
Incident Response → Postmortems → Continuous Improvement
```

## 🎯 Learning Objectives Progress

- [x] Understand basic SRE principles and philosophy
- [x] Learn about SLOs, SLIs, and error budgets
- [x] Identify the four golden signals of monitoring
- [ ] Deep dive into incident response procedures
- [ ] Learn about capacity planning and performance optimization
- [ ] Understand automation and tooling in SRE
- [ ] Practice implementing monitoring and alerting
- [ ] Study real-world SRE case studies

## 💡 Personal Insights

### 2025-01-07 - SRE vs Traditional Operations
The key difference I'm seeing is that SRE treats operations as a software problem. Instead of just manually managing systems, SRE teams write code to automate operations, monitor systems programmatically, and use data-driven approaches to improve reliability.

### 2025-01-07 - Error Budget Concept
The error budget concept is brilliant - it provides a quantitative way to balance the tension between reliability and feature velocity. When you have error budget remaining, you can take more risks with deployments. When you've exhausted your error budget, you focus on reliability improvements.

## 🔍 Areas for Deeper Study
- Practical implementation of SLOs in different types of systems (web services, batch jobs, data pipelines)
- Advanced monitoring techniques beyond the four golden signals
- Incident response automation and tooling
- Capacity planning methodologies and tools
- SRE organizational structures and team dynamics

## 📊 Knowledge Gaps Identified
- **Hands-on monitoring setup**: Need practical experience setting up Prometheus, Grafana, and alerting
- **Real incident response**: Haven't experienced managing a real production incident
- **Capacity planning**: Limited understanding of how to predict and plan for system capacity needs
- **SRE tooling ecosystem**: Need to explore the broader landscape of SRE tools and their use cases