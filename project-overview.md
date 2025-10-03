---
layout: default
title: Project Overview
nav_order: 2
has_children: true
---

# HDM Application: Project Overview

**Author:** Andi Wilson  
**Course:** Independent Study with Dr. Tugrul Daim  
**Department:** Engineering and Technology Management, Portland State University

---

## Introduction

This documentation provides a comprehensive overview of the HDM web application, explaining what it does, how it works, and the technical decisions made during development. The application is designed to support hierarchical decision modeling research using the Analytic Hierarchy Process (AHP) methodology.

## Documentation Structure

This project overview is organized into the following sections:

- **[Application Overview](application-overview)** - What the application does and the AHP methodology
- **[User Guide](user-guide)** - How researchers and experts interact with the application
- **[Technical Details](technical-details)** - AHP calculations, consistency checking, and aggregation
- **[Statistical Analysis](statistical-analysis)** - F-test explanation and interpretation
- **[System Architecture](system-architecture)** - Technical implementation, database, authentication, and deployment
- **[Development Notes](development-notes)** - Data flow, challenges faced, and future considerations

## What This Application Does

The HDM application helps researchers conduct multi-criteria decision analysis by breaking down complex decisions into manageable parts. When researchers need to evaluate alternatives against multiple criteria, they can use this application to structure the problem, collect expert judgments, and analyze the results statistically.

Here's a typical scenario: A researcher wants to evaluate which renewable energy technology is best for a particular region. The decision involves multiple criteria like cost, environmental impact, reliability, and social acceptance. Each criterion might have subcriteria, for example, cost includes initial investment, operating costs, and maintenance costs. The alternatives are the technologies being evaluated, such as solar, wind, and geothermal.

The application allows the researcher to build this hierarchy, invite domain experts to provide their judgments about the relative importance of each element, and then aggregates those judgments to produce a final ranking of the alternatives. It also performs statistical tests to determine whether the experts agree with each other, which is important for validating the results.

## The Decision Modeling Process

The Analytic Hierarchy Process, developed by Thomas Saaty in the 1970s, uses pairwise comparisons to determine weights. Instead of asking experts to directly assign importance scores, the method asks them to compare two items at a time. This approach aligns better with how humans naturally make judgments, we're better at saying "A is more important than B" than assigning absolute numerical values.

When an expert compares two criteria, they indicate which one is more important and by how much. The traditional AHP scale ranges from 1 (equal importance) to 9 (extreme importance of one over the other). In this application, I implemented a slider-based interface that uses a 1-99 scale because it's more intuitive for users, and the system converts these values to the standard AHP scale behind the scenes. The scale excludes 0 and 100 to avoid edge case issues in the conversion mathematics, with 50 representing equal importance, values below 50 favoring the first item, and values above 50 favoring the second item.

The mathematical foundation involves constructing comparison matrices and calculating eigenvectors to derive priority weights. The application uses the AHPY library to perform these calculations, which implements Saaty's eigenvalue method. This library handles the matrix operations and weight calculations according to the established AHP methodology.

## How Users Interact with the Application

The application has different interfaces for different user roles. The researcher who creates the model has access to the Model Builder, where they define the hierarchy structure. This involves naming the decision goal, adding criteria and subcriteria, and listing the alternatives to be evaluated. The interface allows them to build this structure systematically, level by level.

Once the model structure is complete, the researcher adds experts who will evaluate the model. Each expert receives access to evaluate the specific model through a unique URL. This design choice was intentional, it takes experts directly to their evaluation form after they verify their identity through the passwordless login system. They don't need to navigate through multiple pages or search for their assigned model, the URL takes them straight to the evaluation.

The evaluation interface presents comparisons one at a time. The expert sees two items and uses a slider to indicate their judgment. The interface shows progress, so experts know how much of the evaluation remains. If they need to stop and return later, their progress is saved automatically. This was important because some models can be large, and completing all comparisons in one session might not be feasible.

As experts move through the evaluation, they're comparing items at different levels of the hierarchy. First, they compare criteria against each other. If there are subcriteria under a criterion, they then compare those subcriteria. Finally, they compare alternatives with respect to each leaf node in the hierarchy, either subcriteria if they exist, or criteria if there are no subcriteria.

The application provides visual aids to help experts understand where they are in the hierarchy. There's a tree visualization showing the complete model structure, and help text explaining the AHP methodology. The interface also displays consistency indicators, which I'll explain more about shortly.

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

---

