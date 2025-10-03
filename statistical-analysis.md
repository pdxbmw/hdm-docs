---
layout: default
title: Statistical Analysis
parent: Project Overview
nav_order: 4
---

# Statistical Analysis
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

Beyond simple averaging, the application performs statistical tests to determine whether the experts' evaluations are significantly different from each other. This is done using a one-way ANOVA F-test.

## The Core Question

At its core, the F-test answers a fundamental question: when experts give different alternatives different scores, are those differences real and meaningful, or could they just be noise from experts disagreeing with each other?

> **Simple Analogy:** Imagine you asked 5 friends to rate their favorite ice cream flavors, chocolate, vanilla, and strawberry. Each friend gives each flavor a score. The question is: did chocolate actually win, or do your friends just disagree so much that the "winner" is meaningless? If all 5 friends gave chocolate scores of 9-10, vanilla scores of 5-6, and strawberry scores of 2-3, you can trust that chocolate really is the favorite. But if one friend gave chocolate a 2 and another gave it a 10, while they all agree vanilla is mediocre, the apparent winner might not mean anything.

The F-test mathematically determines which scenario you have: reliable ranking with expert agreement, or unreliable ranking with too much disagreement.

## How the F-Test Works

To understand what the F-test does, consider an example with three alternatives being evaluated by five experts. Each expert gives each alternative a score. The question we're trying to answer is: are the differences between the alternatives' average scores meaningful, or could they just be random noise from experts disagreeing with each other?

The F-test works by comparing two types of variation. The first is **between-group variation** (variation between alternatives): do the alternatives have genuinely different scores, or are they essentially the same? For instance, if Alternative A averages 0.45, Alternative B averages 0.35, and Alternative C averages 0.20, there's clear variation between the groups. The second type is **within-group variation** (variation among experts for the same alternative): do experts agree on that alternative's score, or is there disagreement? If all five experts gave Alternative A scores between 0.43 and 0.47, that's low within-group variation, the experts mostly agree. But if the scores ranged from 0.20 to 0.70, that's high within-group variation, the experts strongly disagree.

> **Key Insight:** Between-group variation is the signal (are the alternatives actually different?), and within-group variation is the noise (how much do experts disagree?). The F-test calculates a signal-to-noise ratio.

## The Statistical Measures

The test calculates these variations mathematically using several statistical measures:

**Sum of Squares** is a measure of total variation. It takes all the differences from the average, squares them (so negative and positive differences don't cancel out), and adds them up. A bigger sum means more spread in the data. For between-group variation, it measures how much each alternative's average differs from the overall average. For within-group variation, it measures how much individual expert scores differ from their alternative's average.

**Degrees of Freedom** represents how many independent pieces of information we have. For alternatives, it's the number of alternatives minus 1 (if you know the average and all but one value, you can calculate the last one). For experts, it's based on the number of alternatives and experts. This adjusts the calculations based on sample size.

**Mean Square** is simply the sum of squares divided by degrees of freedom. It's like an average amount of variation per piece of independent information, which lets us compare groups of different sizes fairly.

**F-statistic** is the final ratio of between-group mean square divided by within-group mean square. This single number tells us how much bigger the variation between alternatives is compared to the variation among experts.

## Interpreting the Results

A high F-statistic is what we want. It means the differences between alternatives are large compared to the disagreement among experts. In our example, if Alternative A consistently scores higher than B and C across all experts, the F-statistic will be high. This tells us the ranking is reliable. A low F-statistic suggests experts disagree so much that we can't confidently say the alternatives are truly different from each other.

The application also calculates the critical F-value at a 5% significance level using the F-distribution. This is the threshold for statistical significance. If the calculated F-statistic exceeds the critical value, we can say with 95% confidence that there are real differences between the alternatives. If it doesn't exceed the critical value, the apparent differences might just be due to random variation in expert judgments, and we shouldn't read too much into the ranking.

The Results page presents this information in tables. There's an F-test table showing the sum of squares, degrees of freedom, mean squares, and F-statistic for both between-group and within-group sources of variation. There's also a table showing the critical F-value and related parameters. Researchers familiar with ANOVA will recognize this format immediately, and those less familiar can simply compare the F-statistic to the critical value: if F is greater than F-critical, the results are statistically significant.

| Scenario | F-statistic | Interpretation |
|----------|-------------|----------------|
| High F (> F-critical) | Large between-group variation, small within-group variation | Reliable ranking, experts agree |
| Low F (≤ F-critical) | Small between-group variation, large within-group variation | Unreliable ranking, too much disagreement |
