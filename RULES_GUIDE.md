# JunieGuides Rules Reference

## Introduction

This document serves as the primary reference topic rule for the JunieGuides repository. It facilitates the search for guides relating to projects when this repository is included in an external application. These rules and guides must be referenced by other projects using this set of rules and understood by AI before generating any code.

## Purpose

The JunieGuides repository contains a comprehensive collection of rules, guidelines, and best practices for various technologies, frameworks, and architectural patterns. These guides exist to:

1. Ensure consistency across projects
2. Promote best practices and patterns
3. Provide clear guidance for both developers and AI assistants
4. Enable AI to effectively update and produce code that complies with or intentionally deviates from default implementations

## Repository Structure

The repository is organized into the following main sections:

### 1. Generative Guides

Located in the `generative/` directory, these guides provide detailed instructions for specific technologies and architectural patterns:

#### 1.1 Angular

**Purpose**: Provides guidelines for developing modern, component-based web applications using the Angular framework.

**When to Apply**: 
- When building single-page applications (SPAs)
- When developing complex, interactive web interfaces
- When working with TypeScript-based frontend projects
- When implementing component-based architecture for web applications

**Key Concepts**:
- Component architecture
- Angular modules
- Dependency injection
- Reactive programming with RxJS
- Angular routing and navigation

**Location**: `generative/angular/`

##### Angular 20

**Purpose**: Provides guidelines for developing web applications using Angular 20, the latest major version of the Angular framework released in May 2025.

**When to Apply**:
- When building modern single-page applications with the latest Angular features
- When implementing reactive state management using Angular's built-in solution
- When developing applications that require zero-flicker hydration
- When creating micro-frontend architectures with Angular
- When integrating with Web Components

**Key Concepts**:
- Reactive State Management Framework
- Hydration Improvements
- Micro-Frontend Architecture Support
- Enhanced Developer Experience
- Streamlined Dependency Injection
- Web Components Integration

**Location**: `generative/angular/angular20.md`

#### 1.2 Domain-Driven Design (DDD)

**Purpose**: Offers architectural patterns and implementation strategies for tackling complex domains by connecting the implementation to an evolving model.

**When to Apply**:
- When working with complex business domains with many rules
- When the project is expected to evolve over time
- When there's access to domain experts
- When the technical team values collaboration and learning

**Key Concepts**:
- Bounded contexts
- Entities and value objects
- Aggregates and repositories
- Domain events
- Ubiquitous language

**Location**: `generative/ddd/domaindrivendesign.md`

##### Microfrontends in DDD

**Purpose**: Provides architectural insights into microfrontends within the context of Domain-Driven Design, combining frontend modularity with domain-driven principles.

**When to Apply**:
- When implementing frontend architectures that align with business domains
- When scaling frontend development across multiple teams
- When seeking to apply DDD principles to frontend architecture
- When creating autonomous, domain-focused frontend components
- When implementing a cohesive user experience across multiple bounded contexts

**Key Concepts**:
- Independent teams with end-to-end ownership
- Technology agnosticism for frontend teams
- Bounded contexts in frontend architecture
- Ubiquitous language in UI components
- Strategic design patterns for frontend composition
- Domain-aligned decomposition of UI

**Location**: `generative/ddd/microfronts.md`

#### 1.3 Entity Assist

**Purpose**: Provides a reactive ORM framework for GuicedEE applications using Vert.x and Hibernate Reactive.

**When to Apply**:
- When building reactive applications with GuicedEE
- When working with databases in a non-blocking manner
- **NOT** When implementing the repository pattern
- When integrating Hibernate Reactive with Vert.x

**Key Concepts**:
- Reactive database operations
- Query building
- Entity relationships
- Transaction management
- Integration with GuicedInjection

**Location**: `generative/entity-assist/entity-assist-reactive-rules.md`

#### 1.4 GuicedEE

**Purpose**: Offers framework-specific rules and services for the GuicedEE ecosystem, which provides dependency injection and modular development.

**When to Apply**:
- When building applications using the GuicedEE framework
- When implementing dependency injection with Google Guice
- When working with modular Java applications
- When integrating various services within the GuicedEE ecosystem

