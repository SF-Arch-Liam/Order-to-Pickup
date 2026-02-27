# Order-to-Pickup Salesforce Metadata

Metadata retrieved from Salesforce for an Order-to-Pickup solution. This repo is a Salesforce DX project; source lives under `force-app/main/default`.

## Data Model Highlights

- **BusinessBrand** (custom object)
  - Brand Code (`O2P_Brand_Code__c`): unique, external ID; uppercase/underscore format.
  - Domain Slug (`O2P_Domain_Slug__c`): unique URL slug for Experience Cloud routing.
  - Active flag (`O2P_Active__c`) plus optional theming fields (`O2P_Primary_Color__c`, logo resource, support email).
- **Menus / Price Books** (`Pricebook2`)
  - Lookup to BusinessBrand (`O2P_Business_Brand__c`, required).
  - Effective dates (`O2P_Effective_Start__c`, `O2P_Effective_End__c`) to control which menu is current.
- **Products** (`Product2`)
  - Standard product fields plus menu sort control via `O2P_Display_Order__c`.
  - Priced through standard `PricebookEntry`.
- **Orders** (standard `Order`)
  - Store lookup (`Store__c` to `Location`) with derived Brand formula (`Brand__c` from the store’s brand).
  - Customer contact fields (`Customer_Name__c` required, `Customer_Email__c`, `Customer_Phone__c`) and free-form `Special_Instructions__c`.
  - Pickup scheduling and lifecycle timestamps: `Pickup_Date_Time__c` (required), `Placed_At__c`, `In_Progress_At__c`, `Ready_At__c`, `Picked_Up_At__c`.
  - Payment integration: provider picklist (`Payment_Provider__c`), transaction reference external ID (`Payment_Intent_ID__c`, unique), and payment state picklist (`Payment_Status__c` values: Unpaid, Pending, Authorized, Paid, Partially Refunded, Refunded, Failed, Canceled, Chargeback).
- **Order Line Items** (`OrderItem`)
  - Standard pricing/quantity fields; no custom fields beyond Salesforce defaults.

## Repository Layout

- `force-app/main/default/objects/BusinessBrand/**`: brand model fields (code, slug, active, theme).
- `force-app/main/default/objects/Pricebook2/**`: brand-linked menus with effective dating.
- `force-app/main/default/objects/Product2/**`: products with display ordering; standard pricebook entries.
- `force-app/main/default/objects/Order/**`: store/brand linkage, customer contact, pickup timing, and payment fields.
- `force-app/main/default/objects/OrderItem/**`: standard order line items.

## Notes for Implementers

- Brand is denormalized on Order (`Brand__c`) from the selected Store; update the store to change brand.
- Payment Status and Provider are intended to be system-managed by the integration layer; guardrails (validations/flows) may be added later.
- Pickup date/time is required; lifecycle timestamps are set when statuses change to support SLA reporting.