## Aggregating Multiple Expert Evaluations

When multiple experts evaluate the same model, their individual results need to be aggregated. The Results page handles this aggregation and performs several statistical analyses to help researchers understand the collective expert opinion and the degree of agreement among experts.

The primary aggregation method is calculating the arithmetic mean of the experts' final alternative scores. If three experts give an alternative scores of 0.30, 0.35, and 0.32, the aggregated score is 0.323.

The application displays each expert's individual results in a table, along with their overall consistency ratio. Researchers can click on any expert's row to see detailed breakdowns of that expert's weights at each level of the hierarchy. This transparency is important for understanding how different experts approached the evaluation and where their judgments diverged.

## Statistical Analysis

Beyond simple averaging, the application performs statistical tests to determine whether the experts' evaluations are significantly different from each other. This is done using a one-way ANOVA F-test.

### The Core Question

At its core, the F-test answers a fundamental question: when experts give different alternatives different scores, are those differences real and meaningful, or could they just be noise from experts disagreeing with each other?

> **Simple Analogy:** Imagine you asked 5 friends to rate their favorite ice cream flavors, chocolate, vanilla, and strawberry. > Each friend gives each flavor a score. The question is: did chocolate actually win, or do your friends just disagree so much that the "winner" is meaningless? If all 5 friends gave chocolate scores of 9-10, vanilla scores of 5-6, and strawberry scores of 2-3, you can trust that chocolate really is the favorite. But if one friend gave chocolate a 2 and another gave it a 10, while they all agree vanilla is mediocre, the apparent winner might not mean anything.

The F-test mathematically determines which scenario you have: reliable ranking with expert agreement, or unreliable ranking with too much disagreement.

### How the F-Test Works

To understand what the F-test does, consider an example with three alternatives being evaluated by five experts. Each expert gives each alternative a score. The question we're trying to answer is: are the differences between the alternatives' average scores meaningful, or could they just be random noise from experts disagreeing with each other?

The F-test works by comparing two types of variation. The first is **between-group variation** (variation between alternatives): do the alternatives have genuinely different scores, or are they essentially the same? For instance, if Alternative A averages 0.45, Alternative B averages 0.35, and Alternative C averages 0.20, there's clear variation between the groups. The second type is **within-group variation** (variation among experts for the same alternative): do experts agree on that alternative's score, or is there disagreement? If all five experts gave Alternative A scores between 0.43 and 0.47, that's low within-group variation, the experts mostly agree. But if the scores ranged from 0.20 to 0.70, that's high within-group variation, the experts strongly disagree.

> **Key Insight:** Between-group variation is the signal (are the alternatives actually different?), and within-group variation is the noise (how much do experts disagree?). The F-test calculates a signal-to-noise ratio.

### The Statistical Measures

The test calculates these variations mathematically using several statistical measures:

