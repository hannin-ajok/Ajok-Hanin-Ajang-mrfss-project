# Product Requirements Document (PRD) & Technical Architecture
## Project: Rafik SmartRetail & Dispatch System
**Target Business:** Rafik Wholesale & Retailers  
**Location:** Kalungi Road, Muyenga, Kampala, Uganda  
**Version:** 2.0.0  
**Target Platform:** Progressive Web App (Offline-First Desktop POS + Mobile Dispatch & WhatsApp Catalog).

---

## 1. Executive Summary & Problem Diagnosis

Rafik Wholesale & Retailers operates as a high-turnover hybrid wholesale and retail store on Kalungi Road in Muyenga, Kampala. The store serves a bifurcated customer base: affluent residential households on Muyenga Hill (Tank Hill Road area) and price-sensitive retail and commercial buyers (local duukas, rolex stalls, roadside cafes) down toward Bukasa and Kabalagala.

Despite steady daily sales, the business faces five systemic operational vulnerabilities:
1. **Unmonitored Cash & Shrinkage:** 90% of transactions are in cash, logged manually in physical paper books. Returns, exchanges, and voids are untracked, creating opportunities for cashier skimming and inventory loss.
2. **Topographical Delivery Bottleneck:** Dispatches rely on a single push-bicycle ("Kissa"). Pushing heavy staples (25kg maize flour bags, crates of beer, 20L cooking oil) up the steep gradients of Muyenga Hill causes severe delays (>75 minutes), order cancellations, and physical burnout.
3. **Aggressive Local Competition:** Competitors such as Rakai Boys dominate on bulk staple pricing. Rafik loses repeat business because it lacks customer contact records, purchase histories, and structured re-ordering incentives for high-volume wholesale buyers.
4. **Unrecorded "Katala" (Informal Credit) & Asset Bleed:** Informal store credit (*katala*) extended to trusted local buyers is tracked on scrap paper, leading to uncollected bad debts. Returnable assets (empty beer/soda glass crates, LPG cooking gas cylinders) are loaned out without a structured deposit ledger, creating asset shrinkage.
5. **Infrastructure Instability:** The shop possesses an underutilized 4GB RAM desktop PC subject to regular UMEME electricity outages and throttled 3G/4G network connectivity. Any software solution must be **strictly offline-first**.

---

## 2. Core Personas & Problem Scenarios
+----------------------------------------------------------------------------------------------------+
| OPERATIONAL PERSONAS |
+--------------------------+----------------------------+--------------------------------------------+
| Jax (Cashier / Dispatch) | Store Owner (Auditor) | Musa (Contracted Stage Boda Rider) |
| Focus: Fast barcode keys,| Focus: Shift drawer audit, | Focus: Clear drop-off directions, delivery |
| phone orders, quick MoMo | gross margins, shrinkage | cash collection, fuel fee transparency, |
| lookup, zero manual math.| tracking, katala limits. | returnable crate collection. |
+--------------------------+----------------------------+--------------------------------------------+
code
Code
### Critical Edge Cases
* **The Sudden UMEME Blackout:** Power drops mid-transaction. The POS continues running locally in the browser via ServiceWorker and IndexedDB, buffering transactions without dropping the customer queue.
* **The Split Payment:** A customer pays UGX 50,000 in cash and UGX 135,000 via MTN Mobile Money. The transaction is reconciled atomically under one receipt.
* **The Fake MoMo SMS Scam:** A customer presents a forged SMS confirmation. Jax verifies the incoming transaction code against the store's verified MoMo API/merchant callback screen before releasing goods.
* **The Broken Bulk Request:** A walk-in customer requests 2kg of sugar from a 50kg wholesale sack. The inventory system decrements the parent sack and increments broken bulk units without discrepancies.

---

## 3. The 6 Strategic Differentiators for Rafik

1. **Omnichannel WhatsApp Commerce (Zero-Friction Ordering):**  
   Rather than forcing customers to install a mobile app or complete an account-gated web checkout, the public web catalog compiles orders into an encrypted cart link that generates a pre-structured WhatsApp order message directly to the shop’s phone.
