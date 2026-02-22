# 📊 Workflow & Use Case — Billing Mitracom

## 1. Use Case Diagram

### 1.1 Aktor Sistem

```mermaid
graph LR
    SA["👑 Super Admin"]
    AD["🧑‍💼 Admin"]
    FN["💰 Finance"]
    TK["🔧 Teknisi"]
    NOC["📡 NOC"]
    CS["👤 Customer"]
    SYS["⚙️ System/Scheduler"]

    SA --> AD
    AD --> FN
    AD --> TK
    AD --> NOC

    style SA fill:#7c3aed,color:#fff
    style AD fill:#2563eb,color:#fff
    style FN fill:#059669,color:#fff
    style TK fill:#d97706,color:#fff
    style NOC fill:#dc2626,color:#fff
    style CS fill:#0891b2,color:#fff
    style SYS fill:#6b7280,color:#fff
```

| Aktor | Deskripsi | Akses |
|-------|-----------|-------|
| **Super Admin** | Full control | Semua fitur + user management |
| **Admin** | Operasional harian | Semua kecuali user management |
| **Finance** | Keuangan | Dashboard, billing, invoice, transaksi |
| **Teknisi** | Lapangan | Dashboard, pelanggan, tiket, WO, jaringan |
| **NOC** | Network Operations | Dashboard, pelanggan, jaringan, GenieACS |
| **Customer** | Pelanggan (portal) | Portal self-service + mobile API |
| **System** | Scheduler/Cron | Auto-generate invoice, reminder, suspend |

---

### 1.2 Use Case — Admin Panel

```mermaid
graph TB
    subgraph "👥 Manajemen Pelanggan"
        UC1["Tambah Pelanggan"]
        UC2["Edit Pelanggan"]
        UC3["Hapus Pelanggan"]
        UC4["Lihat Peta Pelanggan"]
        UC5["Assign Paket/NAS/ODP"]
    end

    subgraph "💰 Billing & Invoice"
        UC6["Buat Invoice Manual"]
        UC7["Generate Invoice Otomatis"]
        UC8["Mark as Paid"]
        UC9["Cetak Invoice PDF"]
        UC10["Kirim Notifikasi Tagihan"]
        UC11["Hapus Invoice"]
        UC12["Lihat Riwayat Transaksi"]
        UC13["Laporan Keuangan"]
        UC14["Laporan Cashflow"]
    end

    subgraph "📡 Manajemen Jaringan"
        UC15["Kelola NAS"]
        UC16["Kelola OLT"]
        UC17["Kelola ONU"]
        UC18["Kelola ODP"]
        UC19["GenieACS: Kelola CPE"]
        UC20["GenieACS: Reboot/Reset"]
        UC21["GenieACS: Set WiFi"]
    end

    subgraph "🎫 Support & Work Order"
        UC22["Buat Tiket"]
        UC23["Assign Tiket"]
        UC24["Reply Tiket"]
        UC25["Buat Work Order"]
        UC26["Update Status WO"]
        UC27["Tambah Log WO"]
        UC28["Gunakan Item Inventaris"]
    end

    subgraph "⚙️ Sistem & Settings"
        UC29["Kelola User & Role"]
        UC30["Atur Permissions"]
        UC31["Konfigurasi System"]
        UC32["Broadcast Pesan"]
        UC33["Generate Voucher"]
        UC34["Kelola Inventaris"]
    end
```

### 1.3 Use Case — Customer Portal

```mermaid
graph TB
    CS["👤 Customer"]

    CS --> P1["Login Portal"]
    CS --> P2["Lihat Dashboard"]
    CS --> P3["Lihat Invoice"]
    CS --> P4["Cetak Kwitansi"]
    CS --> P5["Bayar Online"]
    CS --> P6["Buat Tiket Support"]
    CS --> P7["Kelola Device WiFi"]
    CS --> P8["Speed Test"]
    CS --> P9["Request Upgrade Paket"]
    CS --> P10["Ubah Password"]
    CS --> P11["Lihat Profil"]

    P7 --> P7a["Ubah SSID"]
    P7 --> P7b["Ubah Password WiFi"]
    P7 --> P7c["Reboot Device"]

    style CS fill:#0891b2,color:#fff
```

