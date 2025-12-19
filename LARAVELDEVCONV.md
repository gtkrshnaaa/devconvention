# Laravel Development Standards & Architectural Guidelines

**Version:** 1.0.0
**Context:** Universal Monolith Architecture
**Enforcement:** Mandatory for all contributors

---

## 1. Introduction & Philosophy

In high-velocity development environments, architectural inconsistency is the primary driver of technical debt. This document establishes a **Universal Standard** for Laravel projects, aiming to minimize cognitive load and maximize predictability.

Our philosophy is built on three pillars:

1. **Semantic Clarity:** Directory structures and naming must explicitly describe the business intent.
2. **Structural Predictability:** A developer entering any domain within the codebase should immediately understand the hierarchy without documentation.
3. **Scalability:** The architecture must support infinite horizontal growth of features without cluttering the root namespace.

---

## 2. Naming Conventions

Strict adherence to naming conventions ensures that code remains uniform across different modules.

| Code Element | Case Format | Pattern / Standard | Examples |
| --- | --- | --- | --- |
| **Controller** | `PascalCase` | Singular + Suffix | `[Resource]Controller`, `OrderController` |
| **Model** | `PascalCase` | Singular Entity | `[Entity]`, `User`, `Invoice` |
| **Method** | `camelCase` | Verb + Noun | `store[Resource]`, `verify[Status]` |
| **Variable** | `snake_case` | Lowercase | `$variable_name`, `$total_amount` |
| **Database Table** | `snake_case` | Plural | `[entities]`, `order_items` |
| **View Directory** | `kebab-case` | Lowercase | `resources/views/[domain-name]/` |
| **Route Name** | `kebab.dot` | Hierarchical | `[scope].[resource].[action]` |

---

## 3. Directory Architecture (Semantic Nested Tree)

Flat directory structures are strictly prohibited. We utilize a **Scope-First, Domain-Second** architectural pattern to organize logic.

### A. Controllers (`app/Http/Controllers`)

Controllers are organized primarily by **Access Scope** (Who is using it?), followed by **Domain Context** (What feature is it?).

```text
app/Http/Controllers/
├── [Access_Scope]/                 <-- L1: e.g., 'Admin', 'Merchant', 'Public'
│   ├── DashboardController.php     <-- Scope-level Logic
│   │
│   ├── [Domain_Context_A]/         <-- L2: Feature Group A (e.g., 'Finance')
│   │   ├── [Resource]Controller.php
│   │   └── [SubResource]Controller.php
│   │
│   └── [Domain_Context_B]/         <-- L2: Feature Group B (e.g., 'Inventory')
│       └── [SpecificProcess]Controller.php
│
└── Controller.php                  <-- Base Controller

```

### B. Views (`resources/views`)

The View layer must strictly implement a **Mirroring Pattern** against the Controller structure. Layout files are named explicitly after the Access Scope they serve.

```text
resources/views/
├── layouts/                        <-- Master Layout Definitions
│   ├── [scope-slug-a].blade.php    <-- e.g., 'admin.blade.php'
│   ├── [scope-slug-b].blade.php    <-- e.g., 'merchant.blade.php'
│   └── components/                 <-- Shared Partials
│
├── [scope-slug-a]/                 <-- Views for Scope A
│   ├── dashboard/
│   │   └── index.blade.php
│   │
│   ├── [domain-slug-a]/            <-- Domain Context
│   │   ├── [resource-name]/
│   │   │   ├── index.blade.php
│   │   │   ├── create.blade.php
│   │   │   └── edit.blade.php
│   │
│   └── [domain-slug-b]/
│       └── ...
│
└── [scope-slug-b]/                 <-- Views for Scope B
    └── ...

```

---

## 4. Routing Guidelines (The "Clean Route" Protocol)

Routing files often become the messiest part of a monolith. To prevent this, we enforce the **One-Line Rule**.

**Rules:**

1. **No Closures:** Logic inside `routes/web.php` is strictly prohibited. All logic must reside in Controllers.
2. **One Line Per Route:** Each route definition must occupy a single line.
3. **Strict Grouping:** Routes must be nested within `Access Scope` and `Domain` groups.

**Implementation Standard:**

```php
use App\Http\Controllers\[Access_Scope]\[Domain_Context]\[Resource]Controller;

// GROUP 1: ACCESS SCOPE (e.g., Admin Panel, API, Member Area)
Route::prefix('[scope-slug]')->name('[scope].')->group(function () {

    // A. Scope-Level Routes
    Route::get('/', [DashboardController::class, 'index'])->name('dashboard');

    // B. DOMAIN CONTEXT GROUP (Feature Specific)
    Route::prefix('[domain-slug]')->name('[domain].')->group(function () {
        
        // Resource Controller (Preferred for CRUD)
        Route::resource('[resources]', [Resource]Controller::class);

        // Single Responsibility Actions (ONE LINE RULE)
        Route::get('[resource]/export', [[Resource]Controller::class, 'export'])->name('[resource].export');
        Route::post('[resource]/verify', [[Resource]Controller::class, 'verify'])->name('[resource].verify');
        
    });

    // C. SECONDARY DOMAIN GROUP
    Route::prefix('[other-domain]')->name('[other-domain].')->group(function () {
        Route::get('stats', [AnalyticsController::class, 'daily'])->name('analytics.daily');
    });

});

```

---

## 5. Governance & Code Review

This standard is binding. During the Code Review process, any Pull Request found violating these structural or naming conventions—specifically **flat directory structures** or **closure-based routing**—will be rejected and must be refactored before merging.
