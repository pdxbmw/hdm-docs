---
layout: default
title: Development Notes
nav_order: 7
---

# Development Notes
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Data Flow Through the Application

Understanding how data moves through the application helps clarify the overall architecture. When a researcher creates a model in the Model Builder, the frontend state captures the hierarchy structure as they build it. Each time they add a criterion or alternative, a state update occurs, and the UI reflects the change immediately.

When they save the model, the frontend state serializes the tree structure to JSON and sends it to the backend via a Reflex event. The backend creates a TreeModel database record with this JSON in the tree_data column. The alternatives list and associated experts are stored in their respective columns and tables.

When an expert loads their evaluation form, the application queries the TreeModel by ID, deserializes the JSON tree structure, and uses it to determine what comparisons need to be made. It also checks for any existing ComparisonWeight records for that expert and model, which allows resuming partially completed evaluations.

As the expert makes comparisons, each one is immediately saved to the database as a ComparisonWeight record. These records include the level (criteria, subcriteria, alternatives), parent node context, the two items being compared, and the weight value. The unique constraint on these fields ensures an expert can't accidentally create duplicate comparisons.

When the expert completes all required comparisons, the system marks the ExpertJudgement record as complete and triggers calculation. The backend retrieves all ComparisonWeight records for that expert and model, organizes them by level and parent, and calls the AHP calculation service.

The AHPService processes the comparisons in hierarchical order, criteria first, then subcriteria under each criterion, then alternatives under each leaf node. It constructs comparison matrices, calculates weights, checks consistency, and aggregates to final scores. The results are returned as a structured dictionary with weights at each level and overall alternative scores.

When a researcher views the Results page, the system retrieves all completed ExpertJudgement records for the model and runs calculations for each one. It then aggregates the expert results, calculates summary statistics, and performs the F-test analysis. All of this computed data is packaged into state variables that the UI components access to render the tables and displays.

## Challenges and Solutions

Several technical challenges came up during development that required specific solutions. One was managing the state of the AHPY Compare objects. The library maintains some internal state indexed by object name, and if multiple Compare objects with the same name exist, it can cause conflicts. The solution was to append a unique identifier to every Compare object name, ensuring no collisions even when processing multiple evaluations simultaneously.

Another challenge was handling the flexible hierarchy structure. Models can have criteria only, or criteria with subcriteria, and different criteria might have different numbers of subcriteria. This variability meant the calculation logic needed to be dynamic, determining the structure at runtime and adapting the calculations accordingly. The solution involved careful organization of comparison data by parent node, allowing the calculation to traverse the hierarchy recursively.

Database performance was a consideration, particularly for the Results page which needs to load many related records. The schema uses foreign key relationships with appropriate indexes, and the database queries are structured to minimize round trips. SQLModel's relationship loading helps by fetching related records efficiently.

User interface challenges included making the evaluation form intuitive despite the mathematical complexity underneath. The slider interface for comparisons, progress indicators, and tree visualization all contribute to making the process clearer. The help dialog provides context about AHP methodology for users who need it.

## Future Considerations

There are several areas where the application could be enhanced. Export functionality for results would be valuable, researchers often need to include tables and charts in publications or reports. PDF generation, Excel export, and chart image download would support this need.

Additional visualization options could make results more interpretable. Weight distribution charts, sensitivity analysis graphs showing how results change with different assumptions, and comparison heatmaps could all provide insights beyond the current tables.

The aggregation method could be made configurable. While arithmetic mean is currently used, some researchers prefer geometric mean for aggregating expert weights. Offering both options would increase flexibility.

Testing coverage could be expanded. The application currently relies on manual testing and database constraints for validation. Comprehensive unit tests for calculation logic, integration tests for database operations, and end-to-end tests for user workflows would increase confidence in the system's correctness.

Performance optimization for very large models would be beneficial. The current implementation handles typical research models (tens of criteria and alternatives) well, but models with hundreds of elements might encounter performance issues in the matrix calculations. Caching intermediate results and optimizing query patterns could address this.

## Conclusion

The HDM application provides researchers with a tool for conducting structured multi-criteria decision analysis. It implements the Analytic Hierarchy Process methodology, handles multiple expert evaluations, and performs statistical analysis on the results. The application is built using Python-based web technologies and can be deployed as a containerized application.

The development process involved learning new frameworks, implementing established mathematical methods, designing a database schema that balances normalization with practical needs, and creating a user interface that makes complex methodology accessible. The result is a functional application that supports Dr. Daim's research program and can serve the broader ETM research community.

This project provided hands-on experience with full-stack web development, scientific computing in Python, database design and ORM usage, containerization and deployment, and translating research methodology into software. It demonstrates how modern web technologies can be applied to academic research tools while maintaining the mathematical rigor required for scholarly work.
