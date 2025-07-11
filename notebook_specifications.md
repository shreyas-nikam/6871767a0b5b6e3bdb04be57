
# Jupyter Notebook Specification: Insurance Mitigation Impact Modeler

This document outlines the technical specification for a Jupyter Notebook designed to simulate the impact of insurance policies on operational risk, focusing on capital relief and policy effectiveness. It defines the logical flow, necessary markdown explanations, and conceptual code sections, adhering strictly to LaTeX formatting for all mathematical content.

---

## 1. Notebook Overview

This interactive Jupyter Notebook serves as a practical lab application for understanding how insurance policies mitigate operational risk. Users will simulate operational loss events, apply various insurance policy structures, and observe the transformation of gross risk into net risk, along with the assessment of capital relief.

### Learning Goals
Upon completion of this notebook, users will be able to:
- Understand the key insights contained in the provided `Operational Risk Manager Handbook` (Chapter 7: Insurance Mitigation) and supporting data.
- Learn how insurance policies can act as a hedge for operational risks.
- Explore the concepts of deductibles, covers, and their impact on transferred and retained losses.
- Understand the calculation of capital relief in the context of insurance mitigation.
- Recognize the effect of policy haircuts and qualifying criteria on capital relief.

### Expected Outcomes
The notebook will enable users to:
- **Simulate** operational loss events and analyze their distribution.
- **Define** and apply various insurance policy parameters to a simulated loss portfolio.
- **Visualize** the payout function of an insurance policy and its effect on individual loss events.
- **Quantify** the reduction in aggregate risk from gross to net due to insurance.
- **Assess** the capital relief achieved, considering various regulatory criteria and haircuts.
- **Gain intuitive understanding** of the complex relationship between insurance policy structure and operational risk transfer through interactive exploration.

---

## 2. Mathematical and Theoretical Foundations

This section details the core mathematical models and theoretical concepts underpinning the insurance mitigation analysis.

### 2.1 Operational Loss Simulation
Operational loss events ($X_i$) will be simulated to represent the gross risk exposure. While real-world losses can be complex, for this simulation, losses will be drawn from a Log-Normal distribution, a common choice for modeling severity in operational risk due to its positive skewness and non-negative values.

### 2.2 Insurance Policy Parameters and Payout Function

An insurance policy is defined by key parameters:
- **Deductible ($d$)**: The amount of loss below which the insurer pays nothing.
- **Cover per loss event ($c$)**: The maximum amount the insurer will pay for any single loss event after the deductible is met.
- **Overall Policy Limit ($L$)**: The maximum total amount the insurer will pay over the policy period for all covered losses.

The **payout function**, $L_{d,c}(X_i)$, determines the portion of a single loss event $X_i$ that is covered by insurance. This function is defined as:
$$L_{d,c}(X_i) = \min(\max(X_i - d, 0), c)$$
This formula calculates the amount paid by the insurer for an individual loss $X_i$:
1. `max(X_i - d, 0)`: Determines the loss amount exceeding the deductible $d$. If $X_i \le d$, this value is $0$.
2. `min(..., c)`: Ensures that the payout does not exceed the per-loss cover $c$.

### 2.3 Risk Aggregation and Reduction

For a series of $N$ simulated operational loss events ($X_1, X_2, \ldots, X_N$):

#### Gross Aggregate Risk
The **gross aggregate risk** ($S_{gross}$) represents the total losses before any insurance mitigation. It is the sum of all individual loss events:
$$S_{gross} = \sum_{i=1}^{N} X_i$$

#### Net Aggregate Risk
The **net aggregate risk** ($S_{net}$) represents the total losses retained by the entity after accounting for insurance payouts. This is calculated by subtracting the total transferred losses (sum of payouts for each event) from the gross aggregate risk:
$$S_{net} = S_{gross} - \sum_{i=1}^{N} L_{d,c}(X_i)$$
It is important to note that the sum of $L_{d,c}(X_i)$ for all events must also be capped by the overall policy limit $L$. The actual transferred loss due to insurance, $S_{transferred}$, is $\min(\sum_{i=1}^{N} L_{d,c}(X_i), L)$. Therefore, $S_{net} = S_{gross} - S_{transferred}$.