### 1.4 Use Case — System Automation

```mermaid
graph TB
    SYS["⚙️ System Scheduler"]

    SYS --> S1["Generate Invoice Bulanan\n⏰ 02:00 harian"]
    SYS --> S2["Kirim Reminder Tagihan\n⏰ 08:00 harian"]
    SYS --> S3["Auto-Suspend Overdue\n⏰ 03:00 harian"]
    SYS --> S4["Process Queue Jobs"]

    S1 --> S1a["Cek pelanggan aktif"]
    S1 --> S1b["Hitung tagihan + pajak"]
    S1 --> S1c["Buat invoice"]
    S1 --> S1d["Kirim notifikasi"]

    S3 --> S3a["Cek invoice overdue"]
    S3 --> S3b["Suspend pelanggan"]
    S3 --> S3c["Disconnect RADIUS"]

    style SYS fill:#6b7280,color:#fff
```

---

## 2. Workflow Diagram

### 2.1 Alur Utama — Lifecycle Pelanggan

```mermaid
flowchart TD
    START(("🟢 Mulai")) --> REG["Admin: Daftarkan\nPelanggan Baru"]
    REG --> ASSIGN["Assign Paket,\nNAS, ODP"]
    ASSIGN --> RADIUS["Sync ke RADIUS\n(PPPoE account)"]
    RADIUS --> WO["Buat Work Order\nInstalasi"]
    WO --> TECH["Teknisi: Pasang\ndi Lokasi"]
    TECH --> ACTIVE["Status: Active ✅"]

    ACTIVE --> MONTHLY{"Setiap Bulan"}
    MONTHLY --> INV["System: Generate\nInvoice Otomatis"]
    INV --> NOTIF["Kirim Notifikasi\nWhatsApp / Telegram"]
    NOTIF --> PAY{"Pelanggan\nBayar?"}

    PAY -->|"✅ Bayar"| PAID["Invoice: Paid\nLanjut Bulan Depan"]
    PAID --> MONTHLY

    PAY -->|"❌ Tidak Bayar"| REMIND["System: Kirim\nReminder (H-3)"]
    REMIND --> PAY2{"Bayar\nSebelum Jatuh Tempo?"}
    PAY2 -->|"✅ Ya"| PAID
    PAY2 -->|"❌ Tidak"| OVERDUE["Invoice: Overdue"]
    OVERDUE --> SUSPEND["System: Auto-Suspend\nDisconnect RADIUS"]
    SUSPEND --> CONTACT["Admin: Hubungi\nPelanggan"]
    CONTACT --> RESOLVE{"Pelanggan\nBayar?"}
    RESOLVE -->|"✅ Bayar"| REACTIVATE["Reactivate\nStatus: Active"]
    REACTIVATE --> MONTHLY
    RESOLVE -->|"❌ Tidak"| DEACT["Deactivation\nWork Order"]
    DEACT --> ENDX(("🔴 Selesai"))

    style START fill:#22c55e,color:#fff
    style ACTIVE fill:#22c55e,color:#fff
    style PAID fill:#22c55e,color:#fff
    style SUSPEND fill:#ef4444,color:#fff
    style ENDX fill:#ef4444,color:#fff
```

### 2.2 Alur Pembayaran Online

```mermaid
flowchart TD
    A["Customer: Pilih Invoice"] --> B["Klik Bayar Online"]
    B --> C{"Payment Gateway\nAktif?"}
    C -->|"❌ Tidak"| D["Tampilkan Halaman\nPembayaran Manual"]
    C -->|"✅ Ya"| E["Create Payment\n(Midtrans/Xendit)"]
    E --> F["Redirect ke\nHalaman Pembayaran"]
    F --> G{"Status\nPembayaran?"}
    G -->|"✅ Success"| H["Callback: Update\nTransaction = Success"]
    H --> I["Update Invoice\nStatus = Paid"]
    I --> J["Kirim Notifikasi\nPembayaran Diterima"]
    G -->|"❌ Failed"| K["Transaction = Failed\nInvoice tetap Unpaid"]
    G -->|"⏳ Pending"| L["Transaction = Pending\nTunggu Callback"]
    L --> G

    style H fill:#22c55e,color:#fff
    style I fill:#22c55e,color:#fff
    style K fill:#ef4444,color:#fff
```

