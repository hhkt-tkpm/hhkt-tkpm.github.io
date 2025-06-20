---
sidebar_position: 3
---

# Source Code Organization

This document is a guide for developers on the structure of the Student Management project. It is designed to help new developers understand the source code. The project is organized into the following main directories:

## Client (Frontend)

### src/

- **app/**

  - class-management/ - Class management
  - course-management/ - Course management
  - enroll-class/ - Class enrollment
  - reference-management/ - Reference management
  - status-rules-configuration/ - Status rules configuration
  - student-management/ - Student management
  - transcript/ - Transcript
  - layout.tsx - Main application layout
  - page.tsx - Home page

- **components/** - Contains reusable UI components
- **interfaces/** - TypeScript interface definitions
- **libs/**
  - api/ - API call functions
  - hooks/ - Custom React hooks
  - services/ - Logic handling services
  - stores/ - State management with Zustand
  - utils/ - Utility functions
  - validators/ - Form validation with Zod/Yup

### Configuration files

- package.json - Dependency management
- tsconfig.json - TypeScript configuration
- next.config.ts - Next.js configuration
- eslint.config.mjs - ESLint configuration

## Server (Backend)

### src/main/java/org/example/backend/

- **common/** - Shared classes and utilities
- **config/** - Application configuration
- **controller/** - Handles HTTP requests
- **domain/** - Main entities
- **dto/**
  - data/ - Data DTOs
  - request/ - Request DTOs
  - response/ - Response DTOs
- **mapper/** - Entity and DTO mapping
- **repository/** - Database interaction
- **service/**
  - Import/ - Data import handling
  - export/ - Data export handling
  - impl/ - Service interface implementations
- **validator/** - Data validation

### src/main/resources/

- config/ - Configuration files
- db/ - Database scripts
- font/ - Fonts
- application.yml - Spring Boot configuration

### src/test/

- controller/ - Controller tests
- service/ - Service tests
- validator/ - Validator tests

### Configuration files

- pom.xml - Maven dependency management
- docker-compose.yml - Docker configuration
- application.yml - Application configuration

## Docs

Contains documentation:

- The Broken Window Theory & The Boy Scout Rule in Software Development.pdf
- Unit Test Coverage and Best Practices.pdf

## Detailed Explanation

### Client (Frontend)

The frontend is built with Next.js (React + TypeScript), using a modular architecture with separate folders for each feature. Each module in the `app` directory represents a main function of the system.

### Server (Backend)

The backend is built with Spring Boot, following a clear layered architecture:

- Controller: Handles requests/responses
- Service: Contains business logic
- Repository: Database interaction
- DTO: Defines data transfer structures
- Domain: Defines entities

### Testing

Each layer has its own tests to ensure code quality:

- Unit tests for services and validators
- Integration tests for controllers
- E2E tests for main flows

