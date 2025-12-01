# Fairness in Loan Approval Simulation

This project analyzes how different loan approval strategies impact **profitability** and **fairness** using a simulation tool inspired by Google’s *Attacking Discrimination in ML*. I play the role of a lead AI expert at the fictional “Bank of Magic City,” tasked with recommending a lending policy that is both profitable and aligned with fair lending and anti-discrimination principles.

The work is based on an interactive simulation (not my own code), but all analysis, calculations, and discussion in this repository are my own.

---

## Objectives

- Understand how **loan approval thresholds** affect profit and error rates.
- Compare fairness strategies:
  - **Max Profit**
  - **Group Unaware**
  - **Demographic Parity**
  - **Equal Opportunity**
- Compute and interpret **classification metrics** for different demographic groups:
  - True Positive Rate (TPR)  
  - False Positive Rate (FPR)  
  - Precision, Recall, F1  
  - Positive / Negative Predictive Value (PPV / NPV)
- Explore tensions between **profit** and **fairness**, and between **individual** and **group** fairness.
- Reflect on legal, ethical, and long-term implications of algorithmic lending decisions.

---

## Key Ideas & Findings

### 1. Thresholds, Profit, and Correctness

- As the loan threshold moves **left** (more approvals), profit eventually becomes **negative** due to many risky loans defaulting.
- As the threshold moves **right** (fewer approvals), profit initially **increases**, peaks around a mid-range threshold, and then drops again as the bank stops approving enough profitable loans.
- In the simulation:
  - The **highest profit** occurred near a threshold of **54**.
  - The **lowest profitable** threshold was around **42**.
  - The threshold that maximized **correct decisions** (classification accuracy) was different (around **48**), showing that:
    > maximizing correctness is **not** the same as maximizing profit.

Using the profit structure (successful loan: +\$300, default: –\$700), the break-even repayment rate is:

\[
300r - 700(1 - r) = 0 \ r = 0.7
\]

So the bank needs at least a **70% repayment rate** to avoid losses; the thresholds I selected in later analysis keep correctness/repayment well above this.

---

### 2. Group Differences and Fairness Metrics

The simulation considers two demographic groups (blue and orange) with different score distributions.

Under **MAX PROFIT**:

- Blue threshold: ~**61**  
- Orange threshold: ~**50**

These differences reflect how uneven score distributions (e.g., from historical or socioeconomic factors) cause a pure profit optimizer to choose different cutoffs per group.

From the confusion matrices in the simulation, I computed:

- **True Positive Rate (TPR)**
- **False Positive Rate (FPR)**
- **Precision**
- **Recall**
- **F1 score**
- **PPV / NPV** in some settings

Example pattern (summarized):

- Orange often had **higher TPR and precision** than Blue under certain settings, meaning the model was **more favorable and accurate** for the orange group.
- Even with the **same threshold** (Group Unaware), TPR and FPR still differed between groups, illustrating that **ignoring group labels does not guarantee fairness** when features are correlated with group membership.

---

### 3. Comparing Fairness Criteria

I explored four main fairness strategies provided by the simulator:

1. **Max Profit**  
   - Optimizes total profit only.  
   - Often leads to **large disparities** between groups in TPR/FPR.  
   - High legal and ethical risk if it disproportionately harms protected groups.

2. **Group Unaware**  
   - A single common threshold for both groups (e.g., 55).  
   - Intuitively “neutral,” but can **still produce disparate impact** because underlying features reflect structural inequities.  
   - In the simulation, one group had much lower approval/TPR despite the same threshold.

3. **Demographic Parity**  
   - Forces similar **positive prediction rates** across groups (same approval rate).  
   - Implementation in the sim changed thresholds so both groups had equal positive rates (e.g., 37% each).  
   - Strength: promotes equal *access* to loans.  
   - Limitations: may approve relatively riskier applicants from one group and reject safer applicants from another → misalignment with individual-level merit and potential legal tension.

4. **Equal Opportunity**  
   - Focuses on equalizing **TPR** (true positive rate) across groups.  
   - In the sim, both groups were tuned to the same TPR (e.g., 68%) with different thresholds.  
   - Strength: aligned with the idea that **qualified** individuals should have equal chances across groups.  
   - Limitations: overall approval rates can still differ, and systemic inequalities might persist long term.

**My fairness ranking (most → least fair):**

1. Equal Opportunity  
2. Demographic Parity  
3. Group Unaware  
4. Max Profit

This ranking balances:
- **Individual fairness** (treat similar individuals similarly), and  
- **Group fairness** (avoid systematically disadvantaging a protected group).

---

### 4. Custom Thresholds and Tradeoffs

I experimented with **custom thresholds** to balance profit + fairness.