2. **Topography-Aware Hybrid Dispatch Engine:**  
   Orders are algorithmically classified by weight and terrain. Flat, light deliveries (<5kg to Kalungi Flat) remain with Kissa on the bicycle. Heavy or steep deliveries (Tank Hill, Bukasa murram roads) auto-route to contracted Kalungi Stage Boda riders with transparent delivery fees.
3. **The Digital "Katala" (Informal Store Credit) Ledger:**  
   Replaces paper notebooks with digital credit limits, balance aging, and automated weekly WhatsApp/SMS balance statements.
4. **Returnable Container & Asset Tracking:**  
   Maintains real-time bottle, crate, and LPG gas cylinder balances per customer to prevent working capital drain.
5. **Anti-Skimming Blind Shift Balancing:**  
   Cashiers start their shifts with a recorded opening float. At closing, the system prompts for a blind count of cash and mobile money balances before revealing expected system figures. Discrepancies require an Owner PIN to authorize drawer close.
6. **Dynamic Bulk De-Kitting (Carton-to-Piece Conversion):**  
   Automatically manages relationships between parent wholesale packaging (e.g., carton of Mukwano soap with 24 bars) and child retail units, preventing discrepancies between wholesale and retail stock counts.

---

## 4. System Architecture & Technical Specifications
code
Code
CLIENT TIER
+-------------------------------------------------------------------+
| Progressive Web App (PWA) / Next.js 15 |
| +---------------------------+ +------------------------------+ |
| | Cashier & Dispatch Screen | | Public WhatsApp Catalog | |
| | (Keyboard shortcuts) | | (Zero-login, static/cached) | |
| +-------------+-------------+ +--------------+---------------+ |
| | | |
| Local Dexie.js (IndexedDB) <------- ServiceWorker |
+----------------+-------------------------------+------------------+
| (Background Sync Engine) | (Direct WhatsApp)
v v
+-------------------------------------------------------------------+
| BACKEND API TIER (Next.js App Router / Node.js) |
| - Transaction Coordinator - Bulk De-kitting Logic |
| - Topography Dispatch Matrix - Blind Reconciliation Engine |
| - Katala Debt Ledger - Returnable Assets Controller |
+--------------------------------+----------------------------------+
|
v
+-------------------------------------------------------------------+
| PERSISTENCE TIER (PostgreSQL / Supabase) |
| Row-level security, immutable transaction logs, integer UGX |
+-------------------------------------------------------------------+
code
Code
### Technical Stack Justification
* **Application Framework:** Next.js 15 (App Router, React 19, Tailwind CSS).
* **Local Persistence:** Dexie.js (IndexedDB wrapper) for instant client-side lookups across 3,000+ SKUs and full offline resilience.
* **Production Database:** Managed PostgreSQL with Drizzle ORM for type-safe schema maintenance and migrations.
* **Hardware Printing:** Native Web Serial API and Web Bluetooth API communicating directly in ESC/POS command bytes to 58mm/80mm thermal receipt printers without vendor driver bloat.
* **Notifications & Messaging:** Africa's Talking SMS API for local payment reminders and system SMS notifications.
* **Currency Formatting:** All transactions, prices, and ledgers use non-decimal integer representations in Uganda Shillings (`UGX`).
Order Intake (Weight, Volume, Drop Zone)
                                         |
               +-------------------------+-------------------------+
               |                                                   |
        Weight <= 5kg AND                                   Weight > 5kg OR
        Zone == Kalungi Road / Flat Bukasa                  Zone == Muyenga Hill / Bukasa Murram
               |                                                   |
               v                                                   v
     [BICYCLE DISPATCH - KISSA]                          [BODA DISPATCH - KALUNGI STAGE]
     - Base Fee: UGX 1,500 - 2,500                       - Base Fee: UGX 3,500 - 6,000
     - Internal rider incentive logged                   - Direct cash disbursement to stage rider
     - Transit time: 10 - 20 mins                        - Transit time: 10 - 15 mins

       ### Digital Dispatch Slip Format (Thermal Receipt / WhatsApp Text)
