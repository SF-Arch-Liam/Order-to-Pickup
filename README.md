# Order-to-Pickup Salesforce Metadata

Salesforce DX source for an LWR Experience Cloud “Order to Pickup” site. Supports multiple brands and stores.

## Personas & Access

- **Customer (guest)**: browse menu, add items, choose store/pickup time, place order; no login.
- **Counter staff (authenticated)**: Customer Community+ users; view/update orders for their store; status flow New → In Progress → Ready → Picked Up.
- **Admin/Dev**: internal license; manages brands, menus, deployments.

## Data Model Highlights

- **BusinessBrand**: unique code `O2P_Brand_Code__c` (external ID), domain slug `O2P_Domain_Slug__c`, active flag `O2P_Active__c`, theming (`O2P_Primary_Color__c`, logo/support fields).
- **Pricebook2 (Menu)**: lookup to BusinessBrand via `O2P_Business_Brand__c` (required); effective dating `O2P_Effective_Start__c` / `O2P_Effective_End__c`.
- **Product2**: display ordering `O2P_Display_Order__c`; priced through `PricebookEntry`.
- **Order**: store lookup `Store__c` (Location); derived brand formula `Brand__c`; customer fields (`Customer_Name__c` required, `Customer_Email__c`, `Customer_Phone__c`, `Special_Instructions__c`); pickup and lifecycle timestamps (`Pickup_Date_Time__c` required, `Placed_At__c`, `In_Progress_At__c`, `Ready_At__c`, `Picked_Up_At__c`); payment fields (`Payment_Provider__c`, `Payment_Status__c`, unique external ID `Payment_Intent_ID__c`).
- **OrderItem**: standard line items tied to Order, Product2, PricebookEntry; no custom fields.

## Key Relationships

- Brand → Menus (BusinessBrand 1–M Pricebook2)
- Brand → Stores (BusinessBrand 1–M Location)
- Menu → Prices (Pricebook2 1–M PricebookEntry)
- Product → Prices (Product2 1–M PricebookEntry)
- Store → Orders (Location 1–M Order)
- Brand → Orders (BusinessBrand 1–M Order via derived `Brand__c`)
- Order → Items (Order 1–M OrderItem)
- OrderItem → Product2 and PricebookEntry

ERD image: `assets/image-58fd0bf0-99a1-4068-bc47-9654d4d8fcd2.png`

## Order Lifecycle & Payments

- Status flow: New → In Progress → Ready → Picked Up (plus optional Canceled).
- Timestamps set per transition (`In_Progress_At__c`, `Ready_At__c`, `Picked_Up_At__c`); `Pickup_Date_Time__c` required at checkout.
- Payments: `Payment_Provider__c`, `Payment_Status__c` (Unpaid, Pending, Authorized, Paid, Partially Refunded, Refunded, Failed, Canceled, Chargeback), unique intent reference `Payment_Intent_ID__c`.

## Repository Layout

- `force-app/main/default/objects/BusinessBrand/**`: brand code/slug/active/theme.
- `force-app/main/default/objects/Pricebook2/**`: brand-linked menus with effective dating.
- `force-app/main/default/objects/Product2/**`: products + display order; standard pricebook entries.
- `force-app/main/default/objects/Order/**`: store/brand linkage, customer contact, pickup timing, payment fields.
- `force-app/main/default/objects/OrderItem/**`: standard order lines.

## Working With This Metadata

1. Authorize org (if needed): `sfdx force:auth:web:login -a <alias>`
2. Deploy: `sfdx force:source:deploy -p force-app -u <alias>`
3. Validate only: `sfdx force:source:deploy -p force-app -u <alias> -c`
4. Pull changes (scratch/org synced): `sfdx force:source:pull -u <alias>`

## Notes for Implementers

- Multi-brand ready: brand is derived on Order from the selected Store; change Store to change brand.
- Guard payment fields for system use; keep intent ID unique for webhook idempotency.
- Pickup datetime is required; lifecycle stamps support SLA/ops reporting.
- Scaling to more brands/sites: you can add additional LWR Experience sites per brand; assign that brand’s Community+ users and map their stores; reuse shared objects (BusinessBrand, Pricebook2, Product2, Order/OrderItem) to avoid duplication while isolating access via site guest user/profile and brand/store-scoped sharing.
