# Laravel & Filament Development Standards

**Version:** 1.1.0 (Enhanced for TALL Stack)
**Context:** Universal Monolith & Multi-Tenant SaaS
**Enforcement:** Mandatory for all contributors

---

## 1. Introduction & Philosophy

In high-velocity development environments, architectural inconsistency is the primary driver of technical debt. This document establishes a **Universal Standard** for Laravel projects utilizing **Filament**, aiming to minimize cognitive load and maximize predictability.

Our philosophy is built on three pillars:

1. **Semantic Clarity:** Directory structures and naming must explicitly describe the business intent.
2. **Structural Predictability:** A developer entering any domain within the codebase should immediately understand the hierarchy without documentation.
3. **Scalability:** The architecture must support infinite horizontal growth of features without cluttering the root namespace.

---

## 2. Naming Conventions

Strict adherence to naming conventions ensures that code remains uniform across different modules, including Filament components.

| Code Element | Case Format | Pattern / Standard | Examples |
| --- | --- | --- | --- |
| **Controller** | `PascalCase` | Singular + Suffix | `OrderController`, `PaymentController` |
| **Model** | `PascalCase` | Singular Entity | `User`, `Invoice`, `Shop` |
| **Filament Resource** | `PascalCase` | Singular + Resource | `OrderResource`, `ProductResource` |
| **Filament Cluster** | `PascalCase` | Domain Name | `Logistics`, `Finance`, `Settings` |
| **Filament Widget** | `PascalCase` | Context + Stats/Chart | `RevenueStats`, `OrdersChart` |
| **Database Table** | `snake_case` | Plural | `orders`, `order_items` |
| **Route Name** | `kebab.dot` | Hierarchical | `shop.product.checkout` |

---

## 3. Directory Architecture (Semantic Nested Tree)

Flat directory structures are strictly prohibited. We utilize a **Scope-First, Domain-Second** architectural pattern.

### A. Filament Panels (`app/Filament`)

Filament resources **MUST NOT** reside loosely in the root `Resources` directory. We enforce the use of **Clusters** to group resources by their Domain Context.

```text
app/Filament/
├── [Panel_Name]/               <-- L1: Scope (e.g., 'Admin', 'App')
│   ├── Clusters/               <-- L2: Domain Grouping (Strictly Enforced)
│   │   ├── [Domain_Name]/      <-- e.g., 'Finance'
│   │   │   ├── Resources/      <-- Resources belonging to Finance
│   │   │   │   ├── InvoiceResource.php
│   │   │   │   └── TransactionResource.php
│   │   │   └── Pages/          <-- Custom Pages for Finance
│   │   │       └── FinancialReport.php
│   │   │
│   │   └── [Domain_Name_B]/    <-- e.g., 'Logistics'
│   │       └── ...
│   │
│   ├── Pages/                  <-- Only for Dashboard or Global Pages
│   └── Widgets/                <-- Global Widgets
│
└── ...

```

### B. Controllers (`app/Http/Controllers`)

Used primarily for the **Public Storefront** or API endpoints, not for the Admin Panel.

```text
app/Http/Controllers/
├── [Access_Scope]/             <-- L1: e.g., 'Storefront', 'API'
│   ├── [Domain_Context]/       <-- L2: Feature Group (e.g., 'Checkout')
│   │   ├── [Resource]Controller.php
│   │   └── [Action]Controller.php
│   └── ...
└── Controller.php

```

### C. Views (`resources/views`)

Mirroring pattern applies. Filament views (custom blade) must be isolated.

```text
resources/views/
├── filament/                   <-- Filament Specific Views
│   ├── [panel-slug]/           <-- e.g., 'app'
│   │   ├── [cluster-slug]/     <-- e.g., 'finance'
│   │   │   └── pages/          <-- Custom Page Views
│   │   └── ...
│
└── storefront/                 <-- Public Facing Views
    ├── [domain-slug]/
    │   └── ...

```

---

## 4. Filament Implementation Standards

To maintain the "High Profile" quality of the SaaS, specific UI/UX patterns are enforced within the PHP classes.

### A. The "God Mode" View (Infolists)

Do not rely on "Edit Forms" for viewing data. Use **Infolists** for a read-only, document-style presentation.

* **Requirement:** Every Resource must define an `infolist()` method.
* **Style:** Use `Section` and `Split` components to organize data logically, mirroring a real-world document (e.g., Invoice).

### B. Action Grouping

To prevent UI clutter on tables with many actions.

* **Rule:** If a Table or Page has more than **2 primary actions**, they must be grouped using `ActionGroup`.
* **Iconography:** Use consistent Heroicons (Outline version).

### C. Multi-Tenancy Scoping

Since `Bhandara` is a multi-tenant SaaS:

* **Rule:** **NEVER** query data using `User::find()` or `Model::all()` in the `App` panel.
* **Standard:** Always rely on Filament's automatic tenant scoping or use `$tenant = Filament::getTenant();`.

---

## 5. Routing Guidelines (The "Clean Route" Protocol)

Filament handles its own routing via Service Providers. This section applies to **Public Storefront** and **Custom Controllers**.

**Rules:**

1. **No Closures:** Logic inside `routes/web.php` is strictly prohibited.
2. **One Line Per Route:** Each route definition must occupy a single line.
3. **Strict Grouping:** Routes must be nested within `Access Scope` and `Domain` groups.

**Implementation Standard:**

```php
use App\Http\Controllers\Storefront\Order\CheckoutController;

// GROUP: PUBLIC STOREFRONT (Tenant Aware)
Route::prefix('u/{shop_slug}')->name('storefront.')->group(function () {

    // Domain: Catalog
    Route::controller(CatalogController::class)->group(function () {
        Route::get('/', 'index')->name('home');
        Route::get('/p/{product}', 'show')->name('product.show');
    });

    // Domain: Checkout
    Route::prefix('checkout')->name('checkout.')->group(function () {
        Route::post('/cart/add', [CartController::class, 'add'])->name('cart.add');
        
        // Single Responsibility Action
        Route::post('/process', [CheckoutController::class, 'process'])->name('process');
    });

});

```

---

## 6. Governance & Code Review

This standard is binding. During the Code Review process, the following violations will trigger an immediate rejection:

1. **Flat Filament Resources:** Creating a Resource in the root folder instead of a **Cluster**.
2. **Logic in Routes:** Using Closures in `web.php`.
3. **Missing Type Hinting:** All Filament form components must be strictly typed.
4. **Tenant Leak:** Accessing global data without tenant scoping in the App Panel.