**Key Concepts**:
- Dependency injection
- Service Provider Interface (SPI)
- Module configuration
- Lifecycle management
- Integration with other technologies

**Location**: `generative/guicedee/`

##### GuicedEE Function Documentation

The following GuicedEE function documentation files serve as the default rules to follow when this repository is present in another project module:

###### Core Injection Framework

**GuicedInjection** (`generative/guicedee/functions/guiced-injection-rules.md`):
- **Purpose**: Provides comprehensive guidelines for using the GuicedInjection framework, the core dependency injection system of GuicedEE.
- **When to Apply**: When implementing dependency injection, service discovery, or lifecycle management in GuicedEE applications.
- **Key Features**: SPI implementation, module configuration, lifecycle management, job management.

**GuicedInjection Framework Guidelines** (`generative/guicedee/functions/guiced-injection-rules.md`):
- **Purpose**: Provides detailed guidelines for using the GuicedInjection framework, including package structure, SPI implementation, module configuration, and best practices.
- **When to Apply**: When developing modular applications with GuicedEE, implementing service discovery, managing application lifecycle, or integrating with Vert.x.
- **Key Features**: CRTP (Curiously Recurring Template Pattern), SPI interfaces and implementations, module configuration with JPMS, lifecycle hooks, integration with Vert.x, testing with Guice.

###### Vert.x Integration

**GuicedVertx** (`generative/guicedee/functions/guiced-vertx-rules.md`):
- **Purpose**: Outlines the recommended package structure and usage guidelines for the GuicedVertx module.
- **When to Apply**: When integrating Vert.x with GuicedEE applications for reactive, event-driven functionality.
- **Key Features**: Event bus integration, verticle configuration, Vert.x 5 migration.

**GuicedVertxPersistence** (`generative/guicedee/functions/guiced-vertx-persistence-rules.md`):
- **Purpose**: Provides guidelines for database connectivity and ORM capabilities using Vert.x and Hibernate Reactive.
- **When to Apply**: When implementing database access in GuicedEE applications with reactive patterns.
- **Key Features**: Database module creation, connection configuration, reactive mode, transaction management.

**GuicedVertxRest** (`generative/guicedee/functions/guiced-vertx-rest-rules.md`):
- **Purpose**: Defines patterns for building RESTful APIs with Vert.x in GuicedEE applications.
- **When to Apply**: When creating REST endpoints and API services in GuicedEE applications.
- **Key Features**: REST endpoint definition, request/response handling, authentication integration.

**GuicedVertxSockets** (`generative/guicedee/functions/guiced-vertx-sockets-rules.md`):
- **Purpose**: Provides guidelines for implementing WebSocket functionality with Vert.x.
- **When to Apply**: When adding real-time communication capabilities to GuicedEE applications.
- **Key Features**: WebSocket handlers, socket event processing, client-server communication.

**GuicedVertxWeb** (`generative/guicedee/functions/guiced-vertx-web-rules.md`):
- **Purpose**: Outlines patterns for web application development with Vert.x Web.
- **When to Apply**: When building web applications or web interfaces with GuicedEE.
- **Key Features**: Route configuration, template rendering, static resource handling, form processing.

###### Integration Technologies

**GuicedHibernate** (`generative/guicedee/functions/guiced-hibernate-rules.md`):
- **Purpose**: Provides guidelines for integrating Hibernate ORM with GuicedEE.
- **When to Apply**: When implementing traditional (non-reactive) database access in GuicedEE applications.
- **Key Features**: Entity configuration, session management, transaction handling, query optimization.

**GuicedRabbit** (`generative/guicedee/functions/guiced-rabbit-rules.md`):
- **Purpose**: Outlines patterns for integrating RabbitMQ messaging with GuicedEE.
- **When to Apply**: When implementing message queuing and asynchronous communication in GuicedEE applications.
- **Key Features**: Queue configuration, message publishing, message consumption, error handling.