**Sum of Squares** is a measure of total variation. It takes all the differences from the average, squares them (so negative and positive differences don't cancel out), and adds them up. A bigger sum means more spread in the data. For between-group variation, it measures how much each alternative's average differs from the overall average. For within-group variation, it measures how much individual expert scores differ from their alternative's average.

**Degrees of Freedom** represents how many independent pieces of information we have. For alternatives, it's the number of alternatives minus 1 (if you know the average and all but one value, you can calculate the last one). For experts, it's based on the number of alternatives and experts. This adjusts the calculations based on sample size.

**Mean Square** is simply the sum of squares divided by degrees of freedom. It's like an average amount of variation per piece of independent information, which lets us compare groups of different sizes fairly.

**F-statistic** is the final ratio of between-group mean square divided by within-group mean square. This single number tells us how much bigger the variation between alternatives is compared to the variation among experts.

### Interpreting the Results

A high F-statistic is what we want. It means the differences between alternatives are large compared to the disagreement among experts. In our example, if Alternative A consistently scores higher than B and C across all experts, the F-statistic will be high. This tells us the ranking is reliable. A low F-statistic suggests experts disagree so much that we can't confidently say the alternatives are truly different from each other.

The application also calculates the critical F-value at a 5% significance level using the F-distribution. This is the threshold for statistical significance. If the calculated F-statistic exceeds the critical value, we can say with 95% confidence that there are real differences between the alternatives. If it doesn't exceed the critical value, the apparent differences might just be due to random variation in expert judgments, and we shouldn't read too much into the ranking.

The Results page presents this information in tables. There's an F-test table showing the sum of squares, degrees of freedom, mean squares, and F-statistic for both between-group and within-group sources of variation. There's also a table showing the critical F-value and related parameters. Researchers familiar with ANOVA will recognize this format immediately, and those less familiar can simply compare the F-statistic to the critical value: if F is greater than F-critical, the results are statistically significant.

| Scenario | F-statistic | Interpretation |
|----------|-------------|----------------|
| High F (> F-critical) | Large between-group variation, small within-group variation | Reliable ranking, experts agree |
| Low F (≤ F-critical) | Small between-group variation, large within-group variation | Unreliable ranking, too much disagreement |

---

## Technical Implementation Choices

### Framework and Architecture

The application is built using Reflex, which is a relatively new Python framework for building web applications. I chose Reflex because it allows the entire application to be written in Python, including the user interface. This was appealing for a research-oriented application where the calculations are naturally done in Python using libraries like NumPy and SciPy.

Reflex handles the complexity of frontend-backend communication through WebSockets, which enables real-time updates. When an expert completes their evaluation, for instance, the system immediately recalculates results and updates the display without requiring a page refresh. The framework uses a state-based architecture where UI components automatically re-render when the underlying state changes.

### Database Design

For data persistence, the application uses PostgreSQL with SQLModel as the ORM layer. SQLModel is a relatively new library that combines SQLAlchemy's database capabilities with Pydantic's type validation. Each model, expert, evaluation, and comparison weight is stored in the database with appropriate relationships and constraints.

The database schema uses a normalized structure for comparison weights while storing the hierarchical tree structure as JSON. This is a practical compromise, the tree structure is inherently hierarchical and changes as a unit, so JSON storage makes sense. But comparison weights are queried individually and need to be validated, so they're stored in a traditional relational table.

One technical detail worth mentioning is the use of short alphanumeric IDs instead of sequential integers or UUIDs. These 12-character IDs are URL-friendly and provide sufficient uniqueness without exposing information about how many models exist in the system. They're generated using cryptographically secure random functions to prevent guessing.

The application includes database migration support through Alembic. As the schema evolved during development, Alembic tracked the changes and can automatically update a production database to match the current schema. This was essential for iterative development where the data model needed refinement based on testing and feedback.

---

## Authentication and Access Control

User authentication is handled by Clerk, a third-party authentication service. This outsourcing of authentication was a deliberate choice, implementing secure authentication is complex and error-prone, and using a specialized service reduces security risks. The application uses passwordless authentication, Clerk sends one-time codes via email rather than requiring users to create and remember passwords. This approach reduces security risks associated with password management while providing a smooth user experience. Clerk also provides OAuth integration, which means users can sign in with existing accounts from providers like Google.

The application identifies users by email address, which serves as the unique identifier. When a researcher creates a model, it's associated with their email. When experts are added to a model, they're identified by email as well. This approach works well for an academic context where email-based communication is standard.

Access control is relatively straightforward. Researchers can only see and modify their own models. Experts can only access evaluation forms for models to which they've been explicitly added. The Results page is accessible to the model owner and shows aggregated results that protect individual expert confidentiality while allowing detailed analysis.

---

## Deployment Architecture

The application is containerized using Docker, which packages the application with all its dependencies into a single deployable unit. The container includes multiple services that need to run together: the Reflex backend server, Redis for state management, and Caddy as a reverse proxy.

Redis is used for managing real-time state synchronization. When multiple users are accessing the application simultaneously, Redis helps coordinate their sessions through pub/sub messaging. This is particularly important for WebSocket-based real-time updates.

Caddy serves as the reverse proxy, handling incoming HTTP requests and routing them appropriately. It serves the static frontend files directly and proxies API requests to the Reflex backend. This separation allows the frontend to be served efficiently while keeping the backend stateless and scalable.

The containerized application can be deployed to various platforms like Render, Railway, or cloud providers like Google Cloud Run. These platforms handle TLS termination, so the application itself doesn't need to manage SSL certificates. The database runs as a separate service, which is standard practice, you don't want your database inside your application container because data needs to persist independently.

Environment variables configure things like database connection strings and API keys. This keeps sensitive information out of the codebase and allows the same container image to be deployed in different environments (development, staging, production) with different configurations.

---

## Data Flow Through the Application

Understanding how data moves through the application helps clarify the overall architecture. When a researcher creates a model in the Model Builder, the frontend state captures the hierarchy structure as they build it. Each time they add a criterion or alternative, a state update occurs, and the UI reflects the change immediately.

When they save the model, the frontend state serializes the tree structure to JSON and sends it to the backend via a Reflex event. The backend creates a TreeModel database record with this JSON in the tree_data column. The alternatives list and associated experts are stored in their respective columns and tables.

When an expert loads their evaluation form, the application queries the TreeModel by ID, deserializes the JSON tree structure, and uses it to determine what comparisons need to be made. It also checks for any existing ComparisonWeight records for that expert and model, which allows resuming partially completed evaluations.

As the expert makes comparisons, each one is immediately saved to the database as a ComparisonWeight record. These records include the level (criteria, subcriteria, alternatives), parent node context, the two items being compared, and the weight value. The unique constraint on these fields ensures an expert can't accidentally create duplicate comparisons.

When the expert completes all required comparisons, the system marks the ExpertJudgement record as complete and triggers calculation. The backend retrieves all ComparisonWeight records for that expert and model, organizes them by level and parent, and calls the AHP calculation service.

The AHPService processes the comparisons in hierarchical order - criteria first, then subcriteria under each criterion, then alternatives under each leaf node. It constructs comparison matrices, calculates weights, checks consistency, and aggregates to final scores. The results are returned as a structured dictionary with weights at each level and overall alternative scores.

When a researcher views the Results page, the system retrieves all completed ExpertJudgement records for the model and runs calculations for each one. It then aggregates the expert results, calculates summary statistics, and performs the F-test analysis. All of this computed data is packaged into state variables that the UI components access to render the tables and displays.

---

## Challenges and Solutions

Several technical challenges came up during development that required specific solutions. One was managing the state of the AHPY Compare objects. The library maintains some internal state indexed by object name, and if multiple Compare objects with the same name exist, it can cause conflicts. The solution was to append a unique identifier to every Compare object name, ensuring no collisions even when processing multiple evaluations simultaneously.

Another challenge was handling the flexible hierarchy structure. Models can have criteria only, or criteria with subcriteria, and different criteria might have different numbers of subcriteria. This variability meant the calculation logic needed to be dynamic, determining the structure at runtime and adapting the calculations accordingly. The solution involved careful organization of comparison data by parent node, allowing the calculation to traverse the hierarchy recursively.

Database performance was a consideration, particularly for the Results page which needs to load many related records. The schema uses foreign key relationships with appropriate indexes, and the database queries are structured to minimize round trips. SQLModel's relationship loading helps by fetching related records efficiently.

User interface challenges included making the evaluation form intuitive despite the mathematical complexity underneath. The slider interface for comparisons, progress indicators, and tree visualization all contribute to making the process clearer. The help dialog provides context about AHP methodology for users who need it.

---

## Future Considerations

There are several areas where the application could be enhanced. Export functionality for results would be valuable - researchers often need to include tables and charts in publications or reports. PDF generation, Excel export, and chart image download would support this need.

Additional visualization options could make results more interpretable. Weight distribution charts, sensitivity analysis graphs showing how results change with different assumptions, and comparison heatmaps could all provide insights beyond the current tables.

The aggregation method could be made configurable. While arithmetic mean is currently used, some researchers prefer geometric mean for aggregating expert weights. Offering both options would increase flexibility.

Testing coverage could be expanded. The application currently relies on manual testing and database constraints for validation. Comprehensive unit tests for calculation logic, integration tests for database operations, and end-to-end tests for user workflows would increase confidence in the system's correctness.

Performance optimization for very large models would be beneficial. The current implementation handles typical research models (tens of criteria and alternatives) well, but models with hundreds of elements might encounter performance issues in the matrix calculations. Caching intermediate results and optimizing query patterns could address this.

---

## Conclusion

The HDM application provides researchers with a tool for conducting structured multi-criteria decision analysis. It implements the Analytic Hierarchy Process methodology, handles multiple expert evaluations, and performs statistical analysis on the results. The application is built using Python-based web technologies and can be deployed as a containerized application.

The development process involved learning new frameworks, implementing established mathematical methods, designing a database schema that balances normalization with practical needs, and creating a user interface that makes complex methodology accessible. The result is a functional application that supports Dr. Daim's research program and can serve the broader ETM research community.

This project provided hands-on experience with full-stack web development, scientific computing in Python, database design and ORM usage, containerization and deployment, and translating research methodology into software. It demonstrates how modern web technologies can be applied to academic research tools while maintaining the mathematical rigor required for scholarly work.
