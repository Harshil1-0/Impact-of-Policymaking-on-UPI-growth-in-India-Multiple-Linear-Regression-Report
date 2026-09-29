# Growth and Impact of Policy Making on Digital Payments (UPI) in India

A statistics course project asking whether India's Unified Payments Interface (UPI) grew mainly through organic adoption or through specific policy interventions. The analysis uses a 10-year monthly dataset (April 2016 – March 2026), exploratory data analysis, a multiple linear regression model with policy dummy variables, VIF-based multicollinearity diagnostics, and a non-parametric Wilcoxon rank-sum test comparing pre- and post-2020 growth.

**Course:** MA2210 Statistical Methods, Mahindra University
**Faculty:** Prof. Venkateshwararao Teega
**Date:** May 12, 2026

## Team

| Name | Roll No. |
|------|----------|
| Shrisha Pattanaik | SE24UCAM027 |
| Vasavi Agarwal | SE24UCAM062 |
| Harshil Pansala | SE24UCAM051 |
| Amaan Rehman | SE24UCAM059 |
| Pearl Mendapara | SE24UCAM043 |
| Dhruv Bohra | SE24UCAM068 |

## Research Question

To what extent is UPI's growth driven by organic adoption versus specific policy interventions (demonetisation, Zero MDR, Credit Card on UPI, tighter operational limits, the COVID-19 lockdown)?

## Data

| | |
|---|---|
| Sources | RBI & NPCI (monthly transaction volume and value); government portals and policy bulletins (exact policy dates) |
| Period | April 2016 – March 2026 |
| Granularity | Monthly |
| Observations | 120 |
| Units | Volume in millions of transactions; value in ₹ crore |
| Output file | `UPI_Policy_Impact_Dataset.csv` |

**Policy flags (binary indicators)**

- Demonetisation (November 2016)
- Zero MDR policy (January 2020)
- COVID-19 lockdown (March 2020)
- Credit Card on UPI (June 2022)
- Tighter operational limits (August 2025)

**Preprocessing**

- Dates parsed with `as.Date()` for time-series operations
- Units standardised to millions (volume) and ₹ crore (value)
- Initial (April 2016) growth rates set to 0 to keep the series continuous

**Derived variables**

$$\text{Average Ticket Size} = \frac{\text{Transaction Value}}{\text{Transaction Volume}}$$

- Month-over-month volume growth (%)
- Month-over-month value growth (%)

## Exploratory Data Analysis

- Histograms of volume and value are strongly right-skewed, consistent with exponential rather than linear growth.
- A volume-vs-time line plot shows structural breaks around demonetisation (Nov 2016) and the COVID-19 lockdown (Mar 2020), with growth accelerating markedly after 2017.
- Average ticket size does not track volume closely (the "chai-tapri effect"): UPI's scale comes from high-frequency, low-value transactions rather than larger individual transfers.
- Volume and value are almost perfectly correlated (≈ 1.0), so only one was used as the regression's dependent variable.
- A correlation heatmap flags overlap between certain policy-date variables, foreshadowing the multicollinearity found later.
- A boxplot of volume shows high-end outliers that reflect the exponential growth phase rather than data errors.
- Histograms suggest non-normal data, which motivates the non-parametric test used later.

## Multiple Linear Regression

$$Y_{\text{Vol}} = \beta_0 + \beta_1 X_{\text{ATS}} + \beta_2 D_{\text{Demo}} + \beta_3 D_{\text{ZeroMDR}} + \beta_4 D_{\text{CC}} + \beta_5 D_{\text{Limits}} + \beta_6 D_{\text{COVID}} + \varepsilon$$

```r
model <- lm(Transaction_Volume_Millions ~ Avg_Ticket_Size_INR +
              Policy_Demonetization_Nov2016 + Policy_Zero_MDR_Jan2020 +
              Policy_CreditCard_on_UPI_Jun2022 + Policy_Tighter_Limits_Aug2025 +
              Event_COVID19_Lockdown_Mar2020, data = data)
summary(model)
```

