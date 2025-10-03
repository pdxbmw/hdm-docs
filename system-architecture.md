---
layout: default
title: System Architecture
parent: Project Overview
nav_order: 5
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

The application is containerized using Docker, which packages the application with all its dependencies into a single deployable unit. The container includes multiple services that need to run together: the Reflex backend server, Redis for state management, and Caddy as a reverse proxy.

Redis is used for managing real-time state synchronization. When multiple users are accessing the application simultaneously, Redis helps coordinate their sessions through pub/sub messaging. This is particularly important for WebSocket-based real-time updates.

Caddy serves as the reverse proxy, handling incoming HTTP requests and routing them appropriately. It serves the static frontend files directly and proxies API requests to the Reflex backend. This separation allows the frontend to be served efficiently while keeping the backend stateless and scalable.

The containerized application can be deployed to various platforms like Render, Railway, or cloud providers like Google Cloud Run. These platforms handle TLS termination, so the application itself doesn't need to manage SSL certificates. The database runs as a separate service, which is standard practice, you don't want your database inside your application container because data needs to persist independently.

Environment variables configure things like database connection strings and API keys. This keeps sensitive information out of the codebase and allows the same container image to be deployed in different environments (development, staging, production) with different configurations.
