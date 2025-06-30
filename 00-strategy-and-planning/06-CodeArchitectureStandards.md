Code Architecture & Standards
This document defines the official code architecture, folder structure, and coding standards for the project. These rules must be enforced by ESLint to ensure consistency and maintainability.

I. Core Architectural Principle: Feature-Based Architecture
We will use a feature-based (or "feature-sliced") architecture. This means code is organized by what it does from a user's perspective (e.g., authentication, project-dashboard, file-upload) rather than by file type (e.g., putting all components in one folder and all hooks in another).

Primary Goal: High cohesion and low coupling. A single feature folder should contain everything it needs to function, making it easy to work on, test, or even remove without breaking other parts of the application.

II. The Three Architectural Layers
The entire application is divided into three distinct layers. Data and dependencies must only flow in one direction: App -> Features -> Shared.

app Layer (The Top Layer)

Location: src/app

Purpose: Initializes the application, sets up routing, and connects all the different parts. This layer knows about everything below it.

Rules:

Can import from features and shared.

Cannot be imported by anything else.

features Layer (The Middle Layer)

Location: src/features/[feature-name] (e.g., src/features/file-upload)

Purpose: Contains individual, self-contained business features. Each feature folder has its own components, hooks, api, and lib subfolders.

Rules:

Can import from the shared layer.

CANNOT import from the app layer.

CANNOT import from another feature's folder directly. (e.g., file-upload cannot import from project-dashboard).

shared Layer (The Bottom Layer)

Location: src/shared

Purpose: Contains code that can be used by any part of the application. This includes reusable UI components (like buttons, modals), utility functions (like date formatters), and core API clients.

Rules:

CANNOT import from app or features. It must be completely self-contained and have no knowledge of the business logic it's being used in.

III. Enforcement with eslint-plugin-boundaries
These architectural rules will be automatically enforced using the eslint-plugin-boundaries library. The configuration will be set up in the .eslintrc.js file to prevent illegal imports between layers.

Example Rule:

A rule will be set to throw an error if any file inside src/shared tries to import from src/features.

IV. Naming Conventions
Folders: kebab-case (e.g., file-upload, user-profile).

Components (React): PascalCase (e.g., FileUploadButton.tsx).

Hooks (React): camelCase with a use prefix (e.g., useUserProfile.ts).

Utility Functions: camelCase (e.g., formatDate.ts).

API Route Handlers: UPPERCASE for HTTP methods (e.g., GET.ts, POST.ts).