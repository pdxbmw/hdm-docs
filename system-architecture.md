---
layout: default
title: System Architecture
nav_order: 6
---

# System Architecture
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

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

## Authentication and Access Control

User authentication is handled by Clerk, a third-party authentication service. This outsourcing of authentication was a deliberate choice, implementing secure authentication is complex and error-prone, and using a specialized service reduces security risks. The application uses passwordless authentication, Clerk sends one-time codes via email rather than requiring users to create and remember passwords. This approach reduces security risks associated with password management while providing a smooth user experience. Clerk also provides OAuth integration, which means users can sign in with existing accounts from providers like Google.

The application identifies users by email address, which serves as the unique identifier. When a researcher creates a model, it's associated with their email. When experts are added to a model, they're identified by email as well. This approach works well for an academic context where email-based communication is standard.

Access control is relatively straightforward. Researchers can only see and modify their own models. Experts can only access evaluation forms for models to which they've been explicitly added. The Results page is accessible to the model owner and shows aggregated results that protect individual expert confidentiality while allowing detailed analysis.

## Deployment Architecture

The application is hosted on Reflex's hosting platform, which handles the deployment and infrastructure management. This platform-as-a-service approach simplifies deployment by managing the server infrastructure, scaling, and updates automatically.

The database is hosted on Supabase, a PostgreSQL hosting service. Supabase provides a managed database instance with automatic backups, connection pooling, and a web-based interface for database management. The application connects to Supabase using connection strings configured through environment variables.

Authentication is handled by Clerk, as mentioned earlier. Clerk's service runs independently and integrates with the application through their SDK and API. This separation of concerns means authentication infrastructure is managed externally, reducing the operational burden.

Environment variables configure the connections between these services, including the Supabase database connection string, Clerk API keys, and any other service credentials. This keeps sensitive information out of the codebase and allows for easy configuration changes without modifying the application code.
