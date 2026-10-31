# Product Requirements Document (PRD)
## Project: Rafik SmartRetail & Dispatch System
**Target Business:** Rafik Wholesale & Retailers  
**Location:** Kalungi Road, Muyenga, Kampala, Uganda  
**Version:** 1.0.0  
**Status:** Ready for Agentic Build

---

## 1. Executive Summary & Problem Diagnosis
Rafik Wholesale & Retailers operates as a hybrid wholesale and retail store on Kalungi Road, Muyenga. Despite strong daily turnover, the business suffers from three systemic operational leaks:
1. **Unmonitored Cash & Shrinkage:** 90% of transactions are cash, tracked manually in paper notebooks. Returns are unrecorded, allowing unverified inventory exits and cashier skimming.
2. **Delivery Bottleneck:** In-store sales agents (Jax) take phone orders and dispatch heavy groceries to hilly terrain (Muyenga, Bukasa, Kabalagala) via a single push-bicycle ("Kissa"), causing delivery delays, customer attrition, and lost wholesale re-orders.
3. **Customer Churn to Competitors:** Customers switch to nearby competitors (e.g., Rakai Boys) because Rafik lacks customer contact records, purchase histories, and structured repeat-order incentives.
4. **Hardware & Power Fragility:** The store owns an unused PC and suffers from frequent UMEME power cuts. Any solution must be **offline-first**, functional on both low-spec desktop and mobile devices.

---

## 2. Core Personas
- **Jax (Sales Agent / Cashier):** Fast transaction entry, simple product lookup, needs instant phone-order booking, zero technical barrier.
- **Store Owner (Administrator / Auditor):** Audits daily cash drawers against sales, reconciles MoMo accounts, authorizes inventory returns/voids, monitors gross profit and shrinkage.
- **Delivery Rider (Boda/Bicycle):** Needs clear dispatch slips with customer phone, drop-off location (e.g., "Bukasa near railway"), and payment collection status (Paid via MoMo vs. Collect Cash).
- **Wholesale / High-Value Retail Customer:** Wants fast phone/WhatsApp re-ordering, doorstep delivery within 30 minutes, and accurate item pricing.

---

## 3. Scope of the System (3 Core Pillars)

### Module 1: Cash & Stock Shield (POS & Anti-Theft Ledger)
- **Fast Barcode / Search POS:** Optimized for keyboard shortcuts and touch/mouse entry on the existing shop computer.
- **Dual-Pricing Engine:** Instant toggle between Wholesale (bulk cartons/sacks) and Retail (single units).
- **Cashier Shift & Drawer Balancing:** Cashiers start a shift with opening float. At shift end, system prompts for a blind cash count before revealing system totals to eliminate skimming.
- **Strict Void & Return Protocol:** Items cannot be removed from a completed sale or returned without a 4-digit Owner PIN or generated authorization token.
- **Offline-First Synchronization:** Local IndexedDB/SQLite storage allows continuous sales entry during power or internet outages, syncing to the central database once reconnected.

### Module 2: Dispatch & Boda Tracker
- **Fast Phone-Order Intake:** Quick-capture screen to log caller name, phone number, drop-off zone (Muyenga Hill, Bukasa, Kabalagala, Kisugu), items, and payment mode.
- **Rider Allocation:** Assign order to bicycle (Kissa) for local short-range orders, or on-demand Boda riders for heavy/distant trips.
- **Trip Status Machine:** `PENDING_PACK` -> `DISPATCHED` -> `DELIVERED` / `FAILED_RETURNED`.
- **Digital Dispatch Slip:** Printable thermal receipt or WhatsApp dispatch shareable text with order summary and cash collection instructions.

### Module 3: Customer Retention & Auto-CRM
- **Automatic Contact Capture:** Phone number entered during phone-order or checkout automatically builds a customer profile.
- **Wholesale Re-Order Cadence:** Identifies wholesale buyers whose purchase cycle is overdue (e.g., bakeries or small shops that buy flour every 7 days) and flags them for follow-up.
- **Top Customer Tiering:** Flags VIP buyers at checkout so Jax can offer personalized service.

---

## 4. Key Performance Indicators (KPIs)
- **Shrinkage & Cash Variance:** Reduce unrecorded cash/stock variance to < 1.0% of total sales.
- **Order-to-Delivery Time:** Reduce average delivery cycle from 75+ mins to < 35 mins across Muyenga/Bukasa.
- **Repeat Order Rate:** 25% increase in repeat orders from wholesale customers within 60 days.
- **System Uptime:** 100% checkout uptime during UMEME power outages via local offline storage.

---

## 5. Non-Functional & Kampala-Specific Constraints
- **Low Hardware Footprint:** Must run smoothly on 4GB RAM Windows/Linux machine in Chrome/Edge, or mobile browser.
- **Bandwidth Optimization:** Payload sizes must be minimal to handle throttled 3G/4G connections.
- **Currency Format:** All pricing strictly in Uganda Shillings (`UGX`), integer-based (no decimals).