**GuicedCerial** (`generative/guicedee/functions/guiced-cerial-rules.md`):
- **Purpose**: Provides guidelines for serial port communication in GuicedEE applications.
- **When to Apply**: When interfacing with hardware devices via serial ports in GuicedEE applications.
- **Key Features**: Port configuration, data transmission, event handling, error recovery.

**GuicedSwaggerOpenAPI** (`generative/guicedee/functions/guiced-swagger-openapi-rules.md`):
- **Purpose**: Defines patterns for API documentation using Swagger/OpenAPI in GuicedEE applications.
- **When to Apply**: When documenting REST APIs built with GuicedEE.
- **Key Features**: API documentation, schema definition, endpoint documentation, UI configuration.

When implementing GuicedEE applications, these function documentation files should be considered the default rules to follow. They provide comprehensive guidance on how to structure, configure, and implement various aspects of GuicedEE applications.

###### Representations

**Representations in GuicedEE** (`generative/guicedee/services/representations.md`):
- **Purpose**: Provides guidelines for implementing domain-driven interfaces that produce object representations of specified types, following the Curiously Recurring Template Pattern (CRTP).
- **When to Apply**: When converting between domain objects and various representation formats (JSON, XML, Excel) while maintaining type safety and domain-driven design principles.
- **Key Features**: Domain-driven interfaces, CRTP for type-safe fluent interfaces, JSON/XML/Excel representation, custom serializers and deserializers, consistent conversion interfaces.

#### 1.5 Hibernate

**Purpose**: Provides ORM and database access patterns for working with relational databases in Java applications.

**When to Apply**:
- When working with relational databases in Java applications
- When implementing the repository pattern
- When mapping object models to relational database schemas
- When managing database transactions

**Key Concepts**:
- Entity mapping
- Relationship management
- Query strategies
- Transaction handling
- Caching mechanisms

**Location**: `generative/hibernate/`

##### Hibernate 7 Reactive

**Purpose**: Provides guidelines for using Hibernate Reactive (version 7.x) with Vert.x 5 applications, focusing on non-blocking database operations.

**When to Apply**:
- When building reactive, non-blocking applications that require database access
- When using Vert.x 5 with Hibernate
- When implementing JPMS modular applications with Hibernate
- When migrating from traditional blocking Hibernate to reactive patterns
- When working with Mutiny API for reactive programming

**Key Concepts**:
- Non-blocking database operations
- Mutiny API integration (Uni<T>, Multi<T>)
- Explicit transaction management
- SessionFactory initialization
- CRUD patterns in reactive context
- Testing with Testcontainers
- Threading and Event Loop considerations

**Location**: `generative/hibernate/hibernate-7-reactive.md`, `generative/hibernate/hibernate-7-upgrade.md`

#### 1.6 Lombok

**Purpose**: Offers guidelines for code generation and reduction of boilerplate in Java projects using Project Lombok.

**When to Apply**:
- When seeking to reduce boilerplate code in Java projects
- When working with data/model classes that require standard methods
- When integrating with other annotation processors like MapStruct or Hibernate
- When implementing consistent logging patterns

**Key Concepts**:
- Annotation-based code generation
- Integration with other annotation processors
- Best practices for entity classes
- Configuration options
- Common pitfalls and solutions

**Location**: `generative/lombok/`

#### 1.7 MapStruct

**Purpose**: Provides object mapping strategies for converting between different object models in Java applications.

**When to Apply**:
- When mapping between different object models (e.g., DTOs to entities)
- When implementing clean separation between layers
- When working with complex object transformations
- When integrating with other annotation processors like Lombok

**Key Concepts**:
- Type-safe mapping
- Custom mapping methods
- Collection mapping
- Inheritance mapping
- Integration with dependency injection

**Location**: `generative/mapstruct/`

##### MapStruct 6

**Purpose**: Provides guidelines for using MapStruct 6, a code generator that simplifies the implementation of mappings between Java bean types based on a convention over configuration approach.

**When to Apply**:
- When implementing object mapping between different layers of an application
- When working with complex domain models that require transformation
- When integrating with GuicedEE applications
- When seeking to reduce boilerplate code for object transformations
- When implementing type-safe mapping with proper JPMS module support