### 2.3 Alur Work Order

```mermaid
flowchart TD
    A["Buat Work Order"] --> B["Status: Pending"]
    B --> C["Assign Teknisi"]
    C --> D["Teknisi: Mulai Kerja"]
    D --> E["Status: In Progress"]

    E --> F["Log: Tiba di Lokasi"]
    F --> G["Kerjakan Pekerjaan"]
    G --> H{"Butuh\nBarang?"}
    H -->|"Ya"| I["Ambil dari Inventaris\n(Stock Out)"]
    I --> G
    H -->|"Tidak"| J["Log: Masalah Teratasi"]
    J --> K["Upload Dokumentasi"]
    K --> L["Status: Completed ✅"]

    E --> M{"Ada\nMasalah?"}
    M -->|"Ya"| N["Log: Issue Found"]
    N --> G
    M -->|"Batal"| O["Status: Cancelled ❌"]

    style B fill:#f59e0b,color:#000
    style E fill:#3b82f6,color:#fff
    style L fill:#22c55e,color:#fff
    style O fill:#ef4444,color:#fff
```

### 2.4 Alur Tiket Support

```mermaid
flowchart TD
    A["Customer/Admin:\nBuat Tiket"] --> B["Status: Open"]
    B --> C["Admin: Assign\nke Teknisi"]
    C --> D["Status: In Progress"]
    D --> E["Teknisi/Admin:\nTambah Komentar"]
    E --> F{"Butuh\nWork Order?"}
    F -->|"Ya"| G["Buat WO dari Tiket"]
    G --> H["WO Selesai"]
    H --> I["Status: Resolved"]
    F -->|"Tidak"| I
    I --> J{"Customer\nPuas?"}
    J -->|"✅ Ya"| K["Status: Closed ✅"]
    J -->|"❌ Tidak"| D

    style B fill:#ef4444,color:#fff
    style D fill:#f59e0b,color:#000
    style I fill:#3b82f6,color:#fff
    style K fill:#22c55e,color:#fff
```

### 2.5 Alur Inventaris

```mermaid
flowchart LR
    A["Tambah Item Baru"] --> B["Stock: 100"]
    B --> C{"Operasi?"}
    C -->|"Stock In"| D["Vendor Kirim Barang\n+50 units"]
    D --> E["Stock: 150"]
    C -->|"Stock Out"| F["Digunakan di WO\n-3 units"]
    F --> G["Stock: 147"]

    G --> H{"Stock ≤ Min?"}
    H -->|"Ya"| I["⚠️ Low Stock Alert\nTampil di Dashboard"]
    H -->|"Tidak"| J["✅ Normal"]

    G --> K{"Stock = 0?"}
    K -->|"Ya"| L["🔴 Out of Stock\nTampil di Dashboard"]
```

---

## 3. Arsitektur Sistem

```mermaid
flowchart TB
    subgraph CLIENT["🖥️ Client"]
        WEB["Web Browser\n(Admin Panel)"]
        PORTAL["Web Browser\n(Customer Portal)"]
        MOBILE["Mobile App\n(API Client)"]
    end

    subgraph APP["🚀 Laravel Application"]
        direction TB
        MW["Middleware\n(Auth, Role, License)"]
        CTRL["Controllers\n(Web + API)"]
        SRV["Services\n(Business Logic)"]
        MDL["Models\n(Eloquent ORM)"]
    end

    subgraph EXTERNAL["🔌 External Services"]
        direction TB
        RADIUS["FreeRADIUS\n(PPPoE Auth)"]
        GACS["GenieACS\n(TR-069 CPE)"]
        WA["WhatsApp API\n(Notifikasi)"]
        TG["Telegram Bot\n(Notifikasi)"]
        PG["Payment Gateway\n(Midtrans/Xendit)"]
    end

    subgraph DATA["💾 Data Layer"]
        MYSQL["MySQL Database"]
        STORE["File Storage\n(Uploads, PDF)"]
        QUEUE["Queue\n(Database Driver)"]
    end

    WEB --> MW
    PORTAL --> MW
    MOBILE --> MW
    MW --> CTRL
    CTRL --> SRV
    SRV --> MDL
    MDL --> MYSQL
    SRV --> RADIUS
    SRV --> GACS
    SRV --> WA
    SRV --> TG
    SRV --> PG
    CTRL --> STORE
    CTRL --> QUEUE

    style CLIENT fill:#dbeafe,color:#000
    style APP fill:#fef3c7,color:#000
    style EXTERNAL fill:#fce7f3,color:#000
    style DATA fill:#d1fae5,color:#000
```

