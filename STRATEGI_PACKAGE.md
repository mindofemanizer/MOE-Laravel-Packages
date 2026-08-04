# Strategi Package & Module — MOE Architecture

**Versi:** 4.1  
**Status:** Active  
**Terakhir diperbarui:** 2026-08-04

---

## 1. Visi

Semua modul bisnis dibangun sebagai **Composer package mandiri** yang bisa di-install lintas project. Satu package, dipakai di mana pun.

---

## 2. Standar Ekosistem

| Aspek | Nilai |
|-------|-------|
| Namespace PHP | `Moe\` (PascalCase, BUKAN `MOE\`) |
| Branch default | `main` |
| Constraint Composer | `dev-main` (Composer membaca = branch `main`) |
| PHP | `^8.2` |
| Illuminate | `^11 \| ^12 \| ^13` |
| Test framework | **Pest ^3.0** (wrapper PHPUnit, syntax `it()` / `expect()`) |
| Test harness | Orchestra Testbench `^8 \| ^9 \| ^10` |
| Repository | VCS GitHub (`https://github.com/mindofemanizer/MOE-Laravel-{Name}`) |
| Total package | 23 |

> **Peringatan case-sensitivity:** Linux filesystem case-sensitive. Namespace `MOE\` akan gagal autoload di Linux meski PHP class name case-insensitive. SELALU pakai `Moe\`.

---

## 3. Arsitektur

```
INFRASTRUCTURE
├── moe/laravel-core              → Base contracts, traits, exceptions, BaseService
├── moe/laravel-settings          → Global key-value (typed, cached, encrypted-at-rest)
├── moe/laravel-profiles          → Per-user key-value (morph, typed fields, HasProfiles trait)
├── moe/laravel-auth              → Livewire komponen (login, register, forgot/reset password)
└── moe/laravel-foundation        → Meta-package: require Core+Settings+Profiles+Auth; middleware (SetLocale/MaintenanceMode); trait HasAccount

UTILITY (cross-cutting concerns)
├── moe/laravel-identifiers       → Sqids, Document Number, Facade MoeId
├── moe/laravel-content-workflow  → State machine, scheduling, versioning, audit log, WYSIWYG (Tiptap)
├── moe/laravel-image-pipeline    → Image processing (preset-driven, queue-aware, watermark), Facade MoeImage
├── moe/laravel-notify            → Notification multi-channel
├── moe/laravel-task              → Task & assignment management
└── moe/laravel-template          → Template management (email, dokumen, dll)