**Key Concepts**:
- Maven and Gradle coordinates (original and GuicedEE)
- JPMS module information
- Property mapping with different names
- Multiple source parameters
- Type conversions
- Custom mapping methods
- Collection mapping
- Nested bean mappings
- Update methods
- Integration with CDI and Spring

**Location**: `generative/mapstruct/mapstruct-6.md`

#### 1.8 Vert.x

**Purpose**: Offers reactive programming guidelines for building scalable, event-driven applications using the Vert.x toolkit.

**When to Apply**:
- When building reactive, non-blocking applications
- When implementing event-driven architectures
- When working with high-concurrency systems
- When developing microservices

**Key Concepts**:
- Event loop model
- Verticles
- Event bus
- Reactive programming
- Non-blocking I/O

**Location**: `generative/vertx/`

##### Vert.x 5 OAuth2 Authentication

**Purpose**: Provides comprehensive guidelines for implementing OAuth 2.0 authentication and authorization in Vert.x 5 applications.

**When to Apply**:
- When implementing secure authentication in Vert.x 5 applications
- When replacing full Keycloak dependency with lightweight OAuth2 integration
- When supporting various OAuth2 flows (Authorization Code, Client Credentials, etc.)
- When implementing OpenID Connect Discovery
- When managing tokens and authorization with roles

**Key Concepts**:
- OAuth2 flows (Authorization Code, Password Credentials, Client Credentials)
- JWT/On-Behalf-Of Flow
- OpenID Connect Discovery
- Token management (refreshing, revoking)
- Authorization with roles
- JWT validation
- Integration with identity providers

**Location**: `generative/vertx/vertx-5-oauth2-flow-guide.md`

##### Vert.x 5 Transaction Handling

**Purpose**: Provides guidelines for handling transactions in Vert.x applications, with support for both blocking and reactive JDBC-style database clients.

**When to Apply**:
- When implementing database transactions in Vert.x applications
- When working with both blocking and reactive database clients
- When migrating from threaded transactional systems to event-based flows
- When implementing proper error handling and resource cleanup in database operations
- When using event bus consumers with transactional operations

**Key Concepts**:
- Explicit transaction management
- Blocking transactions (JDBC)
- Reactive transactions (Vert.x SQL Client)
- Transaction utility classes
- Error handling and cleanup
- Event bus integration
- Thread context management
- Anti-patterns to avoid

**Location**: `generative/vertx/vertx-5-transaction-handling.md`

#### 1.9 Web Components

**Purpose**: Provides guidelines for component-based UI development using web components standards.

**When to Apply**:
- When building reusable UI components
- When implementing framework-agnostic components
- When developing micro-frontends
- When creating design systems

**Key Concepts**:
- Custom elements
- Shadow DOM
- HTML templates
- Component lifecycle
- Integration with frameworks

**Location**: `generative/webcomponents/`

##### Angular 20 Web Components

**Purpose**: Provides comprehensive information on creating components that can function as both Angular components and Web Components (Custom Elements) using Angular 20.

**When to Apply**:
- When building reusable UI components that need to work across different frameworks
- When creating framework-agnostic components with Angular 20
- When leveraging Angular's features while producing standard web components
- When implementing a design system that needs to be consumed by non-Angular applications
- When migrating from Angular-specific components to more portable solutions

**Key Concepts**:
- Angular Elements
- Custom Elements API
- Shadow DOM encapsulation
- Component interoperability
- Property and event binding across frameworks
- Standalone components

**Location**: `generative/webcomponents/angular20-webcomponents.md`

##### Angular 20 Web Components in Micro Frontend Architecture

**Purpose**: Provides comprehensive information on integrating Angular 20, Web Components, and Micro Frontend Architecture to create modular, maintainable, and scalable applications.

**When to Apply**:
- When implementing micro frontend architecture with Angular 20
- When creating domain-driven frontend decomposition
- When building systems that require team autonomy and independent deployment
- When developing framework-agnostic component libraries
- When implementing cross-team UI integration strategies

