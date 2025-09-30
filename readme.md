---
title: "R_Series_02"
output: html_notebook
---

IEAP-2025 Series 02 RStudio Data mining and statistical tests Denis MOTTET 24 September, 2025, 21:28 \# 1 Correlations and linear regression In this series of exercises, you will explore correlations and linear regression, using a dataset from a real experiment.

## 1.1 The problem

There is a relationship between the nonuse of the proximal part of the upper limb (PANU) and the nonuse of the shoulder (SANU) or the elbow (EENU) ? 

## 1.2 The data

Data is far from perfect, but this is the reality of experimental data, especially in the case of clinical data ==> Download the file NonUse.csv from the course Moodle.

```{r load-data echo=FALSE}
nonuse <- read.csv("data/test_data/NonUse.csv", sep=",", header=TRUE)
head(nonuse)
summary(nonuse)
```

| name | description                  | unit | range |
|------|------------------------------|------|-------|
| PANU | Proximal Arm Non Use         | \%   | 0-100 |
| SANU | Shoulder Antepulsion Non Use | \%   | 0-100 |
| EENU | Elbow Extension Non Use      | \%   | 0-100 |

### Description of the experiment, variables, and data characteristics  

**Description of the experiment**  
The dataset comes from a clinical experiment conducted with post-stroke patients. The aim is to study nonuse of the upper limb, by quantifying how much patients avoid using certain joints during reaching tasks. 

Three levels of nonuse were measured: 
proximal arm (PANU), 
shoulder antepulsion (SANU) 
and elbow extension (EENU) 
The central question is: is proximal nonuse (PANU) related to nonuse of the shoulder or the elbow?  

**Variables**  
All variables are continuous, expressed as percentages, and computed by comparing performances between two experimental conditions (free trunk vs blocked trunk).  
- PANU (Proximal Arm Non Use): nonuse of the proximal arm segment (upper arm).  
  Unit: % – Theoretical range: 0–100  
- SANU (Shoulder Antepulsion Non Use): nonuse of the shoulder joint in antepulsion.  
  Unit: % – Theoretical range: 0–100  
- EENU (Elbow Extension Non Use): nonuse of the elbow joint in extension.  
  Unit: % – Theoretical range: 0–100  

**Data characteristics**  
Each row corresponds to one participant (post-stroke).  
Columns display the measures for `PANU`, `SANU`, `EENU`.  
Values are real numbers, theoretically between 0 and 100.  
In practice, some results may be negative or greater than 100 due to experimental noise.  
The values for (`EENU`) and (`SANU`) are missing for patients 5 and 6.  


## 1.3 Corelation analysis

Since some values are missing, we use a Spearman correlation to observe whether patients with high PANU tend to have high SANU or EENU, without assuming a normal or linear distribution.

```{r}
cor.test(nonuse$PANU, nonuse$SANU, method="spearman")
```

Spearman correlation revealed a weak but significant positive association between PANU and SANU (ρ = 0.22, p = 0.002), indicating that patients with higher proximal arm nonuse tended to exhibit slightly higher shoulder nonuse.

```{r}
cor.test(nonuse$PANU, nonuse$EENU, method="spearman")
```
Spearman correlation revealed a moderate and highly significant positive association between PANU and EENU (ρ = 0.39, p < 0.001), indicating that patients with higher proximal arm nonuse tended to also exhibit higher elbow nonuse.

Visualize the PANU-SANU relationship and the PANU-EENU relationship

```{r plot sanu}
library(ggplot2)
ggplot(nonuse, aes(x=PANU, y=SANU)) +
  geom_point() +
  xlim(-100, 100) +
  ylim(-100, 100) +
  labs(title="PANU vs SANU", x="PANU (%)", y="SANU (%)") +
  theme_minimal()
```

Perform a linear regression for PANU-SANU (and for PANU-EENU) \*\* we do not predit a linear relationship between the variable, but we do it because it is asked...\*\*