**Initial model**

| Variable | Estimate | t | p |
|----------|---------:|---:|---:|
| Intercept | 200.01 | 0.123 | 0.903 |
| Avg Ticket Size | −0.0163 | −0.154 | 0.878 |
| Demonetisation | 270.11 | 0.167 | 0.868 |
| Zero MDR | 781.23 | 0.413 | 0.680 |
| Credit Card on UPI | 9443.53 | 14.40 | < 0.001 |
| Tighter Limits | 8801.43 | 8.69 | < 0.001 |
| COVID Lockdown | 1911.42 | 1.002 | 0.319 |

Residual SE 2603; **R² = 0.872**, Adjusted R² = 0.865; F(6, ·) = 128.3, p < 0.001.

**Reading the tests**

- **t-tests:** only Credit Card on UPI and Tighter Limits are individually significant (p < 0.05); average ticket size, demonetisation, Zero MDR and the COVID lockdown are not.
- **F-test:** the model as a whole is highly significant (F = 128.34, p < 0.001).
- **R²:** the model explains 87.2% of the variance in transaction volume; the small gap to Adjusted R² (86.5%) suggests no serious overfitting.

## Multicollinearity (VIF)

```r
library(car)
vif(model)
```

| Variable | VIF |
|----------|----:|
| Avg Ticket Size | 2.33 |
| Demonetisation | 2.55 |
| Zero MDR | **14.84** |
| Credit Card on UPI | 1.80 |
| Tighter Limits | 1.13 |
| COVID Lockdown | **15.36** |

Using the common rule of thumb (VIF > 10 = severe collinearity), Zero MDR and the COVID lockdown overlap heavily, since both cluster around early 2020. **Zero MDR was dropped**, keeping COVID Lockdown as the proxy for that structural shift. R² stayed at about 0.87 after refinement, so little explanatory power was lost, while coefficient stability and the reliability of the remaining t-tests improved.

## Time-Series Smoothing

3-month and 6-month moving averages (`zoo::rollmean`) were used to separate the underlying trend from month-to-month noise. After 2020 the gap between the raw series and its moving averages widens, which the report reads as acceleration rather than a temporary spike.

## Hypothesis Test: Pre- vs Post-2020 Growth

Since the growth-rate data is non-normal, a **Wilcoxon rank-sum test** was used instead of a t-test:

- $H_0$: growth rates are identical pre- and post-2020
- $H_1$: growth rates differ significantly
- **W = 2593, p = 2.413 × 10⁻⁶**

$p \ll 0.05$, so $H_0$ is rejected: there is strong evidence of a structural shift in UPI's growth trajectory after 2020.

## Key Findings

1. **Policies that moved the needle:** Credit Card on UPI and Tighter Operational Limits are the only individually significant predictors of transaction volume.
2. **Volume, not ticket size, drives growth:** Average Ticket Size is not a significant predictor (p = 0.878); UPI's scale rests on high-frequency, low-value payments rather than larger transfers.
3. **Demonetisation as catalyst, not driver:** it triggered early adoption but has no significant independent effect once later, more structured policies are in the model.
4. **Zero MDR and COVID overlap:** their high VIFs reflect a real-world "synergy effect," since fee waivers and the push toward contactless payment happened close together in time.
5. **2020 as a turning point:** the regression identifies *which* policies mattered; the Wilcoxon test shows *when* the growth pattern itself changed.

## Broader Interpretation (as argued in the report)

- **Financial inclusion:** the pre/post-2020 shift and the significance of Credit Card on UPI are read as evidence that digital payments became a daily necessity and a bridge to formal credit.
- **Economic formalisation:** the near-1.0 volume–value correlation is read as UPI moving informal cash activity into a traceable digital ledger, useful for tax and GDP tracking.
- **Bidirectional influence:** the report's view is that growth and policy reinforce each other — demonetisation reacted to and accelerated early adoption, while later policies (Credit Card on UPI, UPI Lite) were more deliberately used to steer outcomes such as credit penetration.

