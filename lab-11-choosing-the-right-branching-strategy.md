# Lab 11: Choosing the Right Branching Strategy

__Objective__: To practise assessing project requirements and selecting an appropriate branching strategy

## Prerequisites

- Completion of Labs 1-10 or equivalent Git knowledge
- Understanding of different branching strategies (GitHub Flow, Git Flow, Trunk-Based Development)
- Git installed on your machine
- A terminal or command prompt

## About This Lab

In Module 11, you learnt about three main branching strategies:
- **GitHub Flow** - Simple, continuous deployment
- **Git Flow** - Structured releases with multiple versions
- **Trunk-Based Development** - Frequent integration to main branch

The choice depends on factors like:
- Deployment frequency
- Team size
- Product type
- CI/CD maturity
- Risk tolerance

In this lab, you'll create assessment frameworks to help choose the right strategy for different scenarios.

## Step-by-Step Exercise

### Part 1: Assessing Deployment Patterns

1. **Create a Strategy Assessment Repository**

    Set up a repository to document your branching strategy assessments.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir strategy-selection-lab
    cd strategy-selection-lab
    git init
    ```
    </details>

2. **Evaluate Deployment Frequency**

    Create a document that assesses deployment frequency for different project types. Use `cat > filename << 'EOF'` to create the file.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > deployment-frequency.md << 'EOF'
# Deployment Frequency Assessment

## High Frequency (Multiple times per day)
**Characteristics:**
- Automated CI/CD pipeline
- Fast rollback capability
- Feature flags available

**Recommended:** Trunk-Based Development or GitHub Flow

## Medium Frequency (Weekly)
**Characteristics:**
- Scheduled releases
- Some manual testing
- Staging environment

**Recommended:** GitHub Flow or Git Flow

## Low Frequency (Monthly/Quarterly)
**Characteristics:**
- Complex release process
- Multiple supported versions
- Long testing cycles

**Recommended:** Git Flow
EOF

    git add deployment-frequency.md
    git commit -m "Add deployment frequency assessment"

    cat deployment-frequency.md
    ```
    </details>

3. **Assess Team Size and Maturity**

    Create a team assessment document. Consider Git experience, team size, and collaboration style.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > team-assessment.md << 'EOF'
# Team Assessment Guide

## Small Team (1-5 developers)
- **Git Experience:** Basic to intermediate
- **Best Fit:** GitHub Flow
- **Reason:** Simple workflow, easy to learn, minimal overhead

## Medium Team (6-20 developers)
- **Git Experience:** Intermediate
- **Best Fit:** GitHub Flow or Git Flow
- **Reason:** Depends on deployment frequency and product type

## Large Team (20+ developers)
- **Git Experience:** Advanced
- **Best Fit:** Trunk-Based Development
- **Reason:** Minimises merge conflicts, requires mature practices
EOF

    git add team-assessment.md
    git commit -m "Add team size assessment guide"
    ```
    </details>

### Part 2: Creating Decision Frameworks

4. **Build a Product Type Guide**

    Different product types suit different strategies. Create a guide mapping product types to branching strategies.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > product-type-guide.md << 'EOF'
# Product Type Branching Guide

## Web Applications (SaaS)
- **Deployment:** Continuous, centrally hosted
- **Versioning:** Rolling updates
- **Recommended:** GitHub Flow, Trunk-Based Development
- **Avoid:** Git Flow (unnecessary overhead)

## Mobile Applications
- **Deployment:** App store approval required (days/weeks)
- **Versioning:** Strict version numbers
- **Recommended:** Git Flow
- **Reason:** Structured release process matches app store constraints

## APIs and Services
- **Deployment:** Can deploy very frequently
- **Versioning:** API versioning (v1, v2)
- **Recommended:** Trunk-Based Development, GitHub Flow

## Desktop Software
- **Deployment:** User downloads and installs
- **Versioning:** Major.minor.patch
- **Recommended:** Git Flow
- **Reason:** Structured releases with support for multiple versions
EOF

    git add product-type-guide.md
    git commit -m "Add product type guide"
    ```
    </details>