```text
========================================
RAFIK SMART RETAIL | DISPATCH SLIP
ORDER NO    : #RF-2026-1082
DATE/TIME   : 02-Nov-2026 14:15 EAT
CASHIER     : JAX
----------------------------------------
CUSTOMER    : Mama Kevin
PHONE       : +256 772 123 456
DROP ZONE   : Bukasa, Below Railway (Near Water Point)
DIRECTIONS  : Yellow gate after slope
----------------------------------------
ORDER ITEMS:
  1x Maganjo Maize Flour 25kg (Wholesale)
  2x Fortune Buto Cooking Oil 3L
  1x Crate Bell Lager [1 EMPTY CRATE COLLECTED]
----------------------------------------
PAYMENT SUMMARY:
  Merchandise Total : UGX 185,000
  Delivery Fee      : UGX   4,000
  Bottle Deposit    : UGX       0 (Swap Verified)
  --------------------------------------
  TOTAL TO COLLECT  : UGX 189,000
  PAYMENT MODE      : CASH ON DELIVERY
----------------------------------------
DISPATCH METHOD     : BODA (Musa - Kalungi Stage)
RIDER PHONE         : +256 701 987 654
========================================

-- Enums for constrained application state
CREATE TYPE unit_type AS ENUM ('PIECE', 'CARTON', 'SACK', 'CRATE', 'KG', 'LITRE');
CREATE TYPE payment_channel AS ENUM ('CASH', 'MTN_MOMO', 'AIRTEL_MONEY', 'KATALA_CREDIT', 'SPLIT');
CREATE TYPE order_status AS ENUM ('PENDING_PACK', 'PACKED', 'DISPATCHED', 'DELIVERED', 'RETURNED_CANCELLED');
CREATE TYPE dispatch_mode AS ENUM ('BICYCLE_KISSA', 'STAGE_BODA', 'IN_STORE_PICKUP');
CREATE TYPE shift_status AS ENUM ('OPEN', 'BALANCED', 'DISCREPANCY_FLAGGED');

-- Customers and Katala credit limits
CREATE TABLE customers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    full_name VARCHAR(150) NOT NULL,
    phone_number VARCHAR(20) UNIQUE NOT NULL,
    zone VARCHAR(100) NOT NULL,
    specific_directions TEXT,
    is_wholesale BOOLEAN DEFAULT FALSE,
    credit_limit_ugx BIGINT DEFAULT 0,
    current_debt_ugx BIGINT DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Products, pricing tiers, returnable assets, and de-kitting relations
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    barcode VARCHAR(100) UNIQUE,
    product_name VARCHAR(255) NOT NULL,
    category VARCHAR(100) NOT NULL,
    base_unit unit_type NOT NULL,
    retail_price_ugx BIGINT NOT NULL,
    wholesale_price_ugx BIGINT NOT NULL,
    cost_price_ugx BIGINT NOT NULL,
    stock_quantity NUMERIC(10, 2) NOT NULL DEFAULT 0,
    reorder_level NUMERIC(10, 2) NOT NULL DEFAULT 5,
    is_returnable BOOLEAN DEFAULT FALSE,
    returnable_deposit_ugx BIGINT DEFAULT 0,
    parent_product_id UUID REFERENCES products(id),
    conversion_factor NUMERIC(10, 2) DEFAULT 1.0,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Shifts and Blind Drawer Audits
CREATE TABLE shifts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    cashier_name VARCHAR(100) NOT NULL,
    opened_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    closed_at TIMESTAMP WITH TIME ZONE,
    opening_float_ugx BIGINT NOT NULL,
    declared_cash_ugx BIGINT,
    system_expected_cash_ugx BIGINT,
    discrepancy_ugx BIGINT,
    supervisor_override BOOLEAN DEFAULT FALSE,
    status shift_status DEFAULT 'OPEN'
);

-- Orders and fulfillment tracking
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_code SERIAL UNIQUE,
    customer_id UUID REFERENCES customers(id),
    shift_id UUID REFERENCES shifts(id),
    total_amount_ugx BIGINT NOT NULL,
    delivery_fee_ugx BIGINT DEFAULT 0,
    payment_method payment_channel NOT NULL,
    momo_transaction_ref VARCHAR(100),
    order_status order_status DEFAULT 'PENDING_PACK',
    dispatch_mode dispatch_mode NOT NULL,
    rider_name VARCHAR(100),
    rider_phone VARCHAR(20),
    crates_outbound INT DEFAULT 0,
    crates_returned INT DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Line items for every sale
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID REFERENCES products(id),
    quantity_sold NUMERIC(10, 2) NOT NULL,
    unit_price_ugx BIGINT NOT NULL,
    is_wholesale_pricing BOOLEAN DEFAULT FALSE,
    line_total_ugx BIGINT NOT NULL
);

-- Audit ledger for Katala informal credit accounts
CREATE TABLE katala_ledger (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID REFERENCES customers(id),
    order_id UUID REFERENCES orders(id),
    amount_borrowed_ugx BIGINT DEFAULT 0,
    amount_repaid_ugx BIGINT DEFAULT 0,
    balance_after_ugx BIGINT NOT NULL,
    notes TEXT,
    recorded_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

// lib/db/dexie-client.ts
import Dexie, { Table } from 'dexie';

export interface LocalSaleRecord {
  localId?: number;
  orderId: string;
  items: Array<{
    productId: string;
    productName: string;
    qty: number;
    unitPriceUgx: number;
    isWholesale: boolean;
  }>;
  totalUgx: number;
  paymentMethod: 'CASH' | 'MTN_MOMO' | 'AIRTEL_MONEY' | 'KATALA_CREDIT' | 'SPLIT';
  shiftId: string;
  isSynced: boolean;
  createdAt: number;
}

export class RafikLocalDatabase extends Dexie {
  salesTable!: Table<LocalSaleRecord, number>;

  constructor() {
    super('RafikRetailLocalDB');
    this.version(1).stores({
      salesTable: '++localId, orderId, isSynced, createdAt, shiftId'
    });
  }
}

export const localDb = new RafikLocalDatabase();
// lib/sync/sync-runner.ts
import { localDb, LocalSaleRecord } from '../db/dexie-client';

export async function processOfflineQueue(apiEndpoint: string, authToken: string): Promise<{
  syncedCount: number;
  remainingCount: number;
}> {
  const pendingSales = await localDb.salesTable.where({ isSynced: false }).toArray();

  if (pendingSales.length === 0) {
    return { syncedCount: 0, remainingCount: 0 };
  }

  try {
    const response = await fetch(`${apiEndpoint}/api/v1/pos/reconcile-offline`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${authToken}`
      },
      body: JSON.stringify({ transactions: pendingSales })
    });

    if (!response.ok) {
      throw new Error(`Sync failed with HTTP status: ${response.status}`);
    }

    const { processedOrderIds } = (await response.json()) as { processedOrderIds: string[] };

    await localDb.transaction('rw', localDb.salesTable, async () => {
      for (const orderId of processedOrderIds) {
        await localDb.salesTable
          .where({ orderId })
          .modify({ isSynced: true });
      }
    });

    const remaining = await localDb.salesTable.where({ isSynced: false }).count();
    return { syncedCount: processedOrderIds.length, remainingCount: remaining };
  } catch (error) {
    console.warn('Network unavailable. Offline transactions safely stored locally.', error);
    return { syncedCount: 0, remainingCount: pendingSales.length };
  }
}
// lib/hardware/thermal-printer.ts
export async function printReceiptThermal(receiptText: string): Promise<boolean> {
  if (!('serial' in navigator)) {
    console.error('Web Serial API is not supported on this browser version.');
    window.print(); // Fallback to browser print preview
    return false;
  }

  try {
    // Request thermal printer COM port connection
    const port = await (navigator as any).serial.requestPort();
    await port.open({ baudRate: 9600 });

    const encoder = new TextEncoder();
    const writer = port.writable.getWriter();

    // ESC/POS Initialization: ESC @ (0x1B, 0x40)
    const initCmd = new Uint8Array([0x1B, 0x40]);
    // ESC/POS Cut Paper: GS V 65 0 (0x1D, 0x56, 0x41, 0x00)
    const cutCmd = new Uint8Array([0x1D, 0x56, 0x41, 0x00]);

    await writer.write(initCmd);
    await writer.write(encoder.encode(receiptText));
    await writer.write(cutCmd);

    writer.releaseLock();
    await port.close();
    return true;
  } catch (err) {
    console.error('Failed to write raw bytes to thermal printer:', err);
    return false;
  }
}
+---------------------------------------------------------------------------------------------------+
| RAFIK POS v2.0 | Cashier: JAX | Shift: #41 | DB: [LOCAL OK] | Sync: [ALL SAVED] | 14:45 EAT       |
+---------------------------------------------------------------------------------------------------+
| [F2] Lookup Product: [ maganjo maize                       ] [x] BULK WHOLESALE PRICING (F9)      |
+-------------------------------------------------------------------+-------------------------------+
| CART LINE ITEMS                                                   | BILLING SUMMARY               |
| 1. Maganjo Maize Flour 25kg          x 2 @ 85,000 =   UGX 170,000 | Subtotal     : UGX 220,000    |
| 2. Fortune Cooking Oil 3L            x 1 @ 35,000 =   UGX  35,000 | Delivery Fee : UGX   4,000    |
| 3. Soda Glass Crate Deposit          x 1 @ 15,000 =   UGX  15,000 | Deposit      : UGX  15,000    |
|                                                                   | ----------------------------- |
|                                                                   | TOTAL DUE    : UGX 239,000    |
+-------------------------------------------------------------------+-------------------------------+
| [F4] CASH PAID | [F5] MOMO PUSH | [F6] KATALA CREDIT | [F7] SPLIT TENDER | [SPACE] DISPATCH SLIP  |
+---------------------------------------------------------------------------------------------------+