Example setting:
- Blue threshold: **54**
- Orange threshold: **46**

This configuration:

- Achieved **high profit** (~\$27,700, not far from the max of ~\$32,400).
- Produced **high accuracy** (≈80–90%) and **high TPR** (≈81–89%) for both groups.
- Represented a compromise between:
  - Pure Max Profit, and
  - Strict fairness constraints (Demographic Parity / Equal Opportunity).

Still, tension remained:
- Pushing TPRs and approval rates to be **closer** often **reduced overall performance** (profit and approval for qualified people).
- Trying to satisfy both **Demographic Parity** and **Equal Opportunity** simultaneously was possible only at points with **low approval rates and low profit**, which are unattractive in practice.

---

### 5. Individual vs Group Fairness

- **Individual Fairness:**  
  Similar applicants (same risk/score) should have similar outcomes, regardless of group.
- **Group Fairness:**  
  Groups defined by protected attributes should not face systematic disadvantages (e.g., huge gaps in approval or TPR).

Using the **Custom** setting, I constructed examples where:

- Group fairness (similar positive rates) was achieved, but  
- Individual fairness was violated (TPR and error patterns noticeably different).

Then I searched for thresholds that:  
- Brought **TPR and accuracy** for both groups close together, and  
- Kept **positive rates** reasonably similar.

One such pair was:
- Blue: 64  
- Orange: 58  

Here, correctness, incorrectness, TPR, and positive rate were all relatively close across groups, but **overall approval and TPR were low**, showing that “fair” configurations can sacrifice usefulness.

---

### 6. Fairness Through Unawareness & Proxy Variables

The **Group Unaware** strategy (same threshold, no group label) illustrates “fairness through unawareness.”

Intended benefit:
- Don’t explicitly use protected attributes → avoid intentional discrimination.

However, the simulation highlights pitfalls:

- Large differences in TPR and positive rates still appear.
- Features like income, employment history, or location can act as **proxies** for race, class, or other protected traits.
- As a result, the model can still have **disparate impact**, even if protected attributes are removed from the input.

I discuss examples of possible proxies:
- Geographic region (ZIP code)
- Education level
- Income or wealth indicators

These can indirectly encode race or socioeconomic status and bias lending outcomes.

---

### 7. Long-Term / Delayed Impact

I extended the reasoning beyond one-time decisions to think about **multi-period, long-term consequences**:

- Initially “fair-looking” thresholds (balanced TPR and approval rates) can, over time:
  - Increase wealth for already advantaged groups.
  - Leave disadvantaged groups stuck with low approval rates and fewer opportunities to build credit/wealth.
- To model **delayed impact**, we could:
  - Simulate multiple loan cycles.
  - Let successful repayments improve future scores.
  - Track wealth and future approval rates by group across cycles.

This reveals that **short-term fairness metrics** can hide **long-term inequality**.

---

### 8. Explainability

Even though the simulator uses a simple **score + threshold** model, it raises explainability issues that generalize to more complex ML systems.

If explaining a decision to an applicant, I would:

- Show their position relative to the **threshold**.
- Explain that their score is based on features like credit history, income, etc.
- Relate decisions to **TPR/FPR** and bank risk, but in human language (e.g., “we approve loans in cases where similar applicants historically repay at least X% of the time”).

There’s a tension between:
- **Simple models** (easier to explain, less expressive), and  
- **Complex models** (more accurate, more opaque, more room for hidden bias via proxies).

I suggest simulation enhancements to improve explainability while keeping things simple, such as:
- Allowing “what-if” exploration (e.g., “what if my score or income were higher?”).
- Tooltips or narratives explaining how each metric (TPR, FPR, Positive Rate, etc.) connects to fairness.

---

## Tech & Methods

Although the original simulation is an external web tool, this repo focuses on:

- **Reproducing and extending the analysis**:
  - Metric calculations (TPR, FPR, Precision, Recall, F1, PPV, NPV).
  - Simple Python functions for fairness metrics.
  - (Optional) Notebook-based exploration of hypothetical confusion matrices.
- **Structured reporting**:
  - Written analysis (this README + PDF report).
  - Clear articulation of tradeoffs and long-term effects.

---

## How to Use This Repo

1. Read this `README.md` for a high-level overview.
2. Open `report/loan_fairness_simulation_report.pdf` for the full writeup.
3. (Optional) Explore the notebooks in `notebooks/` for metric calculations and visualizations.
4. Use the structure and narrative for interview prep when discussing:
   - Model risk,
   - Fairness in ML,
   - Responsible AI in financial services.

---

## Author

Created by **Tracy Huang** as part of a course project on fairness in machine learning and lending.
