---
layout: default
title: User Guide
nav_order: 3
---

# User Guide
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## How Users Interact with the Application

The application has different interfaces for different user roles. The researcher who creates the model has access to the Model Builder, where they define the hierarchy structure. This involves naming the decision goal, adding criteria and subcriteria, and listing the alternatives to be evaluated. The interface allows them to build this structure systematically, level by level.

Once the model structure is complete, the researcher adds experts who will evaluate the model. Each expert receives access to evaluate the specific model through a unique URL. This design choice was intentional, it takes experts directly to their evaluation form after they verify their identity through the passwordless login system. They don't need to navigate through multiple pages or search for their assigned model, the URL takes them straight to the evaluation.

## The Evaluation Process

The evaluation interface presents comparisons one at a time. The expert sees two items and uses a slider to indicate their judgment. The interface shows progress, so experts know how much of the evaluation remains. If they need to stop and return later, their progress is saved automatically. This was important because some models can be large, and completing all comparisons in one session might not be feasible.

As experts move through the evaluation, they're comparing items at different levels of the hierarchy. First, they compare criteria against each other. If there are subcriteria under a criterion, they then compare those subcriteria. Finally, they compare alternatives with respect to each leaf node in the hierarchy, either subcriteria if they exist, or criteria if there are no subcriteria.

## Visual Aids and Help

The application provides visual aids to help experts understand where they are in the hierarchy. There's a tree visualization showing the complete model structure, and help text explaining the AHP methodology. The interface also displays consistency indicators to help experts understand the quality of their judgments.