**Key Concepts**:
- Shell application architecture
- Web Components as integration points
- Domain-driven decomposition
- Module Federation
- Cross-framework communication
- Shared design systems
- Independent deployment pipelines

**Location**: `generative/webcomponents/angular20-webcomponents-microfronts.md`

## How to Use These Rules

This section provides guidance on how to effectively apply the rules in this repository to your development projects.

### For Developers

**Purpose**: Provide developers with a clear process for applying these rules in their daily work.

**When to Apply**:
- When starting a new project
- When joining an existing project that follows these rules
- When refactoring code to align with best practices
- When reviewing code for compliance with standards

**Key Steps**:

1. **Reference the Appropriate Guide**: 
   - Identify the technology or pattern you're working with
   - Locate the corresponding guide in this repository
   - Review the guide thoroughly before implementation

2. **Follow the Guidelines**: 
   - Implement the recommended patterns and practices
   - Use the provided examples as templates
   - Adhere to the specified naming conventions and structure

3. **Understand the Context**: 
   - Each guide provides context for when to apply specific patterns
   - Consider the specific requirements of your project
   - Adapt the guidelines to your specific use case while maintaining the core principles

4. **Deviate When Necessary**: 
   - When business requirements demand deviation from the rules, document the reasons
   - Create explicit documentation explaining why the deviation was necessary
   - Consider creating a proposal to update the rules if the deviation represents a common use case

**Examples**:
- A developer working on a new Angular component would reference the Angular guide for component structure
- When implementing a database access layer, the developer would follow the Entity Assist or Hibernate guidelines
- If a project requires a different approach than what's documented, the developer would document the rationale

### For AI Assistants

**Purpose**: Provide AI assistants with a framework for understanding and applying these rules when generating code.

**When to Apply**:
- When generating code for projects that reference this repository
- When providing recommendations for code structure or patterns
- When reviewing code for compliance with these standards
- When suggesting refactoring approaches

**Key Steps**:

1. **Pre-Generation Understanding**: 
   - Before generating any code, AI must understand the relevant rules from this repository
   - Analyze the project structure to identify which rules apply
   - Consider the specific context of the request

2. **Rule Application**: 
   - Apply the rules from the appropriate guides when generating code
   - Follow the specified patterns, naming conventions, and structure
   - Include appropriate comments referencing the relevant guidelines

3. **Context Awareness**: 
   - Consider the specific context of the project and adapt the rules accordingly
   - Balance adherence to rules with practical considerations
   - Recognize when different rules might conflict and resolve appropriately

4. **Explicit Deviation**: 
   - When generating code that intentionally deviates from the rules, explicitly state the deviation and the rationale
   - Provide alternative implementations that would comply with the rules
   - Explain the trade-offs between compliant and non-compliant approaches

5. **Rule Prioritization**: 
   - When multiple rules apply, prioritize them based on:
     - Project-specific requirements
     - Technology-specific best practices
     - General software engineering principles
   - Document the prioritization decisions in comments

**Examples**:
- An AI assistant generating a new service class would follow the GuicedInjection guidelines for SPI implementation
- When suggesting a refactoring approach, the AI would reference specific sections of the relevant guide
- If generating code that deviates from the rules, the AI would include comments explaining why and how it deviates

## Rule Categories

The rules in this repository fall into several categories:

### 1. Structural Rules

**Purpose**: Define how code should be organized to ensure consistency, maintainability, and scalability across projects.

**When to Apply**:
- When starting a new project
- When refactoring existing code
- When establishing coding standards for a team
- When integrating multiple modules or components

**Key Concepts**:
- Package naming conventions
- File organization
- Module structure
- Component relationships
- Layering principles
- Separation of concerns

**Examples**:
- Using consistent package naming like `com.company.project.feature.component`
- Organizing files by feature rather than by type
- Defining clear module boundaries with explicit dependencies
- Establishing consistent component interaction patterns

### 2. Implementation Rules

**Purpose**: Specify how specific patterns should be implemented to ensure correct, efficient, and maintainable code.

**When to Apply**:
- When implementing business logic
- When applying design patterns
- When solving common programming problems
- When implementing framework-specific functionality