5. **Create a CI/CD Maturity Assessment**

    Your CI/CD maturity level limits which strategies you can adopt. Create a maturity assessment.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > cicd-maturity.md << 'EOF'
# CI/CD Maturity Assessment

## Level 1: Manual (Low)
- No automated tests
- Manual deployment
- **Compatible with:** Git Flow
- **Not compatible with:** Trunk-Based Development

## Level 2: Basic Automation
- Some automated tests (30-50% coverage)
- CI runs on pull requests
- Semi-automated deployment
- **Compatible with:** GitHub Flow, Git Flow

## Level 3: Advanced Automation
- Comprehensive test suite (70%+ coverage)
- CI on every commit
- Automated deployment to staging
- **Compatible with:** All strategies
- **Recommended:** GitHub Flow, Trunk-Based Development

## Level 4: Continuous Deployment
- Extensive test coverage (>85%)
- Automated deployment to production
- Feature flags for control
- Automated rollback
- **Best fit:** Trunk-Based Development
EOF

    git add cicd-maturity.md
    git commit -m "Add CI/CD maturity levels"
    ```
    </details>

6. **Build a Decision Matrix**

    Create a practical decision matrix to evaluate strategies against specific criteria.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > decision-matrix.md << 'EOF'
# Branching Strategy Decision Matrix

## Scoring Guide
- 1 = Poor fit
- 2 = Moderate fit
- 3 = Good fit
- 4 = Excellent fit

## Example: Web SaaS Application

| Factor                   | Weight | GitHub Flow | Trunk-Based | Git Flow |
|--------------------------|--------|-------------|-------------|----------|
| Deployment frequency     | HIGH   | 4           | 4           | 1        |
| Team size (10 devs)      | MED    | 3           | 3           | 3        |
| CI/CD maturity           | HIGH   | 3           | 4           | 2        |
| Team Git experience      | MED    | 4           | 3           | 3        |
| Test coverage (75%)      | HIGH   | 3           | 4           | 3        |
| **Weighted Total**       |        | **3.5**     | **3.8**     | **2.2**  |

**Recommendation:** Trunk-Based Development

## Example: Mobile Application

| Factor                   | Weight | GitHub Flow | Trunk-Based | Git Flow |
|--------------------------|--------|-------------|-------------|----------|
| Deployment frequency     | HIGH   | 2           | 1           | 4        |
| Version management       | HIGH   | 2           | 1           | 4        |
| App store constraints    | HIGH   | 2           | 1           | 4        |
| **Weighted Total**       |        | **2.0**     | **1.0**     | **4.0**  |

**Recommendation:** Git Flow
EOF

    git add decision-matrix.md
    git commit -m "Add decision matrix framework"
    ```
    </details>

### Part 3: Risk and Strategy Selection

7. **Assess Risk Tolerance**

    Different organisations have different risk tolerances. Create a risk assessment guide.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > risk-assessment.md << 'EOF'
# Risk Tolerance Assessment

## High Risk Tolerance (Move Fast)
- Comfortable with rapid changes
- Quick rollback capability
- Feature flags for control
- High test coverage
- **Recommendation:** Trunk-Based Development

## Medium Risk Tolerance (Balanced)
- Code review required
- Staging environment testing
- Manual production deployment
- **Recommendation:** GitHub Flow

## Low Risk Tolerance (Cautious)
- Extensive testing required
- Multiple approval stages
- Release candidates
- Long stabilisation periods
- **Recommendation:** Git Flow

## Industry-Specific Considerations

### Financial Services / Healthcare
- Regulatory requirements
- Audit trails essential
- **Recommendation:** Git Flow (explicit processes)

