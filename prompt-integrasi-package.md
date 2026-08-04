# Prompt Integrasi Package MOE ke Project Lain

Copy prompt ini ke agen yang bekerja di project target. Ganti `[NAMA_PROJECT]` dengan nama project yang dituju.

---

## PROMPT

```
Kamu sedang bekerja di project [NAMA_PROJECT]. Tugas kamu: integrasikan reusable package yang sudah dibuat di GitHub (akun mindofemanizer).

═══════════════════════════════════════════════════════════════
STANDAR EKOSISTEM MOE
═══════════════════════════════════════════════════════════════

- Namespace PHP : Moe\  (PascalCase, BUKAN MOE\)
- Branch default: main
- Constraint    : dev-main  (Composer membaca dev-main = branch main)
- PHP           : ^8.2
- Illuminate    : ^11 | ^12 | ^13

Semua package butuh repository VCS di composer.json project:
"repositories": [{ "type": "vcs", "url": "https://github.com/mindofemanizer/NAMA_REPO" }]

Pola install umum:
composer require moe/NAMA_PACKAGE:dev-main
php artisan vendor:publish --provider="Moe\XXX\XxxServiceProvider" --tag="xxx-config"
php artisan vendor:publish --provider="Moe\XXX\XxxServiceProvider" --tag="xxx-migrations"  (kalau ada)
php artisan migrate

Catatan: tag publish TIDAK seragam antar package. Lihat kolom "Tag" di tiap package di bawah — pakai persis yang tertera.

═══════════════════════════════════════════════════════════════
DAFTAR 23 PACKAGE MOE
═══════════════════════════════════════════════════════════════

--- INFRASTRUCTURE ---

1. moe/laravel-core
   Repo     : MOE-Laravel-Core
   Provider : Moe\Core\CoreServiceProvider
   Tag      : core-config
   Migrasi  : tidak ada
   Isi      : Base contracts, traits, exceptions, BaseService. Hampir semua package depend ke sini → install PERTAMA.

2. moe/laravel-settings
   Repo     : MOE-Laravel-Settings
   Provider : Moe\Settings\MoeSettingsServiceProvider
   Tag      : moe-settings-config, moe-settings-views
   Migrasi  : ya
   Isi      : Global key-value settings (typed, cached, encrypted-at-rest) + UI.
   Catatan  : Tambah di resources/css/app.css: @source "../../vendor/moe/**/resources/views"; lalu npm run build.
              Embed di Livewire/admin: <livewire:settings-manager />.
              Grup: general, address, mail, storage, security, notifications, backup, api, payment.
              Tipe field: text, textarea, toggle, select, checkbox_group, password(encrypted), number, image, livewire_component.
              RecordsActivity trait: RecordsActivity::record('action', $model, $modelId, $old, $new) → audit_logs.

3. moe/laravel-profiles
   Repo     : MOE-Laravel-Profiles
   Provider : Moe\Profiles\MoeProfilesServiceProvider
   Tag      : moe-profiles-config
   Migrasi  : ya
   Isi      : Per-user key-value (morph, typed fields, HasProfiles trait).

4. moe/laravel-auth
   Repo     : MOE-Laravel-Auth
   Provider : Moe\Auth\MoeAuthServiceProvider
   Tag      : moe-auth-config, moe-auth-migrations, moe-auth-views, moe-auth-translations
   Migrasi  : ya
   Isi      : Livewire komponen auth (login, register, forgot/reset password).

5. moe/laravel-foundation
   Repo     : MOE-Laravel-Foundation
   Provider : Moe\Foundation\MoeFoundationServiceProvider
   Tag      : moe-foundation-config
   Migrasi  : tidak ada
   Isi      : Meta-package: require Core+Settings+Profiles+Auth; middleware (SetLocale/MaintenanceMode); trait HasAccount.

--- UTILITY (cross-cutting) ---

6. moe/laravel-identifiers
   Repo     : MOE-Laravel-Identifiers
   Provider : Moe\Identifiers\IdentifiersServiceProvider
   Tag      : moe-identifiers-config, moe-identifiers-migrations
   Migrasi  : ya
   Isi      : Public ID obfuscation (trait HasPublicId) + Document numbering (trait HasDocumentNumber). Facade MoeId.

7. moe/laravel-content-workflow
   Repo     : MOE-Laravel-Content-Workflow
   Provider : Moe\ContentWorkflow\ContentWorkflowServiceProvider
   Tag      : moe-content-config, moe-content-migrations, moe-content-views
   Migrasi  : ya
   Isi      : State machine, scheduling, versioning, audit log, WYSIWYG (Tiptap).
              Blade: @moeContentStatus, @moeContentCan. Livewire: ContentEditor, ContentStatusManager, ContentScheduler, ContentVersionHistory, ContentAuditLog.

8. moe/laravel-image-pipeline
   Repo     : MOE-Laravel-Image-Pipeline
   Provider : Moe\ImagePipeline\ImagePipelineServiceProvider
   Tag      : moe-image-config
   Migrasi  : tidak ada
   Isi      : Pipeline kompresi & transformasi gambar (preset-driven, queue-aware, watermark). Facade MoeImage.
   Pakai    : MoeImage::preset('avatar')->disk('r2')->store($file, $dir)

--- BUSINESS MODULES ---

9. moe/laravel-commerce
   Repo     : MOE-Laravel-Commerce
   Provider : Moe\Commerce\CommerceServiceProvider
   Tag      : commerce-config, commerce-migrations
   Migrasi  : ya
   Isi      : Store, Product, Cart, Order, Invoice + Checkout.

10. moe/laravel-inventory
    Repo     : MOE-Laravel-Inventory
    Provider : Moe\Inventory\InventoryServiceProvider
    Tag      : inventory-config, inventory-migrations
    Migrasi  : ya
    Isi      : Inventory, InventoryMovement, InventoryService.

11. moe/laravel-shipping
    Repo     : MOE-Laravel-Shipping
    Provider : Moe\Shipping\ShippingServiceProvider
    Tag      : shipping-config, shipping-migrations
    Migrasi  : ya
    Isi      : Courier, Zone, CourierZoneRate, ShippingService.

12. moe/laravel-finance
    Repo     : MOE-Laravel-Finance
    Provider : Moe\Finance\FinanceServiceProvider
    Tag      : finance-config, finance-migrations
    Migrasi  : ya
    Isi      : Wallet, WalletTransaction, WalletService, Payment, Refund.

13. moe/laravel-vendor-b2b
    Repo     : MOE-Laravel-Vendor-B2B
    Provider : Moe\VendorB2B\VendorB2BServiceProvider
    Tag      : vendor-b2b-config, vendor-b2b-migrations
    Migrasi  : ya
    Isi      : Vendor, PurchaseOrder, VendorPayout + services.

14. moe/laravel-marketing
    Repo     : MOE-Laravel-Marketing
    Provider : Moe\Marketing\MarketingServiceProvider
    Tag      : marketing-config, marketing-migrations
    Migrasi  : ya
    Isi      : CommissionLedger, Promo, Referral, Attribution + services.

15. moe/laravel-hrm
    Repo     : MOE-Laravel-HRM
    Provider : Moe\HRM\HRMServiceProvider
    Tag      : hrm-config, hrm-migrations
    Migrasi  : ya
    Isi      : Employee, Department, Attendance, Payroll, Leave.

16. moe/laravel-crm
    Repo     : MOE-Laravel-CRM
    Provider : Moe\CRM\CRMServiceProvider
    Tag      : crm-config, crm-migrations
    Migrasi  : ya
    Isi      : Contact, Segment, Lead, Activity.

17. moe/laravel-invoice
    Repo     : MOE-Laravel-Invoice
    Provider : Moe\Invoice\InvoiceServiceProvider
    Tag      : moe-invoice-config
    Migrasi  : ya
    Isi      : Invoice generation & management.

18. moe/laravel-payment
    Repo     : MOE-Laravel-Payment
    Provider : Moe\Payment\PaymentServiceProvider
    Tag      : moe-payment-config
    Migrasi  : ya
    Isi      : Payment gateway abstraction & processing.

19. moe/laravel-subscription
    Repo     : MOE-Laravel-Subscription
    Provider : Moe\Subscription\SubscriptionServiceProvider
    Tag      : moe-subscription-config
    Migrasi  : ya
    Isi      : Plan, Subscription, recurring billing.

20. moe/laravel-notify
    Repo     : MOE-Laravel-Notify
    Provider : Moe\Notify\NotifyServiceProvider
    Tag      : moe-notify-config
    Migrasi  : ya
    Isi      : Notification multi-channel (mail, database, dll).

21. moe/laravel-task
    Repo     : MOE-Laravel-Task
    Provider : Moe\Task\TaskServiceProvider
    Tag      : moe-task-config
    Migrasi  : ya
    Isi      : Task & assignment management.

22. moe/laravel-template
    Repo     : MOE-Laravel-Template
    Provider : Moe\Template\TemplateServiceProvider
    Tag      : moe-template-config
    Migrasi  : ya
    Isi      : Template management (email, dokumen, dll).

23. moe/laravel-multi-tenant
    Repo     : MOE-Laravel-MultiTenant
    Provider : Moe\MultiTenant\MultiTenantServiceProvider
    Tag      : moe-multitenant-config, moe-multitenant-migrations
    Migrasi  : ya
    Isi      : Multi-tenancy (tenant scoping, resolution).

═══════════════════════════════════════════════════════════════
URUTAN INTEGRASI YANG DISARANKAN
═══════════════════════════════════════════════════════════════

1.  moe/laravel-core            (foundation, wajib pertama)
2.  moe/laravel-auth            (auth dibutuhkan banyak module)
3.  moe/laravel-settings        (pengaturan + audit log)
4.  moe/laravel-profiles
5.  moe/laravel-foundation      (meta-package, setelah 1-4)
6.  moe/laravel-image-pipeline
7.  moe/laravel-identifiers
8.  moe/laravel-commerce
9.  moe/laravel-inventory       (setelah commerce)
10. moe/laravel-shipping        (setelah commerce + inventory)
11. moe/laravel-finance         (setelah commerce)
12. moe/laravel-payment         (setelah finance)
13. moe/laravel-invoice         (setelah commerce + finance)
14. moe/laravel-vendor-b2b      (setelah commerce + finance)
15. moe/laravel-subscription    (setelah payment)
16. moe/laravel-crm
17. moe/laravel-marketing       (setelah auth + commerce)
18. moe/laravel-hrm
19. moe/laravel-content-workflow
20. moe/laravel-notify
21. moe/laravel-task
22. moe/laravel-template
23. moe/laravel-multi-tenant

(Beberapa package saling depend — baca composer.json tiap package kalau install gagal.)

═══════════════════════════════════════════════════════════════
DESIGN TOKENS (Tailwind v4 + Material Design 3)
═══════════════════════════════════════════════════════════════

Kalau project pakai Tailwind v4, buat custom theme di resources/css/app.css sendiri sesuai branding project masing-masing.

Contoh struktur:
@import "tailwindcss";
@source "../../vendor/moe/**/resources/views";

@theme {
    --color-primary: #...;
    --color-on-primary: #...;
    --color-surface: #...;
    --font-family-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
    --text-body: 14px;
    --radius-md: 8px;
}

ATURAN: Tiap app punya UI/branding sendiri. Jangan copy design token dari project lain.

═══════════════════════════════════════════════════════════════
ATURAN PENTING
═══════════════════════════════════════════════════════════════

1. Namespace SELALU Moe\ (PascalCase). Jangan pernah pakai MOE\ — Linux filesystem case-sensitive dan akan gagal autoload.
2. JANGAN tambah dependency spesifik project ke package MOE — package harus reusable.
3. Custom components → buat di host app, render via TYPE_LIVEWIRE_COMPONENT (settings).
4. Event SettingsSaved → dispatch oleh package, didengar oleh host app.
5. Config bisa di-publish & di-custom.
6. View package bisa di-publish & di-custom (tag per package, lihat daftar di atas).
```