#### Transferred vs. Retained Losses
- **Transferred Loss**: The portion of gross loss that is covered and paid by the insurance policy. For a single event $X_i$, this is $L_{d,c}(X_i)$. The total transferred loss is $S_{transferred}$.
- **Retained Loss**: The portion of gross loss that remains with the insured entity after insurance payouts. For a single event $X_i$, this is $X_i - L_{d,c}(X_i)$. The total retained loss is $S_{net}$.

### 2.4 Capital Relief Assessment

Capital relief refers to the reduction in regulatory capital requirements due to the risk-mitigating effect of insurance. The calculation is subject to specific **qualification criteria** and **policy haircuts**, as detailed in the `Operational Risk Manager Handbook` [1].

#### Qualification Criteria (Conceptual)
Based on Table 3 from [1], insurance policies must meet certain criteria to be eligible for capital relief. These include:
- Insurer's credit rating (e.g., 'A' or higher).
- Minimum initial policy period (e.g., no less than 1 year).
- Minimum cancellation notice period (e.g., no less than 90 days).
- Policy must be mapped to operational risk exposure.
- Insurance provider must be a 3rd party (with specific rules for captive insurers).
- No exclusions or limitations triggered by supervisory actions or bankruptcy.

For illustration of capital relief, the impact of the insurer's default probability ($PD_A$) on Expected Loss ($EL$) and Unexpected Loss ($UL$) is considered. If we assume no recovery at default, and a full limit loss ($L$) with probability $PD_A$:
- **Expected Loss (EL) due to insurer default**:
    $$EL = PD_A \cdot L$$