**Key Concepts**:
- Design patterns
- Architectural patterns
- Framework-specific patterns
- Error handling approaches
- Concurrency models
- Resource management

**Examples**:
- Using the Builder pattern for complex object creation
- Implementing the Repository pattern for data access
- Following framework-specific conventions for dependency injection
- Using structured exception handling with appropriate granularity

### 3. Integration Rules

**Purpose**: Guide how different technologies should interact to create cohesive, loosely-coupled systems.

**When to Apply**:
- When connecting different services or components
- When designing APIs
- When implementing communication between systems
- When integrating with third-party services

**Key Concepts**:
- Service communication
- API design
- Data exchange formats
- Authentication and authorization
- Contract testing
- Integration patterns
- Error handling across boundaries

**Examples**:
- Using REST for synchronous service communication
- Implementing message queues for asynchronous processing
- Defining clear API contracts with OpenAPI/Swagger
- Implementing token-based authentication for service-to-service communication

### 4. Quality Rules

**Purpose**: Ensure code quality through testing, documentation, performance optimization, and security practices.

**When to Apply**:
- Throughout the development lifecycle
- When establishing quality gates
- When preparing for production deployment
- When maintaining existing systems

**Key Concepts**:
- Testing approaches
- Documentation requirements
- Performance considerations
- Security best practices
- Code review standards
- Monitoring and observability
- Technical debt management

**Examples**:
- Implementing unit tests with appropriate coverage
- Documenting public APIs and complex algorithms
- Optimizing database queries for performance
- Following OWASP guidelines for security
- Establishing code review checklists
- Implementing logging and monitoring

## Updating Code to Comply with Rules

**Purpose**: Provide a structured approach for bringing existing codebases into compliance with these rules while minimizing risk and disruption.

**When to Apply**:
- When integrating a legacy codebase with a project that follows these rules
- When adopting these rules for an existing project
- When updating code after rules have been revised or extended
- When performing technical debt reduction

**Key Process**:

1. **Identify Applicable Rules**: 
   - Determine which rules apply to the codebase
   - Create a checklist of relevant rules from each applicable guide
   - Prioritize rules based on their importance and impact

2. **Assess Compliance**: 
   - Evaluate how well the current code follows the rules
   - Use automated tools where possible to identify non-compliant areas
   - Create a compliance report highlighting areas that need attention
   - Estimate the effort required for each area of non-compliance

3. **Prioritize Changes**: 
   - Focus on high-impact, low-risk changes first
   - Prioritize changes that improve maintainability and reduce technical debt
   - Consider dependencies between changes
   - Create a roadmap for implementing changes

4. **Incremental Adoption**: 
   - Apply changes incrementally rather than all at once
   - Use feature branches or similar techniques to isolate changes
   - Establish clear milestones for compliance
   - Integrate changes regularly to avoid divergence

5. **Test Thoroughly**: 
   - Ensure changes don't introduce regressions
   - Implement or expand automated tests to verify behavior
   - Perform targeted testing of modified components
   - Consider implementing parallel runs of old and new implementations

6. **Document Deviations**: 
   - When full compliance isn't possible, document the reasons
   - Create explicit documentation for each intentional deviation
   - Include rationale, risks, and potential future compliance path
   - Establish a review process for deviations

**Examples**:
- A team migrating from a custom ORM to Entity Assist would identify all data access code, assess current patterns, prioritize entity classes, incrementally convert each entity, thoroughly test each conversion, and document any cases where the standard pattern couldn't be applied
- When updating a codebase to follow the GuicedInjection patterns, a team might first identify all service classes, then assess their current structure, prioritize core services, incrementally refactor each service, test each refactored service, and document any services that couldn't follow the standard pattern

## Rule Precedence

**Purpose**: Establish a clear hierarchy for resolving conflicts between different rules to ensure consistent decision-making.

**When to Apply**:
- When different rules provide contradictory guidance
- When project-specific constraints conflict with general best practices
- When integrating multiple technologies with their own rule sets
- When balancing competing concerns (e.g., performance vs. maintainability)

**Key Principles**:

