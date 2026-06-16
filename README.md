# 🏛️ DKP SWAG — Cemetery & Crematorium Management System

### System Design Case Study | Surabaya City Government

> **Role:** Software Engineer & System Rescue Specialist
> **Client:** Dinas Kebersihan dan Pertamanan (DKP), Pemerintah Kota Surabaya
> **Duration:** September – December 2019 (4 months)
> **Stack:** PHP · CodeIgniter · MariaDB · Bootstrap · jQuery · TCPDF · XAMPP
> **Environment:** On-premise · Windows Server 2012 · 32GB RAM

---

## 🔥 The Mess — Why This System Had to Be Built

When I was brought in, the Green Open Space Department of Surabaya City Government was in a critical situation. Their cemetery data management system, originally developed in 2015, had accumulated critical bugs over four years of neglect — and the original developer had gone completely unreachable.

Daily operations had ground to a halt. Staff had reverted to managing cemetery records in Excel spreadsheets — a fragile, error-prone process for a department responsible for tracking burial plots, cremation records, and retribution billing across multiple cemetery locations in Surabaya.

The most damaging consequence was financial: **invoice generation for grave lease retribution took over one month to produce manually** — so long that the department had effectively stopped issuing invoices altogether. Revenue was leaking silently, and there was no mechanism to recover it.

The bugs in the original system were not minor. They were severe enough that staff could not use it at all. The system had been abandoned in place, and Excel had become the default — for a government department managing the burial records of an entire city.

---

## 🔍 The Diagnosis — What Was Actually Wrong

After a full assessment of the abandoned codebase and the department's operational workflow, the core problems were clear:

- **Critical application bugs** left by the original developer made the system completely unusable
- **No multi-role access control** — the system had no meaningful separation between administrator, field officer, and official roles
- **No billing engine** — there was no automated mechanism to generate, track, or follow up on grave lease invoices
- **No crematorium module** — an entire operational domain was untracked
- **No audit trail** — there was no log of who changed what, making accountability impossible
- **Data integrity risks** — with multiple field officers working concurrently, there was no mechanism to prevent two officers from assigning the same burial plot simultaneously

The original system was not salvageable as-is. What was needed was not a patch — it was a full modernization of the existing codebase, with new modules built on top of a repaired foundation.

---

## 🏗️ The Architecture — How It Was Solved

The system was rebuilt and extended using **CodeIgniter (PHP)** with **MariaDB** as the database engine, deployed on the department's existing **on-premise Windows Server 2012** hardware (32GB RAM) via XAMPP — keeping infrastructure costs at zero while leveraging what the department already owned.

### Deployment Topology

```
[ Field Officers / Office Staff / Officials ]
             ↓ (LAN / Intranet)
        [ Apache (XAMPP) ]
             ↓
      [ CodeIgniter App ]
             ↓
       [ MariaDB Engine ]
    (dkp_swag — 29 tables)
             ↓
    [ Local File System ]
  (KTP / SKTM / Photos / PDFs)
```

The deployment was intentionally lean. Running on existing on-premise hardware with no cloud dependency meant zero additional monthly infrastructure cost for the department — a critical constraint for a government unit with a fixed operational budget.

With a peak concurrent user load of approximately **30 users** (20 field officers + 10 office staff), the on-premise stack was more than sufficient. Apache connection pooling and MariaDB query optimization kept response times well within acceptable bounds under this load.

### Multi-Role Access Control

One of the first architectural decisions was implementing a proper **role-based access control (RBAC)** layer via a custom `Auth` library:

| Role             | Access Scope                                   |
| ---------------- | ---------------------------------------------- |
| Administrator    | Full system access                             |
| Petugas Lapangan | Data entry, search, daily activity input       |
| Pejabat / Kadis  | Reports, document signing, read-only oversight |
| Ahli Waris       | Grave search, payment status                   |

### Key Modules Delivered