BUSINESS MODULES
├── moe/laravel-commerce          → Store, Product, Cart, Order, Invoice + Checkout
├── moe/laravel-inventory         → Inventory, InventoryMovement, InventoryService
├── moe/laravel-shipping          → Courier, Zone, CourierZoneRate, ShippingService
├── moe/laravel-finance           → Wallet, WalletTransaction, WalletService, Payment, Refund
├── moe/laravel-vendor-b2b        → Vendor, PurchaseOrder, VendorPayout + services
├── moe/laravel-marketing         → CommissionLedger, Promo, Referral, Attribution + services
├── moe/laravel-hrm               → Employee, Department, Attendance, Payroll, Leave
├── moe/laravel-crm               → Contact, Segment, Lead, Activity
├── moe/laravel-invoice           → Invoice generation & management
├── moe/laravel-payment           → Payment gateway abstraction & processing
├── moe/laravel-subscription      → Plan, Subscription, recurring billing
└── moe/laravel-multi-tenant      → Multi-tenancy (tenant scoping, resolution)
```

---

## 4. Relasi antar Module

### 4.1 Dependency Graph

```
moe/laravel-core (base)
├── moe/laravel-finance (depends: core)
├── moe/laravel-inventory (depends: core)
├── moe/laravel-shipping (depends: core)
├── moe/laravel-commerce (depends: core, inventory, shipping)
├── moe/laravel-vendor-b2b (depends: core, inventory)
├── moe/laravel-marketing (depends: core)
├── moe/laravel-hrm (depends: core)
├── moe/laravel-crm (depends: core)
├── moe/laravel-invoice (depends: core)
├── moe/laravel-payment (depends: core)
├── moe/laravel-subscription (depends: core)
├── moe/laravel-notify (depends: core)
├── moe/laravel-task (depends: core)
├── moe/laravel-template (depends: core)
├── moe/laravel-multi-tenant (depends: core)
├── moe/laravel-settings (depends: core)
├── moe/laravel-profiles (depends: core)
├── moe/laravel-auth (depends: core)
├── moe/laravel-foundation (depends: core, settings, profiles, auth)
├── moe/laravel-identifiers (depends: core)
├── moe/laravel-content-workflow (depends: core)
└── moe/laravel-image-pipeline (depends: core)
```

> **Catatan:** Saat ini dependency di composer.json minimal (hanya ke `core`, kecuali commerce yang butuh inventory + shipping).
> Ke depan relasi antar-module via contract (Pasal 4.2) dan event (Pasal 4.3) tanpa hard dependency.

### 4.2 Contract Pattern

Module A tidak import model Module B. Module A pakai contract yang didefinisikan Module B.

**Contoh:**

```php
// moe/finance/src/Contracts/WalletProviderInterface.php
interface WalletProviderInterface
{
    public function getWallet(): ?Wallet;
    public function getBalance(): float;
    public function credit(float $amount, string $type, ?string $description = null): WalletTransaction;
    public function debit(float $amount, string $type, ?string $description = null): WalletTransaction;
}
```

```php
// moe/crm/src/Services/SegmentationService.php
public function getTopSpenders(int $limit = 10): Collection
{
    // Tidak import User model, pakai contract
    $providers = app(WalletProviderInterface::class);
    // ... logic
}
```

### 4.3 Event Pattern

Module berkomunikasi via Laravel Events, bukan direct call.

```php
// moe/commerce dispatch event
event(new OrderCompleted($order));

// moe/finance listen event
OrderCompleted::class => CreditWallet::class,

// moe/marketing listen event
OrderCompleted::class => RecognizeCommission::class,

// moe/crm listen event
OrderCompleted::class => LogCustomerActivity::class,
```

---

## 5. Struktur Package

### 5.1 Standard Structure

```
MOE-Laravel-{Module}/
├── src/
│   ├── {Module}ServiceProvider.php
│   ├── Contracts/
│   │   └── {Interface}.php
│   ├── Models/
│   │   └── {Model}.php
│   ├── Services/
│   │   └── {Service}.php
│   ├── Livewire/
│   │   └── {Component}.php
│   ├── Http/
│   │   └── Controllers/
│   ├── Events/
│   │   └── {Event}.php
│   ├── Listeners/
│   │   └── {Listener}.php
│   ├── Traits/
│   │   └── {Trait}.php
│   └── Exceptions/
│       └── {Exception}.php
├── config/
│   └── {module}.php
├── database/
│   ├── migrations/
│   │   └── {timestamp}_{description}.php
│   └── seeders/
│       └── {Module}Seeder.php
├── resources/
│   └── views/
│       └── livewire/
│           └── {component}.blade.php
├── routes/
│   └── web.php
├── lang/
│   └── id/
│       └── {module}.php
├── tests/
│   ├── Pest.php
│   ├── TestCase.php
│   └── Unit/
│       └── {Service}Test.php
├── composer.json
├── phpunit.xml
├── .gitignore
└── README.md
```

### 5.2 Service Provider Pattern

```php
<?php

namespace Moe\Finance;

use Illuminate\Support\ServiceProvider;
use Moe\Finance\Contracts\WalletProviderInterface;
use Moe\Finance\Services\WalletService;

class FinanceServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(WalletProviderInterface::class, WalletService::class);
    }

    public function boot(): void
    {
        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'finance');
        $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'finance');

        $this->publishes([
            __DIR__.'/../config/finance.php' => config_path('finance.php'),
        ], 'finance-config');
    }
}
```

### 5.3 composer.json Pattern

```json
{
    "name": "moe/laravel-finance",
    "description": "Finance module for MOE ecosystem — Wallet, Payment, Refund",
    "type": "library",
    "license": "MIT",
    "autoload": {
        "psr-4": {
            "Moe\\Finance\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Moe\\Finance\\Tests\\": "tests/"
        }
    },
    "require": {
        "php": "^8.2",
        "illuminate/support": "^11|^12|^13"
    },
    "require-dev": {
        "orchestra/testbench": "^9.0",
        "pestphp/pest": "^3.0"
    },
    "config": {
        "allow-plugins": {
            "pestphp/pest-plugin": true
        },
        "sort-packages": true
    },
    "scripts": {
        "test": "pest",
        "test-coverage": "pest --coverage"
    },
    "extra": {
        "laravel": {
            "providers": [
                "Moe\\Finance\\FinanceServiceProvider"
            ]
        }
    },
    "minimum-stability": "dev",
    "prefer-stable": true
}
```

### 5.4 .gitignore Standard

Semua 23 repo memakai `.gitignore` yang sama:

```gitignore
/vendor/
/node_modules/
/.idea/
/.vscode/
/.phpunit.cache/
.phpunit.result.cache
composer.lock
.DS_Store
Thumbs.db
*.log
.env
```

> `vendor/` dan `composer.lock` tidak di-commit. Consumer project yang punya `composer.lock` sendiri.

### 5.5 Testing Standard — Pest

Semua 23 package memakai **Pest ^3.0** sebagai test framework. Pest adalah wrapper di atas PHPUnit dengan syntax fungsional yang lebih ekspresif.

#### tests/Pest.php

```php
<?php

use Moe\{Module}\Tests\TestCase;

uses(TestCase::class)->in(__DIR__);
```

#### tests/TestCase.php

```php
<?php

namespace Moe\{Module}\Tests;

use Moe\{Module}\{Module}ServiceProvider;
use Orchestra\Testbench\TestCase as Orchestra;

abstract class TestCase extends Orchestra
{
    protected function getPackageProviders($app): array
    {
        return [{Module}ServiceProvider::class];
    }

    protected function defineEnvironment($app): void
    {
        $app['config']->set('database.default', 'testing');
        $app['config']->set('database.connections.testing', [
            'driver' => 'sqlite',
            'database' => ':memory:',
            'prefix' => '',
        ]);
    }
}
```

#### Test File Pattern

```php
<?php

use Moe\{Module}\Models\{Model};
use Moe\{Module}\Services\{Service};

beforeEach(function () {
    $this->service = new {Service}();
});

it('does something', function () {
    $result = $this->service->doSomething();

    expect($result)->toBeInstanceOf({Model}::class);
    expect($result->status)->toEqual('active');
});

it('throws on invalid input', function () {
    expect(fn () => $this->service->invalidCall())
        ->toThrow(\Exception::class);
});
```

#### Konversi PHPUnit → Pest

| PHPUnit | Pest |
|---------|------|
| `class FooTest extends TestCase` | (hapus class, file langsung) |
| `protected function setUp()` | `beforeEach(function () { ... })` |
| `public function test_foo()` | `it('foo', function () { ... })` |
| `$this->assertTrue($x)` | `expect($x)->toBeTrue()` |
| `$this->assertEquals($a, $b)` | `expect($b)->toEqual($a)` |
| `$this->assertSame($a, $b)` | `expect($b)->toBe($a)` |
| `$this->assertInstanceOf(C::class, $x)` | `expect($x)->toBeInstanceOf(C::class)` |
| `$this->assertCount($n, $arr)` | `expect($arr)->toHaveCount($n)` |
| `$this->assertNull($x)` | `expect($x)->toBeNull()` |
| `$this->assertNotNull($x)` | `expect($x)->not->toBeNull()` |
| `$this->expectException(E::class)` | `expect(fn () => ...)->toThrow(E::class)` |
| `$this->assertDatabaseHas(...)` | `$this->assertDatabaseHas(...)` (tetap, via TestCase) |

> **Catatan:** `$this->assertDatabaseHas()`, `$this->assertAuthenticatedAs()`, dll. tetap tersedia karena Pest mewarisi TestCase via `uses(TestCase::class)`.

---

## 6. Migration Strategy

### 6.1 Prefix TableName

Semua tabel pakai prefix module untuk hindari conflict:

```php
// moe/finance
Schema::create('finance_wallets', function (Blueprint $table) { ... });
Schema::create('finance_transactions', function (Blueprint $table) { ... });