- **Unexpected Loss (UL) due to insurer default**:
    For illustrative purposes, assuming $UL = 3\sigma$ where $\sigma$ is derived from a two-event probability space (full limit loss with probability $PD_A$, no loss with probability $1 - PD_A$):
    $$UL = 3 \cdot L \cdot \sqrt{PD_A \cdot (1 - PD_A)}$$
    The simplification $UL = 3 PD_A (1 - PD_A)L$ mentioned in the text appears to be a misinterpretation of $3\sigma$ from the source document (it implies $L$ is a variance, which it isn't). The more standard derivation for $\sigma$ from a Bernoulli trial (loss $L$ with prob $PD_A$, loss $0$ with prob $1-PD_A$) is $\sigma = \sqrt{L^2 \cdot PD_A \cdot (1-PD_A)} = L \sqrt{PD_A (1-PD_A)}$. Therefore, we will use $UL = 3 \cdot L \cdot \sqrt{PD_A \cdot (1 - PD_A)}$.

The potential capital relief is reduced by the $UL$ component to account for the risk of insurer default.

#### Policy Haircuts (Conceptual)
Based on Table 4 from [1], specific haircuts are applied to reflect imperfections in policies:
- Residual term of policy haircut (if term < 1 year).
- Policy's cancellation haircut (if cancellation term < 1 year).
- Mismatches in coverage of insurance policies.
- Uncertainty of payment.
- Overall capital relief must not exceed 20% of the gross capital.

Users will be able to toggle the application of these criteria and haircuts to observe their impact on the final capital relief.

---

## 3. Code Requirements

This section specifies the technical requirements for the implementation of the Jupyter Notebook, outlining the expected libraries, data handling, algorithms, and visualizations. No actual Python code should be included.

### 3.1 Expected Libraries

The following open-source Python libraries (from PyPI) are expected to be used:
-   **`numpy`**: For numerical operations, especially for efficient array manipulations and random number generation (e.g., `np.random.lognormal`).
-   **`pandas`**: For data structuring and manipulation, potentially storing simulated losses and derived metrics in DataFrames.
-   **`scipy.stats`**: For probability distribution functions, specifically `lognorm` for generating operational loss severities.
-   **`matplotlib.pyplot`**: For static plotting and foundational visualization capabilities.
-   **`seaborn`**: For enhanced statistical data visualization, building on Matplotlib.
-   **`ipywidgets`**: For creating interactive user controls (sliders, dropdowns, text inputs) within the notebook environment, enabling users to dynamically adjust parameters.

### 3.2 Input/Output Expectations

#### Inputs
The notebook will allow users to define the following parameters interactively using `ipywidgets`:

**A. Simulation Parameters:**
-   **Number of Loss Events (N)**: Integer slider, e.g., range 1,000 to 100,000.
-   **Log-Normal Distribution Parameters**:
    -   **Mean of Log-Loss (mu)**: Float slider, e.g., range 5.0 to 10.0 (representing $\log(\$)$).
    -   **Standard Deviation of Log-Loss (sigma)**: Float slider, e.g., range 0.5 to 2.0.

**B. Insurance Policy Parameters:**
-   **Deductible ($d$)**: Float slider/text input, e.g., range 0 to 1,000,000.
-   **Cover per Loss Event ($c$)**: Float slider/text input, e.g., range 0 to 500,000.
-   **Overall Policy Limit ($L$)**: Float slider/text input, e.g., range 0 to 10,000,000.

**C. Capital Relief Assessment Toggles:**
-   **Apply Qualification Criteria**: Boolean toggle (True/False), e.g., default True.
    -   If True, allow input for `PD_A` (Probability of Default for 'A' rated insurer), e.g., float slider 0.0001 to 0.01.
-   **Apply Policy Haircuts**: Boolean toggle (True/False), e.g., default True.
    -   If True, allow input for haircut percentages (e.g., `coverage_mismatch_haircut`, `uncertainty_payment_haircut`), e.g., float sliders 0.0 to 0.5 (0-50%).
    -   Input for `gross_capital` to check the 20% limit (e.g., float text input).

#### Outputs
The notebook will display the following results:

-   **Summary Statistics**: For simulated gross losses, transferred losses, and retained losses (e.g., mean, standard deviation, min, max, total sum).
-   **Aggregate Risk Values**: Clearly show $S_{gross}$, $S_{transferred}$, $S_{net}$.
-   **Capital Relief Calculation Steps**: Detailed breakdown of how capital relief is calculated, including the impact of $EL$ and $UL$ due to insurer default, and the effect of applied haircuts.
-   **Final Capital Relief Amount**: The quantified capital relief in monetary terms.

### 3.3 Algorithms and Functions

The notebook will implement the following core logic:

1.  **`simulate_operational_losses(N, mu, sigma)`**:
    -   Generates `N` random numbers from a Log-Normal distribution defined by `mu` and `sigma`.
    -   Returns an array/series of simulated gross loss events.

2.  **`calculate_payout_per_event(X_i, d, c)`**:
    -   Applies the payout function $L_{d,c}(X_i) = \min(\max(X_i - d, 0), c)$ for each individual loss event.
    -   Returns an array/series of individual transferred losses.

3.  **`calculate_aggregate_risks(gross_losses, payouts_per_event, policy_limit)`**:
    -   Calculates $S_{gross} = \sum X_i$.
    -   Calculates $S_{transferred} = \min(\sum L_{d,c}(X_i), L)$.
    -   Calculates $S_{net} = S_{gross} - S_{transferred}$.
    -   Returns these three aggregate values.

4.  **`calculate_capital_relief(gross_capital, transferred_losses, apply_criteria, PD_A, L, apply_haircuts, coverage_haircut, uncertainty_haircut)`**:
    -   Initializes potential capital relief based on `transferred_losses`.
    -   If `apply_criteria` is True:
        -   Calculates $EL = PD_A \cdot L$.
        -   Calculates $UL = 3 \cdot L \cdot \sqrt{PD_A \cdot (1 - PD_A)}$.
        -   Adjusts potential capital relief by subtracting $UL$.
    -   If `apply_haircuts` is True:
        -   Applies specified haircut percentages (e.g., `coverage_haircut`, `uncertainty_haircut`) multiplicatively to the relief amount.
        -   Ensures the final relief does not exceed 20% of `gross_capital`.
    -   Returns the final capital relief value.

### 3.4 Visualizations

The notebook will generate the following visualizations, adhering to the specified style and usability guidelines (color-blind-friendly palette, font size $\ge 12$ pt, clear titles, labeled axes, legends, interactivity where supported, static fallback).

1.  **Payout Function Illustration:**
    -   **Type**: Line plot.
    -   **Content**: Illustrates the $L_{d,c}(X_i)$ operator. Plot $X_i$ on the x-axis against three lines on the y-axis:
        -   Gross Loss: $X_i$ (dashed line for reference).
        -   Transferred Loss (Payout): $L_{d,c}(X_i)$.
        -   Retained Loss: $X_i - L_{d,c}(X_i)$.
    -   **Purpose**: Visually explains how deductibles and covers define the transferred and retained portions of a single loss event, similar to Figure 2 in [1].

2.  **Loss Distribution Comparison:**
    -   **Type**: Histograms and/or Kernel Density Estimates (KDE) plots, potentially overlaid or side-by-side.
    -   **Content**:
        -   Distribution of Gross Losses ($X_i$).
        -   Distribution of Transferred Losses ($L_{d,c}(X_i)$ for individual events, pre-cap).
        -   Distribution of Retained Losses ($X_i - L_{d,c}(X_i)$).
    -   **Purpose**: To visually compare the shape, spread, and magnitude of losses before and after insurance, illustrating risk transformation.

3.  **Aggregate Risk Comparison:**
    -   **Type**: Bar chart or horizontal bar chart.
    -   **Content**: Displays $S_{gross}$, $S_{transferred}$, and $S_{net}$ as distinct bars.
    -   **Purpose**: Provides a clear, direct comparison of the total financial impact of insurance.

4.  **Capital Relief Waterfall/Bar Chart:**
    -   **Type**: Bar chart or stacked bar chart.
    -   **Content**:
        -   Initial Potential Capital Relief (based on transferred losses).
        -   Deductions due to insurer default risk ($UL$).
        -   Deductions due to policy haircuts.
        -   Final Capital Relief amount.
    -   **Purpose**: To visually break down the components affecting the final capital relief, highlighting the impact of regulatory criteria and haircuts.

---

## 4. Additional Notes or Instructions

### 4.1 Assumptions and Constraints

-   **Synthetic Dataset**: The notebook will rely on a synthetic dataset generated programmatically within the notebook for operational loss events.
    -   **Content**: The synthetic data will include realistic numeric loss amounts derived from a Log-Normal distribution. No categorical or time-series fields are strictly necessary for this specific model, but the framework would support their inclusion if the scope were broadened.
    -   **Handling & Validation**: Although synthetic, the data generation process will include internal validation steps to ensure valid distributions. The generated "dataset" will be lightweight (< 5 MB) to ensure fast execution. For real datasets, validation would include confirming expected column names, data types, primary-key uniqueness (if applicable), and asserting no missing values in critical fields. Summary statistics for numeric columns will be logged.
-   **Computational Performance**: The lab is designed to execute end-to-end on a mid-spec laptop (8 GB RAM) in fewer than 5 minutes. This implies efficient numerical operations and reasonably sized simulations (e.g., $N \le 100,000$ loss events).
-   **Open-Source Libraries**: Only open-source Python libraries from PyPI may be used.
-   **Narrative and Comments**: All major steps in the notebook will include both comprehensive code comments and brief narrative markdown cells that describe **what** is happening and **why**.

### 4.2 User Interaction

-   **Interactive Parameters**: The notebook will extensively utilize `ipywidgets` to provide interactive controls (sliders, dropdowns, text inputs) for users to modify simulation and policy parameters. This allows learners to rerun analyses with different settings instantly.
-   **Inline Help**: Each interactive control will be accompanied by inline help text or tooltips to clearly describe its purpose and expected input range.

### 4.3 Customization Instructions

Users are encouraged to:
-   Adjust the simulation parameters (`N`, `mu`, `sigma`) to observe the behavior of different loss profiles.
-   Experiment with various insurance policy parameters (`d`, `c`, `L`) to understand their individual and combined effects on risk reduction.
-   Toggle the "Apply Qualification Criteria" and "Apply Policy Haircuts" options to see how regulatory considerations influence capital relief.
-   Modify the haircut percentages to explore hypothetical scenarios.

### 4.4 References

This notebook's theoretical foundations and modeling approaches are based on the following key references:

[1] Chapter 7: Insurance Mitigation, Operational Risk Manager Handbook, PRMIA. This chapter extensively covers the assessment of insurance policies as operational risk hedges, including specific criteria and modeling approaches.
[2] International convergence of capital measurement and capital standards, BIS, June 2006.
[3] Recognising the risk-mitigating impact of insurance in operational risk modeling, BIS, October 2010.
[4] Loss Models, by Stuart A. Klugman, Harry H. Panjer, Gordon E. Willmot.