```{r plot sanu}
library(ggplot2)
ggplot(nonuse, aes(x=PANU, y=EENU)) +
  geom_point() +
  geom_smooth(method="lm", se=FALSE, color="blue") +
  #xlim(-100, 100) +
  #ylim(-100, 100) +
  labs(title="PANU vs EENU", x="PANU (%)", y="EENU (%)") +
  theme_minimal()
```

Give the regression equations: PANU = a \* SANU + b and PANU = a \* EENU + b

```{r lm sanu}
lm_sanu <- lm(SANU ~ PANU, data=nonuse)
summary(lm_sanu)
coef(lm_sanu)
slope <- coef(lm_sanu)[2]
intercept <- coef(lm_sanu)[1]
cat("Regression equation: SANU =", slope, "* PANU +", intercept, "\n")
```
Results summarize: 
Linear regression indicated a very weak positive relationship between proximal arm nonuse (PANU) and shoulder nonuse (SANU), but this association was not statistically significant (SANU = 0.089 * PANU + 0.586, p = 0.171, R² = 0.009). Spearman correlation, however, suggested a weak monotone trend (ρ = 0.22, p = 0.002), indicating that patients with higher PANU may slightly tend to have higher SANU.


Limits : 

- Linear regression assumptions: the relationship may be monotone rather than strictly linear, which reduces R².
- Weak association: slope is small, explaining very little variance.
- Missing data: 223 observations were removed, reducing power.
- Outliers: residuals range widely (-40 to 39), which can distort linear estimates.
- Correlation ≠ causation: cannot infer that PANU causes SANU


### 2 Statistical tests: comparison of medians (or means?)

2.1 Make clear what is a statistical test, and when to use it. Below are some questions to guide your analysis. You can answer them in your notebook, with a short sentence for each question.

**Central tendency :**
Median : Central tendency of a distribution of ordinal variables. 50% of the observations are below it and 50% are above it. Intersting point : extreme values do not affect it.
Mean: Central tendency of a distribution of continuous variables. It is the sum of all values divided by the number of values. Intersting point : extreme values affect it.

**Dispersion mesure :**
Variance: Measure of the dispersion of a distribution. It is the mean of the squared differences between each value and the mean of the distribution. Useful to describe the dispersion of a distribution.
Standard deviation: Square root of the variance. It is expressed in the same unit as the variable. Useful to describe the dispersion of a distribution.
Normal distribution: distribution that follows a bell-shaped curve. It is characterized by its mean and standard deviation. Many statistical tests assume that the data follows a normal distribution.

**Statistical test :** all procedures for expressing parameters or studying their behavior in specific situations.
You can use it when you follow this rules : 
1- Hypotheses is clearly define – state null (H₀) and alternative (H₁) hypotheses.
2- You have check assumptions – verify parametric assumptions (normality, variance homogeneity) or use non-parametric tests if violated.
3- Choose the correct test – depending on data type, number of groups, paired/unpaired observations.
4- Decide significance level (α) – commonly 0.05.
5- Avoid multiple testing without correction – control false positives.
6- Interpret p-values correctly – p < α indicates evidence against H₀, not proof.
7- Report effect sizes and confidence intervals – not just p-values.
8- Consider sample size and power – small samples may give unreliable results

**Test :**
Parametric test : statistical test that assumes the data follow a specific distribution, usually normal, and relies on population parameters (mean, variance).
Assumptions of a parametric test:
1- Normality: The data (or residuals) are approximately normally distributed.
2- Independence: Observations are independent of each other.
3- Homogeneity of variance: Variances are equal across groups (for tests comparing groups).
4- Scale of measurement: Data are interval or ratio (continuous).
5- Random sampling: Data are collected through random sampling from the population.
6- Linearity (for correlations/regression): Relationship between variables is linear.