When rules conflict, follow this precedence order:

1. **Project-specific requirements and constraints**:
   - Business requirements that necessitate specific technical approaches
   - Regulatory or compliance requirements
   - Performance or scalability constraints
   - Integration requirements with existing systems
   - Team composition and expertise

2. **Technology-specific rules from the appropriate guide**:
   - Framework-specific patterns and practices
   - Language-specific idioms and conventions
   - Database-specific optimization techniques
   - Platform-specific implementation details

3. **General architectural principles**:
   - Separation of concerns
   - Single responsibility principle
   - Don't repeat yourself (DRY)
   - Interface segregation
   - Dependency inversion
   - Loose coupling and high cohesion

4. **Common software engineering best practices**:
   - Code readability and maintainability
   - Consistent naming conventions
   - Appropriate commenting and documentation
   - Error handling and logging
   - Testing and testability

**Resolution Process**:

1. Identify the specific rules that are in conflict
2. Determine which level of precedence each rule belongs to
3. Apply the rule from the highest precedence level
4. Document the decision and rationale
5. Consider whether a new rule or exception should be proposed

**Examples**:
- If a project requires integration with a legacy system that uses a specific data format, this project-specific requirement would take precedence over general data format recommendations
- When implementing a feature in Angular, the Angular-specific guidelines would take precedence over general JavaScript best practices when they conflict
- If performance requirements necessitate breaking a general architectural principle like separation of concerns, the performance requirement (as a project-specific constraint) would take precedence, but the deviation should be documented

## Extending the Rules

**Purpose**: Provide a framework for evolving and expanding these rules to address new technologies, patterns, and challenges while maintaining consistency and quality.

**When to Apply**:
- When introducing new technologies to the organization
- When identifying patterns that aren't covered by existing rules
- When existing rules need refinement based on practical experience
- When addressing recurring issues or anti-patterns
- When incorporating industry best practices

**Key Process**:

1. **Follow the existing format and structure**:
   - Use the established section headings and organization
   - Maintain consistent terminology and naming conventions
   - Adhere to the same level of detail and specificity
   - Use similar examples and explanations

2. **Provide clear rationales for new rules**:
   - Explain the problem the rule is solving
   - Describe the benefits of following the rule
   - Reference industry standards or research where applicable
   - Address potential objections or alternative approaches

3. **Include examples of both compliant and non-compliant code**:
   - Show concrete examples of code that follows the rule
   - Provide counter-examples of code that violates the rule
   - Explain the specific issues with non-compliant code
   - Demonstrate the process of transforming non-compliant code to compliant code

4. **Consider backward compatibility**:
   - Assess the impact on existing codebases
   - Identify potential conflicts with existing rules
   - Determine whether existing code needs to be updated
   - Consider phased adoption for significant changes

5. **Document any migration paths for existing code**:
   - Provide step-by-step guidance for updating existing code
   - Include tools or scripts that can assist with migration
   - Specify any interim states or temporary exceptions
   - Define timelines and milestones for adoption

**Submission Process**:

1. Draft the new rule or modification following the guidelines above
2. Submit for peer review by experienced team members
3. Conduct a pilot implementation on a small project
4. Refine based on feedback and practical experience
5. Finalize and integrate into the rules repository
6. Communicate changes to all stakeholders

**Examples**:
- When adding rules for a new framework like Vue.js, follow the same structure as the Angular guide, provide clear rationales for Vue-specific patterns, include examples of proper component structure, consider compatibility with existing web component rules, and document how to migrate from other frameworks
- When refining database access rules based on performance learnings, maintain the existing structure, explain the performance benefits with metrics, show before/after query examples, consider impact on existing database code, and provide a clear migration path with performance testing guidelines

## Conclusion

The JunieGuides repository serves as a comprehensive reference for development rules and guidelines. By following these rules, both human developers and AI assistants can create consistent, high-quality code that adheres to best practices while maintaining the flexibility to adapt to specific project requirements.

When in doubt, refer to the specific guide for the technology or pattern you're working with, and remember that these rules exist to facilitate development, not to constrain creativity or problem-solving.