### E-commerce
- High availability critical
- Rapid iteration valuable
- **Recommendation:** Trunk-Based or GitHub Flow
EOF

    git add risk-assessment.md
    git commit -m "Add risk tolerance assessment"
    ```
    </details>

8. **Create Strategy Selection Checklists**

    Quick reference checklists help make fast decisions.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > strategy-checklists.md << 'EOF'
# Branching Strategy Quick Checklists

## Choose GitHub Flow If:
- ✓ Deploy multiple times per day
- ✓ Web application or API
- ✓ Small to medium team (1-30)
- ✓ Have basic CI/CD
- ✓ Want simplicity
- ✓ Can roll back easily

## Choose Trunk-Based Development If:
- ✓ Deploy very frequently (hourly)
- ✓ Large team (30+)
- ✓ Mature CI/CD and testing
- ✓ Can use feature flags
- ✓ Test coverage >80%
- ✓ Team highly experienced with Git

## Choose Git Flow If:
- ✓ Scheduled releases (monthly, quarterly)
- ✓ Mobile or desktop application
- ✓ Multiple supported versions
- ✓ Complex QA process
- ✓ External approval required (app stores)

## Red Flags

### Don't Use GitHub Flow If:
- ✗ Can't deploy main branch at any time
- ✗ Need to support multiple versions
- ✗ Scheduled release windows only

### Don't Use Trunk-Based If:
- ✗ No feature flag system
- ✗ Low test coverage (<50%)
- ✗ Team new to Git

### Don't Use Git Flow If:
- ✗ Deploy multiple times daily
- ✗ Very small team (< 5 people)
- ✗ Web application with rolling releases
EOF

    git add strategy-checklists.md
    git commit -m "Add strategy selection checklists"

    git log --oneline
    ```
    </details>

## Challenge Exercises

### Challenge 1: Complete Project Assessment

Choose a real or hypothetical project and complete a full assessment:
- Identify deployment frequency
- Assess team maturity
- Evaluate CI/CD capability
- Create decision matrix
- Make final recommendation with rationale

<details>
<summary>Hint</summary>

Use all the templates created in the lab. Be specific about deployment frequency (e.g., "3-5 times per day", not "often"). Assign numeric scores and weights, calculate totals, and clearly state your recommendation.
</details>

### Challenge 2: Strategy Comparison Document

Create a comprehensive comparison document that shows:
- Strengths and weaknesses of each strategy
- When each strategy works best
- Common pitfalls
- Migration paths between strategies

<details>
<summary>Hint</summary>

Organise in a table format with columns for each strategy. Include real-world examples and common scenarios where teams have succeeded or struggled with each approach.
</details>

## Reflection Questions

After completing this lab, consider:

1. **Decision Factors:** Which factor is most important when choosing a branching strategy - deployment frequency, team size, or CI/CD maturity?
2. **Trade-offs:** What are the trade-offs between simplicity (GitHub Flow) and control (Git Flow)?
3. **Evolution:** How should your branching strategy evolve as your team grows?
4. **Context:** Why is there no "one size fits all" branching strategy?

## Clean Up

If you want to remove the lab directory:

```bash
cd ..
rm -rf strategy-selection-lab
```

## Key Selection Criteria

**Deployment Frequency:**
- Daily/hourly: GitHub Flow or Trunk-Based
- Weekly: GitHub Flow or Git Flow
- Monthly/quarterly: Git Flow

**Team Size:**
- 1-10: GitHub Flow
- 10-30: GitHub Flow or Git Flow
- 30+: Trunk-Based Development

**Product Type:**
- Web/SaaS: GitHub Flow or Trunk-Based
- Mobile/Desktop: Git Flow
- APIs: Trunk-Based Development

## What You've Learnt

- Assessing deployment frequency requirements
- Evaluating team size and maturity
- Measuring CI/CD capability
- Considering product type constraints
- Assessing risk tolerance
- Building decision matrices
- Using selection checklists
- Understanding the trade-offs of each strategy
- Making data-driven strategy decisions
- When to reconsider your branching strategy

The right branching strategy depends on your unique context - there's no universal answer!
