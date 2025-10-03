---
layout: default
title: Application Overview
parent: Project Overview
nav_order: 1
---

# Application Overview
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What This Application Does

The HDM application helps researchers conduct multi-criteria decision analysis by breaking down complex decisions into manageable parts. When researchers need to evaluate alternatives against multiple criteria, they can use this application to structure the problem, collect expert judgments, and analyze the results statistically.

Here's a typical scenario: A researcher wants to evaluate which renewable energy technology is best for a particular region. The decision involves multiple criteria like cost, environmental impact, reliability, and social acceptance. Each criterion might have subcriteria, for example, cost includes initial investment, operating costs, and maintenance costs. The alternatives are the technologies being evaluated, such as solar, wind, and geothermal.

The application allows the researcher to build this hierarchy, invite domain experts to provide their judgments about the relative importance of each element, and then aggregates those judgments to produce a final ranking of the alternatives. It also performs statistical tests to determine whether the experts agree with each other, which is important for validating the results.

## The Decision Modeling Process

The Analytic Hierarchy Process, developed by Thomas Saaty in the 1970s, uses pairwise comparisons to determine weights. Instead of asking experts to directly assign importance scores, the method asks them to compare two items at a time. This approach aligns better with how humans naturally make judgments, we're better at saying "A is more important than B" than assigning absolute numerical values.

When an expert compares two criteria, they indicate which one is more important and by how much. The traditional AHP scale ranges from 1 (equal importance) to 9 (extreme importance of one over the other). In this application, I implemented a slider-based interface that uses a 1-99 scale because it's more intuitive for users, and the system converts these values to the standard AHP scale behind the scenes. The scale excludes 0 and 100 to avoid edge case issues in the conversion mathematics, with 50 representing equal importance, values below 50 favoring the first item, and values above 50 favoring the second item.

The mathematical foundation involves constructing comparison matrices and calculating eigenvectors to derive priority weights. The application uses the AHPY library to perform these calculations, which implements Saaty's eigenvalue method. This library handles the matrix operations and weight calculations according to the established AHP methodology.