// lib/commerce/whatsapp-cart.ts
export function buildWhatsAppCartUri(
  storePhone: string,
  customerName: string,
  deliveryZone: string,
  items: Array<{ name: string; qty: number; isWholesale: boolean }>,
  paymentPreference: string
): string {
  const itemizedLines = items
    .map(item => `- ${item.qty}x ${item.name} (${item.isWholesale ? 'Wholesale' : 'Retail'})`)
    .join('\n');

  const textPayload = 
`*NEW ORDER - RAFIK RETAIL & WHOLESALE*
----------------------------------------
*Customer Name:* ${customerName}
*Delivery Area:* ${deliveryZone}
*Payment By:* ${paymentPreference}

*Items Requested:*
${itemizedLines}

Please confirm total stock availability, dispatch mode, and total cost.`;

  return `https://wa.me/${storePhone}?text=${encodeURIComponent(textPayload)}`;
}

+---------------------------------------------------------------------------------------------------+
|                                     SPRINT IMPLEMENTATION SCHEDULE                                |
+----------------------------------+----------------------------------+-----------------------------+
| Sprint 1: Core POS & Shield      | Sprint 2: Dispatch & WhatsApp    | Sprint 3: Ledger & Scale    |
| Duration: Weeks 1 - 3            | Duration: Weeks 4 - 6            | Duration: Weeks 7 - 8       |
|                                  |                                  |                             |
| - Offline IndexedDB DB           | - Dispatch State Machine         | - Katala Credit Module      |
| - Keyboard Shortcuts Layout      | - WhatsApp Cart Link Generator   | - Africa's Talking SMS API  |
| - Parent-Child De-kitting        | - Returnable Crate Tracking      | - Blind Shift Drawer Audit  |
| - Web Serial Thermal Printing    | - Boda Stage Pricing Matrix      | - Predictive Re-order CRM   |
+----------------------------------+----------------------------------+-----------------------------+

11. Acceptance & Operational Verification Criteria
Anti-Skimming Drawer Balancing: Physical cash declared at shift close matches system expectations within 
≤
0.5
%
≤0.5%
 across 30 consecutive business days.
Offline Durability: The POS executes sales continuously during a simulated 4-hour power outage on battery/local cache, and reconciles 100% of cached sales without data corruption when reconnected.
Dispatch Speed: Average turnaround from customer phone call to rider departure drops from 
>
75
>75
 minutes to 
<
25
<25
 minutes across Bukasa and Muyenga Hill.
Zero Returnable Asset Loss: All unreturned beer/soda crates and LPG cylinders are tied directly to an active customer deposit balance or Katala ledger liability, reducing unaccounted container shrinkage to zero.

---

## 5. Topography & Vehicle Dispatch Matrix
