---
layout: default
title: Technical Details
parent: Project Overview
nav_order: 3
---

# Technical Details
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Understanding the Calculations

The calculations happen in several stages. When an expert submits a comparison, the application stores it in the database. Once an expert completes all required comparisons, the system marks their evaluation as complete and runs the AHP calculations for that expert's judgments.

### Comparison Matrices

For each set of comparisons at a given level, the system constructs a comparison matrix. If there are four criteria, for example, the matrix is 4x4, where each cell represents the comparison between two criteria. The diagonal is always 1 (an item compared to itself is equal), and the matrix is reciprocal, if criterion A is 3 times more important than B, then B is 1/3 as important as A.

The AHPY library calculates the principal eigenvector of this matrix, which represents the priority weights. These weights sum to 1.0, representing the relative importance of each element. This calculation is performed separately for criteria comparisons, each set of subcriteria comparisons, and each set of alternative comparisons.

### Local vs Global Weights

To get from local weights to global weights, the system multiplies each element's local weight by its parent's global weight. So if a subcriterion has a local weight of 0.4 (40% importance within its parent criterion) and the parent criterion has a global weight of 0.3 (30% importance overall), the subcriterion's global weight is 0.12 (12% importance in the overall decision).

For the final alternative scores, the system looks at the alternatives' local weights under each leaf node in the hierarchy, multiplies by that node's global weight, and sums across all nodes. This gives each alternative a final priority score representing its overall performance across all criteria.

## Consistency Checking

One of the critical features of AHP is consistency checking. Humans are not perfectly consistent, if you say A is twice as important as B, and B is twice as important as C, you should logically say A is four times as important as C. But people don't always make perfectly transitive judgments.

The consistency ratio measures how much inconsistency exists in a set of comparisons. The AHPY library calculates this by comparing the maximum eigenvalue of the comparison matrix to the matrix size. It then normalizes this against random consistency indices that Saaty derived empirically, essentially, what level of inconsistency would you expect from purely random comparisons.

The application displays these ratios in the results, allowing researchers to assess the quality of each expert's judgments. Here's how consistency ratios are typically interpreted:

| Consistency Ratio | Interpretation | Action |
|-------------------|----------------|--------|
| < 0.10 | Acceptable | No action needed |
| 0.10 - 0.20 | Marginal | Review recommended |
| > 0.20 | Problematic | Expert should revise judgments |

The application calculates consistency ratios at each level. There's a ratio for the criteria comparisons, separate ratios for each set of subcriteria comparisons, and ratios for alternative comparisons under each node. This granular approach helps identify exactly where inconsistencies occur, which is useful if an expert needs to revise their judgments.

## Aggregating Multiple Expert Evaluations

When multiple experts evaluate the same model, their individual results need to be aggregated. The Results page handles this aggregation and performs several statistical analyses to help researchers understand the collective expert opinion and the degree of agreement among experts.

The primary aggregation method is calculating the arithmetic mean of the experts' final alternative scores. If three experts give an alternative scores of 0.30, 0.35, and 0.32, the aggregated score is 0.323.

The application displays each expert's individual results in a table, along with their overall consistency ratio. Researchers can click on any expert's row to see detailed breakdowns of that expert's weights at each level of the hierarchy. This transparency is important for understanding how different experts approached the evaluation and where their judgments diverged.
