# Istilah Sistem & Ekosistem MOE

**Versi:** 2.0  
**Terakhir diperbarui:** 2026-08-04

---

## Glosarium Istilah Bisnis

| Singkatan | Nama | Fungsi |
|-----------|------|--------|
| **CRM** | Customer Relationship Management | Kelola pelanggan, retensi, promosi |
| **HRM** | Human Resource Management | Karyawan, absensi, gaji |
| **ERP** | Enterprise Resource Planning | Integrasi semua operasi bisnis |
| **SCM** | Supply Chain Management | Rantai pasok, vendor, logistik |
| **WMS** | Warehouse Management | Gudang, stok, picking/packing |
| **POS** | Point of Sale | Kasir, transaksi langsung |
| **CMS** | Content Management | Kelola konten website/app |
| **FMS** | Fleet Management | Kendaraan, pengiriman, kurir |

## Istilah Tambahan

| Istilah | Keterangan |
|---------|------------|
| **Afiliasi** | Program referral marketing |
| **Pencairan/Payout** | Penarikan saldo toko/vendor |
| **Modular Monolith** | Arsitektur monolit dengan package terpisah per domain |
| **Konsinyasi** | Sistem barang titipan vendor di gudang/platform |
| **Attribution** | Pelacakan sumber konversi (afiliasi/iklan) |

---

## Ekosistem Package MOE Laravel

23 package Composer mandiri, namespace `Moe\`, branch `main`, constraint `dev-main`.

| Package | Domain | Fungsi |
|---------|--------|--------|
| `moe/laravel-core` | Fondasi | Konfigurasi inti, helper |
| `moe/laravel-foundation` | Meta | Bundel Core + Settings + Profiles + Auth |
| `moe/laravel-settings` | Konfigurasi | Settings global key-value, typed, cached, encrypted |
| `moe/laravel-profiles` | Data | Profil key-value per entitas (polymorphic) |
| `moe/laravel-auth` | Autentikasi | Login, register, OTP multi-channel, Google OAuth, role |
| `moe/laravel-identifiers` | Data | Generator ID/hashID terstruktur |
| `moe/laravel-content-workflow` | Konten | Status, approval, scheduling, versioning |
| `moe/laravel-image-pipeline` | Media | Upload, optimasi, transformasi gambar |
| `moe/laravel-commerce` | Transaksi | Marketplace stores, produk, order |
| `moe/laravel-inventory` | Stok | Produk, stok, gudang (WMS) |
| `moe/laravel-shipping` | Logistik | Ekspedisi, shipping adapter, tracking |
| `moe/laravel-finance` | Keuangan | Wallet, pembayaran, refund, reconcile |
| `moe/laravel-vendor-b2b` | Supply Chain | Vendor, purchase order, konsinyasi (SCM) |
| `moe/laravel-marketing` | Marketing | Komisi, referral, attribution, affiliate |
| `moe/laravel-hrm` | SDM | Departments, employees, attendance, payroll (HRM) |
| `moe/laravel-crm` | Pelanggan | Contacts, segments, leads, activity (CRM) |
| `moe/laravel-invoice` | Dokumen | Generate invoice PDF |
| `moe/laravel-payment` | Pembayaran | Gateway payment adapter |
| `moe/laravel-subscription` | Langganan | Plan, billing cycle, recurring |
| `moe/laravel-notify` | Notifikasi | Multi-channel: email, WhatsApp, SMS, push |
| `moe/laravel-task` | Operasional | Task management, assignment |
| `moe/laravel-template` | Presentasi | Template engine untuk dokumen/email |
| `moe/laravel-multi-tenant` | Arsitektur | Multi-tenant database/schema |

---

## Pemetaan Istilah → Package

| Istilah Bisnis | Package MOE |
|----------------|-------------|
| WMS | `moe/laravel-inventory` |
| SCM | `moe/laravel-vendor-b2b` + `moe/laravel-shipping` |
| Finance/ERP | `moe/laravel-finance` + `moe/laravel-invoice` + `moe/laravel-payment` |
| CRM | `moe/laravel-crm` + `moe/laravel-marketing` |
| HRM | `moe/laravel-hrm` |
| CMS | `moe/laravel-content-workflow` + `moe/laravel-template` |
| POS | Belum ada package khusus (Commerce + Inventory sebagai dasar) |
| FMS | Belum ada package khusus |
