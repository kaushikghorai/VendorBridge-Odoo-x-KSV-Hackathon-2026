# NOTION CRM - Comprehensive Material Request Workflow Guide

This document provides a detailed, end-to-end explanation of the material request lifecycle within the NOTION CRM system. It is designed to clarify the roles, concepts, and precise status transitions that occur across the Site Engineer, Site Manager, and Purchase Officer dashboards.

## 1. Key Roles & Responsibilities

*   **Site Engineer**: Initiates the material request from the construction site. Responsible for defining the requirement, quantities, and ultimately confirming the physical receipt of the materials.
*   **Site Manager**: Acts as the approver. Reviews requests for validity, selects vendors based on cost comparisons, and authorizes the final purchase.
*   **Purchase Officer (POfficer)**: Handles the procurement logistics. Sources quotes from vendors, conducts cost comparisons, and generates formal Purchase Orders.

## 2. Core Concepts

*   **Material Request (MR)**: The initial requisition document created by the Site Engineer detailing what materials are needed at the site.
*   **Cost Comparison (CC)**: A critical procurement step where the Purchase Officer gathers quotations from multiple vendors for the requested materials. This ensures competitive pricing.
*   **Purchase Order (PO)**: The legally binding document issued to the selected vendor, confirming the order details, quantities, and agreed-upon prices.

## 3. Detailed Status Glossary

Every material request moves through specific states. The following table maps the technical Status ID to the UI Label seen by users, its exact meaning, and the role responsible for moving the request out of this state.

| Status ID | UI Label | Meaning | Actioned By |
| :--- | :--- | :--- | :--- |
| **`draft`** | Draft | Request is being written by the Site Engineer but has not yet been submitted for approval. | Site Engineer |
| **`pending`** | Pending Approval | Request is formally submitted and awaits review and approval by the Site Manager. | Manager |
| **`recheck`** | Recheck | The Manager has identified an issue and sent the request back to the Site Engineer for modification or clarification. | Site Engineer |
| **`sign_pending`** | Sign Pending | (Optional workflow) Awaiting a secondary managerial signature before it can proceed to procurement. | Manager |
| **`ready_for_cc`** | Ready for CC | Manager has approved the request. It now sits with the Purchase Officer to gather vendor quotes (Cost Comparison). | Purchase Officer |
| **`cc_pending`** | CC Pending | Purchase Officer has submitted the vendor quotes. Awaiting the Manager to review and select a winning vendor. | Manager |
| **`ready_for_po`** | Ready for PO | A vendor has been selected. The Purchase Officer must now draft and issue the formal Purchase Order. | Purchase Officer |
| **`pending_po`** | Pending PO | The PO has been created and sent to the vendor. The system is waiting for the vendor to confirm and prepare the items. | Purchase Officer |
| **`ready_for_delivery`** | **Ready for Delivery** | The vendor has the items ready, or the items are available in the central warehouse, waiting to be dispatched. | Purchase Officer |
| **`delivery_processing`** | **Out for Delivery** | The materials have physically left the warehouse/vendor and are in transit to the site. | Viewable by All |
| **`delivered`** | **Delivered** | The Site Engineer has physically received the items at the site and confirmed delivery in the system. | Site Engineer |
| **`rejected`** | Rejected | The Manager has completely denied the material request. Terminal state. | Manager |
| **`cc_rejected`** | CC Rejected | The Manager rejected the provided quotes. The Purchase Officer must source new quotes. | Purchase Officer |

---

## 4. Workflows

### A. The Standard Workflow (The Complete Procure-to-Pay Path)

This is the most common path for new, significant material purchases.

1.  **Creation**: Site Engineer creates a new requirement.
    *   *State Transition*: `draft` -> **`pending`**
2.  **Managerial Review**: Manager checks the request against budget and requirements.
    *   *Action*: Approves the request.
    *   *State Transition*: `pending` -> **`ready_for_cc`**
3.  **Cost Comparison (CC)**: Purchase Officer receives the approved request, contacts vendors, and enters quotes into the system.
    *   *Action*: Submits quotes for review.
    *   *State Transition*: `ready_for_cc` -> **`cc_pending`**
4.  **Vendor Selection**: Manager reviews the CC submitted by the Purchase Officer and chooses the best vendor.
    *   *Action*: Approves specific quote.
    *   *State Transition*: `cc_pending` -> **`ready_for_po`**
5.  **PO Generation**: Purchase Officer generates the official PO based on the approved quote and sends it to the vendor.
    *   *Action*: Creates PO.
    *   *State Transition*: `ready_for_po` -> **`pending_po`**
6.  **Vendor Fulfillment**: Vendor acknowledges the PO and prepares the materials. Once ready for pickup or dispatch:
    *   *Action*: Purchase Officer marks items as ready.
    *   *State Transition*: `pending_po` -> **`ready_for_delivery`**
7.  **Dispatch & Transit**: The items are loaded onto a truck for site delivery.
    *   *Action*: Logistics/Purchase updates status.
    *   *State Transition*: `ready_for_delivery` -> **`delivery_processing`** (UI: "Out for Delivery")
8.  **Site Receipt**: The truck arrives at the site. The Site Engineer verifies the quantity and quality against the PO.
    *   *Action*: Site Engineer confirms receipt in the app.
    *   *State Transition*: `delivery_processing` -> **`delivered`**

### B. Direct PO Workflow (Bypassing Cost Comparison)

Used for urgent items, low-value purchases, or when prices are pre-negotiated/fixed (e.g., standard catalog items).

1.  **Creation & Review**: `draft` -> `pending`
2.  **Manager/PO Bypass**: The Manager (or authorized PO) skips the CC phase directly to PO creation.
3.  *State Transition*: `pending` -> **`ready_for_po`** (or `direct_po`)
4.  **Completion**: Follows the standard flow from Step 5 onwards.

### C. Direct Inventory Delivery (Internal Fulfillment)

Used when the requested items are already available in the company's central warehouse or inventory, requiring no external purchase.

1.  **Creation & Review**: `draft` -> `pending`
2.  **Inventory Check**: Manager or PO verifies stock availability.
3.  *State Transition*: `pending` -> **`ready_for_delivery`**
4.  **Completion**: Bypasses all procurement steps and jumps straight to the delivery phase. Follows standard flow from Step 7 onwards.

---

## 5. System Optimizations & Technical Notes

### Resolving Status Duplication
Historically, there was confusion due to overlapping status codes (e.g., `delivery_stage` vs. `delivery_processing`). We have enforced a strict mapping to eliminate ambiguity:
*   **"Ready for Delivery"** (`ready_for_delivery`): The item exists and is packaged, but has not moved.
*   **"Out for Delivery"** (`delivery_processing`): The item is actively in transit.
*   *Note*: The legacy `delivery_stage` status has been deprecated and is now treated identically to `delivery_processing` to maintain backward compatibility without adding UI clutter.

### Dashboard Visibility Improvements
Previously, dashboards filtered strictly for "High Priority" flags, causing standard requests to appear missing.
*   **Action-Based Sorting**: Dashboards now prioritize by **"Latest Activity"**. Any interaction (status change, note added) bumps the request to the top of the queue.
*   **Unified Processing Badges**: To reduce visual noise for the Site Engineer, internal procurement steps (`ready_for_cc`, `ready_for_po`) are grouped under a general "Processing" UI badge.
*   **Distinct Delivery Badges**: "Out for Delivery" and "Delivered" maintain unique, highly visible badges so Site Engineers know exactly when to expect physical arrivals.