// moe/crm
Schema::create('crm_activities', function (Blueprint $table) { ... });
Schema::create('crm_segments', function (Blueprint $table) { ... });
```

### 6.2 Foreign Keys

Foreign key pakai naming convention `{module}_{table}_id`:

```php
$table->foreignId('finance_wallet_id')->constrained('finance_wallets');
$table->foreignId('commerce_order_id')->constrained('commerce_orders');
```

### 6.3 Configurable Table Names

Setiap module punya config untuk override table name:

```php
// config/finance.php
return [
    'tables' => [
        'wallets' => 'finance_wallets',
        'transactions' => 'finance_transactions',
    ],
];
```

```php
// Model
class Wallet extends Model
{
    protected $table;
    
    public function __construct(array $attributes = [])
    {
        parent::__construct($attributes);
        $this->table = config('finance.tables.wallets', 'finance_wallets');
    }
}
```

---

## 7. Config Strategy

### 7.1 Publish Config

```bash
php artisan vendor:publish --provider="Moe\Finance\FinanceServiceProvider" --tag="finance-config"
```

### 7.2 Config Merge

Module config di-merge dengan project config:

```php
// config/finance.php (project level)
return [
    'currency' => 'IDR',
    'min_withdraw' => 50000,
    'hold_days' => 7,
    'tables' => [
        'wallets' => 'wallets', // override table name
    ],
];
```

---

## 8. Installation Guide

### 8.1 Dari GitHub (VCS)

Tambahkan repository VCS untuk setiap package yang dibutuhkan di `composer.json` project:

```json
"repositories": [
    { "type": "vcs", "url": "https://github.com/mindofemanizer/MOE-Laravel-Core" },
    { "type": "vcs", "url": "https://github.com/mindofemanizer/MOE-Laravel-Finance" }
]
```

Lalu:

```bash
composer require moe/laravel-core:dev-main
composer require moe/laravel-finance:dev-main
php artisan vendor:publish --provider="Moe\Finance\FinanceServiceProvider" --tag="finance-config"
php artisan vendor:publish --provider="Moe\Finance\FinanceServiceProvider" --tag="finance-migrations"
php artisan migrate
```

> Lihat `prompt-integrasi-package.md` untuk daftar lengkap 23 package beserta tag publish dan urutan integrasi.

### 8.2 Contoh Kombinasi

```bash
# App baru dengan Foundation (setting + profil + auth langsung siap)
composer require moe/laravel-foundation:dev-main
# Foundation otomatis narik: core, settings, profiles, auth

# Toko saja
composer require moe/laravel-core:dev-main moe/laravel-finance:dev-main moe/laravel-inventory:dev-main moe/laravel-commerce:dev-main