These are the report's interpretive claims rather than statistical results in themselves; the regression and Wilcoxon test establish *which* variables are significant and *that* growth patterns shifted, not the causal or policy story built on top of them.

## Repository Contents

```
.
├── README.md
├── Statistical_Methods_Semester_Project__2_.pdf      # Full written report (23 pp.) — primary source for this README
├── Statistical_Methods_Semester_Project__3_.pdf       # Slide deck (Beamer PDF, 20 slides)
├── Statistical_Methods_Semester_Project.pptx          # Same slide deck, editable (.pptx)
├── Impact-of-Policymaking-on-UPI-Growth-in-India__2_.pdf   # Alternate slide-style deck/handout
└── UPI_Policy_Impact_Dataset.csv                      # Dataset referenced by the R code (not included; see below)
```

## Reproducing the Analysis

Requires R with `ggplot2`, `dplyr`, `corrplot`, `car`, and `zoo`. The report's code assumes a `UPI_Policy_Impact_Dataset.csv` with columns for date, transaction volume and value, average ticket size, MoM growth rates, and one 0/1 column per policy dummy, none of which is included in this repository. The regression and diagnostics can be reproduced with:

```r
library(ggplot2); library(dplyr); library(corrplot); library(car); library(zoo)

data <- read.csv("UPI_Policy_Impact_Dataset.csv")
data$Date <- as.Date(data$Date)

model <- lm(Transaction_Volume_Millions ~ Avg_Ticket_Size_INR +
              Policy_Demonetization_Nov2016 + Policy_Zero_MDR_Jan2020 +
              Policy_CreditCard_on_UPI_Jun2022 + Policy_Tighter_Limits_Aug2025 +
              Event_COVID19_Lockdown_Mar2020, data = data)
summary(model)
vif(model)

wilcox.test(MoM_Growth ~ Phase, data = data)
```

## Note on the Documents

- The written report (`__2_.pdf`) is the fullest and most detailed source and is what this README is based on.
- The Beamer deck (`__3_.pdf` / the `.pptx`) is a condensed, presentation version of the same analysis, dated two days later (May 14 vs May 12); its numbers match the report.
- `Impact-of-Policymaking-on-UPI-Growth-in-India__2_.pdf` appears to be an earlier or alternate slide-style rendering of the same project; it was not used as a source for the figures above, only for confirming team and topic details.
- The 2016–2023 chart shown at the start of `Impact-of-Policymaking...pdf` (surpassing 100 billion transactions in 2023) describes real historical UPI volumes, whereas the regression dataset used elsewhere in the project runs through a projected March 2026 and is not necessarily on the same scale; treat the two figures as illustrating different things.

## References

1. Alonso, C., Bhojwani, P., Han, B., Alwazir, J., Cetina, D., Jain, V., Khera, P., Ogawa, S., Sahay, R., & Shirai, S. (2023). *Stacking up the Benefits: Lessons from India's Digital Journey.* IMF Working Paper No. 2023/078.
2. National Payments Corporation of India (NPCI). (2016–2026). *UPI Product Statistics: Volume, Value, and Ecosystem Performance.* https://www.npci.org.in/what-we-do/upi/product-statistics
3. Reserve Bank of India (RBI). (2022). *Payments Vision 2025: E-payments for Everyone, Everywhere, Everytime.*
4. Ministry of Finance, Government of India. (2024). *Economic Survey 2023-24: Digital Public Infrastructure and the UPI Revolution.*
5. D'Silva, D., Filotto, U., Gualandri, E., & Giannini, C. (2025). *The Digital Leviathan: How the India Stack Redefined State Capacity.* Proceedings of the 5th International Conference on Management Research.
6. Gupta, S., & Sharma, R. (2026). *The Case for Network-level Interoperability of QR Codes in India's Digital Payments Ecosystem.* IIM Bangalore Research Series.