---

## 4. Matriks Hak Akses

```mermaid
graph LR
    subgraph "Modul"
        M1["Dashboard"]
        M2["Customers"]
        M3["Billing"]
        M4["Network"]
        M5["Support"]
        M6["Inventory"]
        M7["Broadcast"]
        M8["Settings"]
        M9["Users"]
    end
```

| Modul | Super Admin | Admin | Finance | Teknisi | NOC |
|-------|:-----------:|:-----:|:-------:|:-------:|:---:|
| Dashboard | ✅ | ✅ | ✅ | ✅ | ✅ |
| Customers | ✅ | ✅ | 👁️ | ❌ | ✅ |
| Plans | ✅ | ✅ | 👁️ | ❌ | ✅ |
| Invoices | ✅ | ✅ | ✅ | ❌ | ❌ |
| Transactions | ✅ | ✅ | ✅ | ❌ | ❌ |
| Reports | ✅ | ✅ | ✅ | ❌ | ❌ |
| NAS/OLT/ONU/ODP | ✅ | ✅ | ❌ | ✅ | ✅ |
| GenieACS | ✅ | ✅ | ❌ | ✅ | ✅ |
| Tickets | ✅ | ✅ | ❌ | ✅ | ✅ |
| Work Orders | ✅ | ✅ | ❌ | ✅ | ✅ |
| Inventory | ✅ | ✅ | ❌ | ❌ | ❌ |
| Broadcast | ✅ | ✅ | ❌ | ❌ | ✅ |
| Vouchers | ✅ | ✅ | ❌ | ❌ | ✅ |
| Settings | ✅ | ✅ | ❌ | ❌ | ❌ |
| Users & Roles | ✅ | ❌ | ❌ | ❌ | ❌ |

> 👁️ = View only, ✅ = Full access, ❌ = No access

---

## 5. Entitas Data (ERD Simplified)

```mermaid
erDiagram
    CUSTOMER ||--o{ INVOICE : has
    CUSTOMER ||--o{ TICKET : creates
    CUSTOMER }o--|| PLAN : subscribes
    CUSTOMER }o--o| NAS : connects
    CUSTOMER }o--o| ODP : located_at
    CUSTOMER ||--o{ ONU : owns

    INVOICE ||--o{ TRANSACTION : payments
    INVOICE }o--|| CUSTOMER : belongs_to

    TICKET ||--o{ TICKET_COMMENT : has
    TICKET }o--o| WORK_ORDER : links_to

    WORK_ORDER ||--o{ WO_LOG : timeline
    WORK_ORDER }o--o{ USER : assigned_to
    WORK_ORDER }o--o{ INVENTORY_ITEM : uses

    INVENTORY_ITEM ||--o{ STOCK_HISTORY : tracks

    USER }o--|| ROLE_PERMISSION : has_role

    NAS ||--o{ CUSTOMER : serves
    OLT ||--o{ ONU : manages
    ODP ||--o{ CUSTOMER : distributes

    PLAN ||--o{ CUSTOMER : used_by
    PLAN ||--o{ HOTSPOT_VOUCHER : generates

    BROADCAST_MESSAGE ||--o{ CUSTOMER : sent_to
    COMPANY_SETTING ||--|| COMPANY_SETTING : singleton
```