# Full platform (seperti KiosKit)
composer require moe/laravel-foundation:dev-main moe/laravel-finance:dev-main moe/laravel-inventory:dev-main moe/laravel-shipping:dev-main moe/laravel-commerce:dev-main moe/laravel-vendor-b2b:dev-main moe/laravel-marketing:dev-main moe/laravel-crm:dev-main moe/laravel-hrm:dev-main
```

---

## 9. Repository GitHub — 23 Package

### Infrastructure

| Package | Repo | Provider |
|---------|------|----------|
| `moe/laravel-core` | https://github.com/mindofemanizer/MOE-Laravel-Core | `Moe\Core\CoreServiceProvider` |
| `moe/laravel-settings` | https://github.com/mindofemanizer/MOE-Laravel-Settings | `Moe\Settings\MoeSettingsServiceProvider` |
| `moe/laravel-profiles` | https://github.com/mindofemanizer/MOE-Laravel-Profiles | `Moe\Profiles\MoeProfilesServiceProvider` |
| `moe/laravel-auth` | https://github.com/mindofemanizer/MOE-Laravel-Auth | `Moe\Auth\MoeAuthServiceProvider` |
| `moe/laravel-foundation` | https://github.com/mindofemanizer/MOE-Laravel-Foundation | `Moe\Foundation\MoeFoundationServiceProvider` |

### Utility

| Package | Repo | Provider |
|---------|------|----------|
| `moe/laravel-identifiers` | https://github.com/mindofemanizer/MOE-Laravel-Identifiers | `Moe\Identifiers\IdentifiersServiceProvider` |
| `moe/laravel-content-workflow` | https://github.com/mindofemanizer/MOE-Laravel-Content-Workflow | `Moe\ContentWorkflow\ContentWorkflowServiceProvider` |
| `moe/laravel-image-pipeline` | https://github.com/mindofemanizer/MOE-Laravel-Image-Pipeline | `Moe\ImagePipeline\ImagePipelineServiceProvider` |
| `moe/laravel-notify` | https://github.com/mindofemanizer/MOE-Laravel-Notify | `Moe\Notify\NotifyServiceProvider` |
| `moe/laravel-task` | https://github.com/mindofemanizer/MOE-Laravel-Task | `Moe\Task\TaskServiceProvider` |
| `moe/laravel-template` | https://github.com/mindofemanizer/MOE-Laravel-Template | `Moe\Template\TemplateServiceProvider` |

### Business Modules

| Package | Repo | Provider |
|---------|------|----------|
| `moe/laravel-commerce` | https://github.com/mindofemanizer/MOE-Laravel-Commerce | `Moe\Commerce\CommerceServiceProvider` |
| `moe/laravel-inventory` | https://github.com/mindofemanizer/MOE-Laravel-Inventory | `Moe\Inventory\InventoryServiceProvider` |
| `moe/laravel-shipping` | https://github.com/mindofemanizer/MOE-Laravel-Shipping | `Moe\Shipping\ShippingServiceProvider` |
| `moe/laravel-finance` | https://github.com/mindofemanizer/MOE-Laravel-Finance | `Moe\Finance\FinanceServiceProvider` |
| `moe/laravel-vendor-b2b` | https://github.com/mindofemanizer/MOE-Laravel-Vendor-B2B | `Moe\VendorB2B\VendorB2BServiceProvider` |
| `moe/laravel-marketing` | https://github.com/mindofemanizer/MOE-Laravel-Marketing | `Moe\Marketing\MarketingServiceProvider` |
| `moe/laravel-hrm` | https://github.com/mindofemanizer/MOE-Laravel-HRM | `Moe\HRM\HRMServiceProvider` |
| `moe/laravel-crm` | https://github.com/mindofemanizer/MOE-Laravel-CRM | `Moe\CRM\CRMServiceProvider` |
| `moe/laravel-invoice` | https://github.com/mindofemanizer/MOE-Laravel-Invoice | `Moe\Invoice\InvoiceServiceProvider` |
| `moe/laravel-payment` | https://github.com/mindofemanizer/MOE-Laravel-Payment | `Moe\Payment\PaymentServiceProvider` |
| `moe/laravel-subscription` | https://github.com/mindofemanizer/MOE-Laravel-Subscription | `Moe\Subscription\SubscriptionServiceProvider` |
| `moe/laravel-multi-tenant` | https://github.com/mindofemanizer/MOE-Laravel-MultiTenant | `Moe\MultiTenant\MultiTenantServiceProvider` |

---

## 10. Naming Convention

| Item | Convention | Contoh |
|------|-----------|--------|
| Package name | `moe/laravel-{module}` | `moe/laravel-finance` |
| Namespace | `Moe\{Module}` | `Moe\Finance` |
| Repo name | `MOE-Laravel-{Module}` | `MOE-Laravel-Finance` |
| Table prefix | `{module}_` | `finance_wallets` |
| Config key | `{module}.{key}` | `finance.currency` |
| Lang key | `{module}::{key}` | `finance::wallet.balance` |
| Route name | `{module}.{name}` | `finance.wallet.topup` |
| View prefix | `{module}::` | `finance::wallet.index` |
| Tag publish | `{module}-config`, `{module}-migrations`, `{module}-views` | `finance-config`, `finance-migrations` |
| Branch | `main` | — |
| Constraint | `dev-main` | — |

---

## 11. Referensi

- [Laravel Package Development](https://laravel.com/docs/master/packages)
- [Package Auto-Discovery](https://laravel.com/docs/master/packages#package-discovery)
- Lihat juga: `prompt-integrasi-package.md` — prompt siap tempel untuk integrasi ke project baru