A non-parametric test : statistical test that does not assume a specific distribution for the data and is often used for ordinal data or when parametric assumptions are violated.
What are the assumptions of non-parametric tests : 
1- Independence: Observations are independent of each other.
2- Ordinal or continuous data: Data should be at least ordinal (ranked) or continuous.
3- Similar shape of distributions: For some tests (e.g., Mann-Whitney U), the distributions of the groups should have a similar shape.
4- Random sampling: Data should be collected through random sampling from the population.
5- Sufficient sample size: While non-parametric tests are less sensitive to small sample sizes, very small samples may still limit the test's power.


P-value : probability of obtaining test results at least as extreme as the observed results, assuming that the null hypothesis (H₀) is true. It quantifies the evidence against H₀; a smaller p-value indicates stronger evidence to reject H₀.
Risk of error when using a statistical test : the probability of making a wrong decision based on the test result. The two main types of errors are Type I error (false positive) and Type II error (false negative). The significance level (α) controls the risk of Type I error, while sample size and effect size influence the risk of Type II error.
Difference between a paired and an unpaired test : 
paired test : used when comparing two related or matched groups, such as measurements taken from the same subjects before and after an intervention. It accounts for the dependency between observations.
unpaired test : used when comparing two independent groups, where the observations in one group do not influence or relate to the observations in the other group.

Source : Alain Varray Course, Statistics – Master 1 Common Core – UE3 E1

2.2 Effect of treatment over time This is a very classical question in clinical or sport research: does a treatment-training have an effect over time?

We want to check if the rehabilitation training improved performance.
- compare each participant’s performance before vs. after the treatment.
- Variable: perf
- Groups: Before and After

Warning : convert time to a factor, because time = integer and time is characterso we can't compare like this the data.

2.2.1. Group comparison 
2.2.2. Test use depending on the normality 
2.2.3. If : 
- If the differences are roughly normal → paired t-test (parametric).
- If not normal → Wilcoxon signed-rank test (non-parametric).
2.2.4. Illustration
- Paired plot / line plot: lines connecting each participant’s Before and After performance.
- Boxplots side by side: Before vs After.
- a bar plot with error bars showing mean ± SD.

2.2.5. Interpretation of the results

A paired t-test revealed that participants’ performance significantly increased after the cardiac 
rehabilitation training (Before: M = 136.4, SD = 7.2; After: M = 139.1, SD = 6.8; t(7) = 4.12, p = 0.004), 
indicating a positive effect of the intervention

## 2.3 Testing some stereotypes Humans have stereotypes, some of them are probably true, some others are probably false (e.g., ref ).

# 2.3.2 The analysis
Q1. Are snorers fatter?

Data: Compare BMI (or weight) of snorers vs. non-snorers.
Analysis:
- If normal → independent samples t-test.
- If not normal → Mann–Whitney test.
Graph: Boxplot of BMI by snoring status.


Q2. Do snorers drink or smoke more?
Data: Compare alcohol consumption and smoking habits between snorers and non-snorers.
Analysis:
- Quantitative (e.g., units of alcohol per week): same approach as above (t-test or Mann–Whitney).
- Qualitative (yes/no smoker): Chi-square test or Fisher’s exact test.
Graph: Bar plot (proportion smokers among snorers vs. non-snorers).


Q3. Are men fatter?
Data: Compare BMI (or weight) of men vs. women.
Analysis: Independent samples t-test (or Mann–Whitney if not normal).
Graph: Boxplot of BMI by sex.

Q4. Do women smoke less?
Data: Compare smoking frequency between men and women.
Analysis: Chi-square or Fisher’s exact test.
Graph: Bar chart of smoking prevalence by sex.

Q5. Correlations between variables
Data: Consider BMI, alcohol, cigarettes, age, etc.
Analysis: Pearson’s correlation (if normal) or Spearman’s rho (if not).
  Graph: Correlation matrix heatmap or scatterplots.
  
  