| Module                  | Description                                                                               |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| Almarhum Management     | Full CRUD for deceased records with document uploads (KTP, SKTM, grave photos)            |
| Burial Plot Management  | Block/capacity tracking with real-time availability checks                                |
| Crematorium Module      | Full operational tracking for cremation activities — previously non-existent             |
| Billing Engine          | Automated invoice generation, payment tracking, due date monitoring, and receipt printing |
| SKTM (Fee Waiver)       | Workflow for processing billing exemptions for underprivileged families                   |
| Executive Dashboard     | Revenue charts, activity summaries, and KPI overview for department heads                 |
| Grave Search            | Public-facing search for next-of-kin to locate burial records                             |
| Audit Log               | Full activity trail — every create, update, and delete recorded with user and timestamp  |
| PDF Document Generation | Invoices, receipts, official letters, and reports — all generated in-system via TCPDF    |

---

## ⚠️ Edge Cases — Where Data Integrity Gets Hard

### The Concurrent Plot Assignment Problem

With 20+ field officers potentially entering burial data simultaneously, the most critical integrity risk was **two officers assigning the same burial plot to different deceased records at the same time**.

This was addressed at two layers:

1. **Application layer:** Before any burial record is committed, the system calls `cek_ketersediaan_plot(ID_ISI)` — a dedicated availability check that reads the current `terisi` status of the requested plot. If already occupied, the transaction is rejected before it reaches the database.
2. **Database layer:** The `UPDATE isi_makam SET terisi = 1 WHERE ID_ISI = ? AND terisi = 0` pattern ensures that even if two requests pass the application check simultaneously, only one will produce a successful row update. The second will update zero rows — which the application detects and handles as a conflict.

This two-layer guard (application pre-check + atomic conditional update) eliminates double-booking without requiring explicit table locking, keeping concurrent performance intact.

### The Invoice Paralysis Problem

The department had stopped issuing invoices entirely because manual generation took over a month. The billing engine solved this by:

- Automating invoice calculation based on tariff tables and tenure duration
- Generating print-ready invoice PDFs on demand — from zero to printed invoice in under one minute
- Tracking payment status, partial payments, and outstanding balances in real time
- Automating due date calculation and flagging overdue accounts for follow-up

What previously took a month of manual work now took **less than one minute** — and the department regained the ability to actually collect the retribution revenue it was legally owed.

---

## 📊 The Impact — What Changed

| Before                                      | After                                                 |
| ------------------------------------------- | ----------------------------------------------------- |
| Excel spreadsheets as primary record system | Unified database with 29 normalized tables            |
| Invoice generation: ~1 month manual effort  | Invoice generation: under 1 minute                    |
| No crematorium tracking                     | Full crematorium operational module                   |
| No role separation                          | 4-role RBAC with scoped access                        |
| No audit trail                              | Complete activity log — every action recorded        |
| Revenue leakage from unbilled retribution   | Automated billing with due date tracking              |
| Critical bugs blocking all operations       | Fully operational system delivered in 4 months        |
| Zero concurrent user support                | Stable under 30 concurrent users on existing hardware |

---

## 📐 System Design Diagrams

For the complete system architecture including Use Case Diagrams, Sequence Diagrams, Entity-Relationship Diagram (29 tables), Class Diagram (15 Controllers, 30 Models), Dependency Map, and Data Flow Diagram — see:

📄 **[BLUEPRINT_DKP_SWAG.md](./BLUEPRINT_DKP_SWAG.md)**

---

## 💡 Key Architectural Decisions

**Why CodeIgniter over a modern framework?**
The department's IT staff needed to maintain this system long-term without external support. CodeIgniter's low learning curve, minimal dependencies, and well-documented MVC pattern made it the pragmatic choice — not the fashionable one. A solution the client can maintain is worth more than a solution built on the latest framework.

**Why on-premise over cloud?**
Zero additional cost. The department already owned the hardware. Moving to cloud would have introduced monthly costs with no operational justification given the user load and data sensitivity requirements.

**Why XAMPP?** 
Client-mandated stack. For a strictly offline intranet with ~30 concurrent users, it was the right fit — zero additional infrastructure cost, and the department's IT staff could maintain it without external support.

---

*This case study is part of my system design portfolio. Client name and sensitive operational data have been retained as the project was a public government engagement. All diagrams were produced from the actual system architecture.*
