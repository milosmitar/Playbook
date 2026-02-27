# iOS Development Playbook (SwiftUI)

## Summary

- [Guidelines](#guidelines)
- [Core Technical Pillars](#core-technical-pillars)
- [Git Workflow](#git-workflow)
- [Development Tools & Technologies](#development-tools-technologies)
- [App Architecture](#app-architecture)
- [Navigation](#navigation)
- [Networking](#networking)
- [Persistent Storage](#persistent-storage)
- [Firebase Integration](#firebase-integration)
- [Testing](#testing)
- [Task Management](#task-management)
- [Developer Onboarding Guide](#developer-onboarding-guide)
    - [App Navigation](#app-navigation)
    - [Creating a New Screen](#creating-new-screen)
    - [Adding an API Endpoint](#adding-api-endpoint)
- [End-to-End Data Flow Explained](#end-to-end-data-flow-explained)
    - [Data Flow Lifecycle](#data-flow-lifecycle)
    - [MockWebService Integration](#mock-web-service-integration)
    

## Guidelines

This document defines internal iOS development standards, architecture
principles, and technical practices used across our SwiftUI projects.

Our goal is to build scalable, maintainable, and testable applications
using modern Apple-native technologies.

------------------------------------------------------------------------

## Core Technical Pillars

### Language

Swift is the primary programming language.

### UI Framework

SwiftUI is used exclusively for UI development.

### Architecture

We follow MVVM with a DataStore pattern.

### Minimum iOS Version

 Current Minimum: iOS 15

 Recommended Minimum: **iOS 16 or higher**

For modern iOS application development, we recommend setting the minimum supported OS version to **iOS 16+**.

This version provides compatibility with approximately **92–95% of active iOS devices worldwide**, ensuring broad market reach while significantly reducing development complexity and long-term maintenance costs.

Supporting older versions such as iOS 15 offers only marginal additional user coverage (typically around **3–5%** of devices), but introduces substantial technical overhead. This includes the need for legacy API support, increased testing scope, additional UI workarounds, and higher risk of platform-specific bugs.

Targeting iOS 16+ enables full use of modern SwiftUI and system capabilities, which results in:

* Faster development and feature delivery
* Cleaner and more maintainable codebase
* Improved performance and UI stability
* Reduced QA and testing effort
* Better long-term scalability of the product

While the minimum version can be adjusted depending on specific project goals or target demographics, supporting significantly older iOS versions is generally not recommended for contemporary SwiftUI-based applications due to increased cost and limited user benefit.


------------------------------------------------------------------------

## Git Workflow

We use a simplified GitFlow model.

Branches: - main → production - dev → development - feature/\* → new
work

Versioning follows Semantic Versioning.

------------------------------------------------------------------------

## Development Tools & Technologies

-   Xcode
-   SwiftUI
-   Async/Await
-   URLSession
-   CoreData
-   Firebase SDK

------------------------------------------------------------------------

## App Architecture

### Layers

MVVM with DataStore Layer

The application follows a Model–View–ViewModel (MVVM) architecture enhanced with a dedicated DataStore layer responsible for handling business logic and data access.

In this structure:

  - Views (SwiftUI) are responsible only for UI rendering.

  - ViewModels manage presentation logic and UI state.

  - DataStores handle networking, data processing, and communication with backend services.

This separation provides:

  - clear responsibility boundaries

  - improved testability

  - better maintainability

  - easier scalability as the application grows

The DataStore layer acts as a centralized entry point for all data operations, ensuring consistent handling of API calls across the application.

------------------------------------------------------------------------

## Navigation

We use a custom Router architecture based on NavigationState and Route
enum.

Navigation is driven by NavigationStack.

------------------------------------------------------------------------

## Networking

Architecture includes:

-   NetworkServiceFactory
-   WebRepository
-   HTTPClient
-   MockWebService

Custom Networking Layer

 The project uses a fully custom networking solution built on top of URLSession and Swift’s Async/Await concurrency model.

This approach was chosen to:

  - maintain full control over request handling

  - support advanced features like certificate pinning

  - provide consistent error mapping

  - avoid unnecessary third-party dependencies

All network requests are executed through a centralized HTTPClient, which is responsible for:

  - request configuration

  - response validation

  - error mapping

  - JSON decoding

This ensures a standardized and predictable networking workflow.

Network Service Factory

A NetworkServiceFactory is used to dynamically provide the correct network implementation depending on the environment.

This enables:

  - seamless switching between real and mock services

  - simplified testing and debugging

  - environment-based configuration

It plays a key role in maintaining flexibility and supporting automated testing.


Concurrency Model
Async/Await

The project uses Swift’s modern Async/Await concurrency model for handling asynchronous operations.

This approach offers:

  - cleaner and more readable asynchronous code

  - reduced callback nesting

  - better error propagation using try/await

  - improved thread safety

Async/Await integrates naturally with SwiftUI’s .task modifier, allowing network calls to be triggered directly within views in a safe and structured way.

------------------------------------------------------------------------

## Persistent Storage

CoreData → structured data\
UserDefaults → settings\
Keychain → secure storage

CoreData

CoreData is used for local data storage and offline persistence.

It provides:

  - efficient object graph management

  - local caching capabilities

  - improved performance through managed contexts

  - built-in data relationships and migration support

This allows the application to maintain local state even when network connectivity is limited.

UserDefaults & Keychain

UserDefaults is used for storing lightweight, non-sensitive configuration data such as app settings and flags.

Keychain is used for securely storing sensitive information, including:

  - authentication tokens

  - user credentials

Using Keychain ensures compliance with Apple security best practices and protects critical user data.

------------------------------------------------------------------------

## Firebase Integration

-   Analytics
-   Crashlytics
-   Push Notifications (Firebase Messaging)

Analytics, Crashlytics, and Notifications

Firebase services are integrated to support monitoring and user engagement.

These include:

  - Firebase Analytics for tracking user behavior

  - Crashlytics for real-time crash reporting

  - Firebase Cloud Messaging for push notifications

This setup provides valuable insights into application performance and helps ensure rapid issue detection.

------------------------------------------------------------------------

## Testing

Unit Tests: - ViewModel logic

UI Tests: - Navigation & screens


The project includes:

-   Unit tests for ViewModels

-   UI tests for critical user flows

-   Mock services for controlled test environments

This testing strategy ensures:

-   business logic correctness

-   UI stability

-   regression prevention

-   reliable feature development

MockWebService plays a central role in enabling deterministic and repeatable tests.

MockWebService used for: - Debug - UI previews - Testing

------------------------------------------------------------------------

## Task Management

Tasks must be: - Actionable - Contextual - Testable

Lifecycle: In Progress → Review → Test → Done

------------------------------------------------------------------------

## Developer Onboarding Guide

## App Navigation

### **Navigation Architecture Overview**

The application uses a **state-driven navigation system** built on top of SwiftUI’s `NavigationStack`. Navigation is centrally managed through a custom **NavigationState** object that acts as a single source of truth for routing.

Instead of relying on scattered navigation logic inside views, all navigation changes are controlled through a shared observable state object injected into the environment.

---

### **NavigationState**

`NavigationState` is an `ObservableObject` responsible for managing the navigation stack using a strongly typed list of routes.

It maintains:

* a `routes` array representing the current navigation stack
* a `selectedHomeTab` property for tab navigation state

Navigation actions are performed by mutating the routes array:

* **push(route)** — adds a new screen to the navigation stack
* **popTo(route)** — navigates back to a specific screen
* **popToRoot()** — returns to the root screen

This design mirrors traditional navigation stacks while remaining fully compatible with SwiftUI’s declarative model.

---

### **Route Enum**

Navigation destinations are defined using a strongly typed `Route` enum.

Each case represents a screen and may include associated data required for that screen, such as:

* identifiers
* models
* bindings

This approach provides:

* compile-time safety
* clear navigation contracts between screens
* reduced risk of runtime routing errors
* explicit data passing between features

---

### **Environment-Based Navigation Injection**

The `NavigationState` object is created at the application root and injected using `@EnvironmentObject`.

This allows any view in the hierarchy to trigger navigation without needing direct references to parent views.

**Benefits include:**

* centralized navigation logic
* simplified view communication
* reduced coupling between screens
* easier testing and debugging

---

### **NavigationStack Integration**

SwiftUI’s `.navigationDestination(for:)` modifier is used to map `Route` values to concrete screen views.

When a new route is pushed into the navigation state, SwiftUI automatically updates the UI to present the corresponding destination.

This ensures that navigation remains fully synchronized with application state.

---

### **Why This Navigation Approach**

This custom routing system was chosen to provide:

* predictable and testable navigation flow
* centralized routing management
* type-safe screen transitions
* clear separation between UI and navigation logic
* scalability for large applications

It also aligns with the unidirectional data flow architecture used throughout the project.

---

### **Navigation Principles**

* Views must not perform direct navigation logic.
* Navigation must always be triggered through `NavigationState`.
* Routes must be strongly typed.
* Associated data should be passed via `Route` cases rather than global state.


------------------------------------------------------------------------

## Creating a New Screen

### Goal

To implement a new feature screen following the project’s architectural and UI standards.

---

### Workflow

#### 1. Create the View

The development process always starts from the UI layer.

* Implement the SwiftUI View based on the provided design.
* Focus only on layout and visual structure.
* Do not include business logic at this stage.

The View should be created together with a **Preview** using mock data to allow rapid UI iteration.

---

#### 2. Add Navigation Support

If the screen is navigable:

* Add a new case in the `Route` enum.
* Connect the screen through `NavigationState`.
* Ensure required parameters are passed via associated values.

---

#### 3. Prepare Environment Dependencies

Each screen typically injects required environment objects such as:

* NavigationState (for routing)
* DataStore objects (acting as domain ViewModels)
* NetworkResponseEnvironment (for global error handling)

These objects allow the View to remain lightweight while delegating logic to appropriate layers.

---

### Principles

* Views must contain UI logic only.
* No networking calls should exist inside Views.
* All screens must include SwiftUI previews.

---

------------------------------------------------------------------------

## Adding an API Endpoint

### Goal

To integrate backend communication following the standardized networking architecture.

---

### Workflow

#### 1. Create Response Models

Define Codable models representing API responses.

These models serve as the data contract between the backend and the application.

---

#### 2. Implement DataStore Method

All API communication must be handled through a DataStore.

Each DataStore represents a logical domain, such as:

* UserDataStore (authentication, profile, registration)
* HomeDataStore (home screen data)
* ProductDataStore (product-related operations)

The DataStore exposes async functions that internally call the networking layer.

---

#### 3. Call Network Service

DataStores communicate with the networking layer via the NetworkService protocol implemented by WebRepository.

This layer:

* builds request resources
* executes requests through HTTPClient
* decodes responses
* maps errors

---

#### 4. Handle Errors via NetworkResponseEnvironment

Errors are not handled directly inside Views.

Instead, all network errors are processed through a shared NetworkResponseEnvironment which:

* interprets server error models
* maps URL errors
* triggers global UI alerts

This ensures consistent error handling across the application.

---

#### 5. Trigger API Call from View

The View initiates API calls using SwiftUI `.task` or user actions.

Typical flow:

View → DataStore → NetworkService → HTTPClient → API → Response → UI

---

### Networking Responsibilities

The HTTPClient is responsible for:

* constructing URL requests
* attaching authentication tokens
* handling automatic token refresh
* validating HTTP responses
* decoding JSON data

This centralized approach ensures secure, consistent, and reusable networking logic.

---

### Principles

* All network calls must go through DataStores.
* Direct URLSession usage is not allowed outside the networking layer.
* Errors must always be handled via NetworkResponseEnvironment.
* API responses must be mapped into strongly typed models.

---

### Final Step

Once data is successfully retrieved:

* The DataStore updates state.
* The View reacts automatically through SwiftUI’s reactive binding system.
* UI is updated without manual refresh logic.


------------------------------------------------------------------------


## End-to-End Data Flow Explained

### Overview

The application follows a **unidirectional, state-driven data flow**.
This ensures predictable UI updates, clear separation of responsibilities, and easier debugging.

All user interactions and data updates move through well-defined architectural layers.

---

## Data Flow Lifecycle

### 1. User Interaction

The flow begins when a user performs an action in the UI, such as:

* opening a screen
* pulling to refresh
* tapping a button
* submitting a form

The SwiftUI View captures this interaction and triggers a corresponding action.

---

### 2. View → DataStore Communication

The View does **not** contain business logic.

Instead, it calls a function on an injected **DataStore**, which acts as a domain-level ViewModel.

Example responsibilities of the DataStore:

* orchestrating API calls
* managing loading state
* mapping raw responses
* exposing UI-ready data

---

### 3. Loading State Management

Before initiating a network request, the DataStore must update a loading state variable.

This allows the View to reactively display:

* progress indicators
* skeleton views
* disabled UI interactions

All API calls must expose loading state to ensure consistent user feedback during network operations.

---

### 4. DataStore → Networking Layer

The DataStore communicates with the networking layer through the **NetworkService** abstraction.

This layer handles:

* request construction
* HTTP execution
* authentication headers
* automatic token refresh
* response validation

The networking stack consists of:

DataStore → NetworkServiceFactory → WebRepository → HTTPClient

---

### 5. HTTPClient Execution

The HTTPClient performs the actual server communication by:

* building URL requests
* attaching authorization tokens
* executing requests using URLSession
* mapping HTTP status codes
* decoding JSON responses

This centralized logic ensures consistent networking behavior across the application.

---

### 6. Error Processing

All errors are routed through the **NetworkResponseEnvironment**.

This component is responsible for:

* interpreting backend error models
* mapping URL and connectivity errors
* triggering global UI alerts
* standardizing error messaging

Views do not directly handle networking errors.

---

### 7. Response Mapping

Once a response is successfully received:

* the networking layer decodes it into strongly typed models
* the DataStore maps it into UI-ready state

This ensures the UI never works with raw API responses.

---

### 8. Reactive UI Update

After the DataStore updates its published state:

* SwiftUI automatically re-renders the View
* no manual UI refresh logic is required

This reactive binding guarantees synchronization between data and UI.

---

## MockWebService Integration

Every new API endpoint must also be implemented in **MockWebService**.

Mocks are required to support:

* SwiftUI previews
* unit testing
* UI testing
* offline development

This ensures deterministic behavior and allows developers to work independently of backend availability.

---

## End-to-End Flow Summary

The complete lifecycle can be summarized as:

User Action
→ View triggers DataStore
→ DataStore updates loading state
→ DataStore calls NetworkService
→ HTTPClient communicates with API
→ Response is decoded and mapped
→ DataStore updates published state
→ SwiftUI automatically updates UI

---

## Why This Data Flow Design

This architecture provides:

* predictable and traceable data movement
* strong separation of concerns
* improved testability
* centralized networking logic
* consistent error handling
* automatic UI synchronization

It aligns with modern SwiftUI and MVVM best practices while ensuring scalability for large applications.


------------------------------------------------------------------------

### Testing Feature
- Unit test ViewModel
- Add mock responses
- Validate states

------------------------------------------------------------------------

## Key Principles

-   Separation of concerns
-   Unidirectional data flow
-   Testable networking
-   Feature-based architecture

------------------------------------------------------------------------

## Notes

This is a living document and should evolve with the project.
