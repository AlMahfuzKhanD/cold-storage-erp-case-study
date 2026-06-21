# ERP Modules

## Overview

This document identifies the major business modules found in the Laravel ERP
codebase and describes them from a business and solution architecture
perspective. It does not expose source code, private data, credentials,
production configuration, or company-specific records.

The source application is a combined ERP, but this public case study intentionally
focuses on the Cold Storage domain. Supporting modules are included only where
they directly help explain cold storage operations, settlement, security, or
integration.

## 1. Cold Storage Season and Configuration Module

**Business Purpose**

Configure season-wise cold storage operations, pricing rules, booking types,
business targets, and operational charges.

**Main Tables**

- `c_s_seasons`
- `cs_booking_types`
- `cs_booking_rates`
- `cs_booking_rate_details`
- `cs_other_charges`
- `cs_business_unit_targets`
- `cs_booking_agreement_conditions`

**Important Features**

- Season creation and default season selection
- Booking type setup
- Booking rate and rate-range management
- Labour, fanning, and bag-loan charge setup
- Business unit target setup
- Agreement condition setup for booking documents

**Related Controllers**

- `CSSeasonController`
- `BookingRateInformationController`
- `CSOtherChargeController`
- `CSBusinessUnitTargetController`
- `CsBookingAgreementConditionController`

**Business Rules**

- Operational records are grouped by season.
- Booking rates and charges can vary by season and booking type.
- Agreement conditions should be configurable and reusable in generated
  documents.
- Default season selection affects day-to-day operational data entry.

## 2. Cold Storage Customer Module

**Business Purpose**

Maintain customers and customer account details used in cold storage booking,
loan, delivery, and collection workflows.

**Main Tables**

- `cs_customers`
- `cs_customer_accounts`
- `cs_order_customers`
- Shared master tables such as `customers`, `companies`, `districts`,
  `upazilas`, and bank/account reference tables

**Important Features**

- Cold-storage-specific customer profile
- Customer code and contact information
- Bank/account reference management
- Delivery/order customer management
- Customer lookup through AJAX endpoints

**Related Controllers**

- `CsCustomerController`
- `CsCustomerAccountController`
- `CsOrderCustomerController`
- `CsAjaxController`
- `CustomerController`

**Business Rules**

- Customer information must be available before booking and financial
  settlement.
- Sensitive customer identity, bank, and document details are excluded from the
  public repository.
- Customer account data supports collection, refund, and due workflows.

## 3. Booking Module

**Business Purpose**

Manage seasonal customer bookings, which are the central business reference for
the cold storage lifecycle.

**Main Tables**

- `cs_target_bookings`
- `c_s_bookings`
- `c_s_booking_details`
- `cs_booking_rates`
- `cs_booking_rate_details`
- `cs_booking_agreement_conditions`

**Important Features**

- Manual and auto booking number handling
- Seasonal customer booking
- Booking type and loan type selection
- Bag quantity and rate capture
- Booking agreement printing
- Booking import/migration support
- Booking status tracking for downstream processes

**Related Controllers**

- `CsTargetBookingController`
- `BookingRateInformationController`
- `CsBookingAgreementConditionController`
- `CsAjaxController`

**Business Rules**

- Booking belongs to a company and season.
- Booking number is the central reference for SR, loan, collection, order,
  delivery, and reporting.
- Booking type affects rates, charges, and later workflow behavior.
- Booking status indicates whether order, manual delivery, or loan processing
  has already occurred.

## 4. Loan and Financial Obligation Module

**Business Purpose**

Manage customer financing, loan requests, opening balances, SR-level loans,
delivery loan adjustment, overdue balances, and season-to-season migration.

**Main Tables**

- `cs_loan_requests`
- `cs_loan_balances`
- `cs_loans`
- `sr_loans`
- `cs_do_loans`
- `cs_credit_due_season_migrations`

**Important Features**

- Loan request creation and approval
- Opening loan balance setup
- SR-level loan recording
- Loan adjustment during delivery
- Loan recovery schedule
- Party loan register
- Overdue and other overdue tracking
- Season credit/due migration

**Related Controllers**

- `CsLoanRequestController`
- `CsLoanBalanceController`
- `LoanInformationController`
- `CsReportController`

**Business Rules**

- Loan activity must be linked to booking, customer, company, and season.
- Approved loan amounts should be distinguished from requested and disbursed
  amounts.
- Delivery settlement must consider outstanding loan and overdue balances.
- Migration should preserve source and target season traceability.

## 5. Booking Collection and Due Collection Module

**Business Purpose**

Track payments received against bookings, delivery dues, and sale obligations.

**Main Tables**

- `cs_collections`
- `cs_credit_customer_dues`
- `cs_due_collections`
- `cs_sale_collections`
- `cs_customer_accounts`

**Important Features**

- Paid booking collection
- Delivery due collection
- Sale collection
- Receipt printing
- Receivable and received amount tracking
- Payment mode and bank/account references
- Credit customer ledger reporting

**Related Controllers**

- `CsCollectionController`
- `CsDueCollectionController`
- `CsSaleCollectionController`
- `CsReportController`

**Business Rules**

- Collections must be linked to the related booking, delivery, sale, customer,
  and company where applicable.
- Advance collections reduce final payable amounts during delivery settlement.
- Due collections must remain traceable to the delivery or sale that created the
  receivable.
- Public payment examples use anonymized data only.

## 6. Store Receive Module

**Business Purpose**

Record the physical receipt of customer goods into cold storage. Store Receive
is the operational stock reference under a booking.

**Main Tables**

- `cs_store_receiveds`
- `cs_target_bookings`
- `sr_loans`
- `cs_storage_events`
- `cs_storage_current`

**Important Features**

- SR number generation and tracking
- Multiple SR records under one booking
- Received quantity, booking quantity, and loan quantity tracking
- Sub-customer handling
- Store receive PDF generation
- Load history reporting

**Related Controllers**

- `CsSrController`
- `CsStorageLoadController`
- `CsStoragePallotController`
- `CsAjaxController`

**Business Rules**

- SR belongs to a booking, company, and season.
- A booking can have multiple SR entries.
- SR is the bridge between commercial booking and physical inventory.
- Reserved and delivered quantities are affected by weight and delivery
  processes.

## 7. Storage Location and Movement Module

**Business Purpose**

Represent the physical storage structure and track where goods are stored or
moved inside the cold storage facility.

**Main Tables**

- `cs_chambers`
- `cs_floors`
- `cs_pockets`
- `cs_pocket_positions`
- `cs_storage_events`
- `cs_storage_movements`
- `cs_storage_current`

**Important Features**

- Chamber setup
- Floor setup
- Pocket setup
- Position setup
- Storage loading
- Pallet/pallot movement
- Current storage location tracking
- Movement history and correction support

**Related Controllers**

- `CsChamberController`
- `CsFloorController`
- `CsPocketController`
- `CsPocketPositionController`
- `CsStorageLoadController`
- `CsStoragePallotController`
- `CsAjaxController`

**Business Rules**

- Physical location follows chamber, floor, pocket, and position hierarchy.
- Loading and pallet movement should create auditable storage events.
- Current storage should represent the latest known position of each SR.
- The same SR may be distributed across multiple positions.
- Current-location records should avoid duplicate rows for the same SR, pocket,
  and position.

## 8. Order, Weight, and Delivery Module

**Business Purpose**

Prepare delivery requests, capture actual machine weight, validate deliverable
stock, finalize delivery orders, and produce delivery documents.

**Main Tables**

- `cs_orders`
- `cs_order_details`
- `cs_weight_data`
- `cs_delivery_order_masters`
- `cs_delivery_order_details`
- `cs_do_carts`
- `cs_do_loans`
- `cs_store_receiveds`

**Important Features**

- Order creation for delivery preparation
- SR-level order lines
- Live weight capture
- Desktop weight synchronization
- Manual delivery order creation
- Auto delivery order creation
- Delivery challan and gate copy printing
- Delivery edit/delete stock synchronization
- Reserved and delivered quantity calculation

**Related Controllers**

- `CsOrderInformationController`
- `CsWeightCalculationController`
- `DoLiveWeightCalculationController`
- `CsDeliveryOrderController`
- `CsSingleDeliveryOrderController`
- `AutoDeliveryOrderController`
- `CsDeliveryOrderReportController`

**Business Rules**

- Delivery quantity cannot exceed weighted reserved stock for the selected SR.
- Weight rows must be linked to SR, booking, company, and order detail.
- Settled weight rows should not be reused in another delivery.
- Delivery is both a stock event and a financial settlement event.
- Delivery edits or deletions must release or resynchronize affected weights and
  SR quantities.

## 9. Rent, Charge, and Delivery Settlement Module

**Business Purpose**

Calculate final delivery settlement by combining stock quantity, actual weight,
rent, charges, advance collections, loan balances, rebates, received amounts,
refunds, and dues.

**Main Tables**

- `cs_delivery_order_masters`
- `cs_delivery_order_details`
- `cs_other_charges`
- `cs_booking_rate_details`
- `cs_collections`
- `cs_loan_balances`
- `cs_do_loans`
- `cs_credit_customer_dues`

**Important Features**

- Rent calculation
- Rate per bag and rate per kg support
- Fanning and labour charge calculation
- Loan and overdue balance adjustment
- Advance paid adjustment
- Commission/rebate adjustment
- Refund and due calculation

**Related Controllers**

- `CsDeliveryOrderController`
- `CsSingleDeliveryOrderController`
- `AutoDeliveryOrderController`
- `CsDueCollectionController`
- `CsReportController`

**Business Rules**

- Actual measured weight can affect settlement.
- Charges must follow configured season/booking rules.
- Outstanding loans and overdue balances must be considered before final
  payable or refundable amount is determined.
- Any remaining amount due should create or update customer due tracking.

## 10. Cold Storage Reporting Module

**Business Purpose**

Provide operational and management visibility for bookings, stock, delivery,
loan, collection, dues, and daily storage activity.

**Main Tables**

- `cs_target_bookings`
- `cs_store_receiveds`
- `cs_weight_data`
- `cs_delivery_order_masters`
- `cs_delivery_order_details`
- `cs_collections`
- `cs_due_collections`
- `cs_loan_balances`
- `cs_storage_events`
- `cs_storage_movements`
- `cs_storage_current`

**Important Features**

- Daily statement report
- Credit customer ledger report
- Loan recovery schedule
- Party loan register
- SR report
- Paid booking collection report
- Loan demand report
- Booking collection report
- Daily pallet report
- PDF and Excel style exports

**Related Controllers**

- `CsReportController`
- `CsDeliveryOrderReportController`
- `ReportController`

**Business Rules**

- Reports should support company, season, customer, booking, SR, and date
  filtering.
- Reports should reconcile operational stock with financial status.
- Exported report examples use sanitized or demo data.

## 11. Seed Stock and Potato Sales Module

**Business Purpose**

Manage seed/potato stock, movement, sale, delivery, and collection as a
supporting business process beside customer cold storage operations.

**Main Tables**

- `seed_stocks`
- `stock_movements`
- `cs_sales`
- `cs_sale_items`
- `cs_sale_collections`

**Important Features**

- Restock entry
- Stock transfer
- Stock movement history
- Seed/potato sale creation
- Sale item lines
- Sale delivery status
- Sale collection and receipt
- Stock report

**Related Controllers**

- `SeedStockController`
- `CsSaleController`
- `CsSaleCollectionController`

**Business Rules**

- Stock movement should preserve source, movement type, reference, date, and
  quantity.
- Sales should reduce available stock only through controlled movement or
  delivery logic.
- Collections should be linked to the sale and customer.

## 12. Accounting and Finance Module

**Business Purpose**

Provide general finance, ledger, bank, voucher, cost center, and financial
reporting functions for the broader ERP.

**Main Tables**

- `afm_coas`
- `afm_calendars`
- `afm_calendar_details`
- `afm_vouchers`
- `afm_voucher_details`
- `afm_voucher_docs`
- `afm_voucher_posts`
- `afm_trial_balances`
- `afm_money_receipts`
- `bank_information`
- `bank_accounts`
- `bank_check_mappings`
- `cost_centers`
- `accounting_setups`
- `voucher_logs`

**Important Features**

- Chart of accounts
- Accounting calendar
- Voucher entry and posting
- Voucher document attachment references
- Money receipt
- Bank and bank account setup
- Bank check mapping
- Cost center setup
- Daily ledger reports
- Summary ledgers by account, customer, supplier, employee, and loan
- Balance sheet and income statement style reports

**Related Controllers**

- `AfmCoaController`
- `AfmCalendarController`
- `AfmVoucherController`
- `AccountingSetupController`
- `AccountReportController`
- `DailyLedgerController`
- `BankDailyLedgerController`
- `GeneralDailyLedgerController`
- `DailyIncomeStatementReportController`
- `BalanceSheetController`
- `CostCenterController`
- `BankInformationController`
- `BankAccountController`
- `BankCheckMappingController`
- `MoneyReceiptController`
- `AccountSummaryLedgerController`
- `CustomerSummaryLedgerController`
- `SupplierSummaryLedgerController`
- `EmployeeSummaryLedgerController`
- `LoanSummaryLedgerController`

**Business Rules**

- Financial documents should be tied to fiscal periods.
- Voucher posting should preserve auditability.
- Ledger summaries should be filterable by date, account, party, and cost
  center where applicable.
- This case study describes finance workflows conceptually without exposing real
  ledger data.

## 13. User, Role, Permission, and Administration Module

**Business Purpose**

Control access, menus, users, roles, permissions, system settings, notifications,
and administrative module setup.

**Main Tables**

- `users`
- `user_infos`
- `user_details`
- `user_addresses`
- `roles`
- `permissions`
- `role_permissions`
- `role_groups`
- `menus`
- `admin_modules`
- `admin_module_groups`
- `settings`
- `notifications`
- `login_logs`
- `user_macs`
- `user_ip_addresses`
- `personal_access_tokens`
- `activity_log`

**Important Features**

- User management
- Admin user management
- Role and permission setup
- Role-permission assignment
- Menu and module setup
- Login logging
- MAC/IP based metadata
- System settings
- Notifications
- Activity logging
- API token support

**Related Controllers**

- `UserController`
- `AdminUserController`
- `UserDetailController`
- `RoleController`
- `PermissionController`
- `RolePermissionController`
- `MenuController`
- `AdminModuleController`
- `SettingController`
- `NotificationController`

**Business Rules**

- Users should only access modules allowed by assigned roles and permissions.
- Login and activity logs support auditability.
- Token records support API/desktop integrations.
- Public documentation excludes real users, login logs, MAC addresses, IP
  addresses, and token data.

## 14. API and Integration Module

**Business Purpose**

Expose selected ERP capabilities to companion applications, desktop clients, and
data-sharing workflows.

**Main Tables**

- `personal_access_tokens`
- `cs_weight_data`
- `users`
- `menus`
- `companies`
- `departments`
- `divisions`
- `hr_employees`
- `market_setups`

**Important Features**

- JWT-authenticated user/menu API
- Sanctum-authenticated desktop weight API
- Weight login/logout
- Order list for weighing desktop app
- Idempotent weight synchronization
- Live weight receive/latest weight endpoints
- Market hierarchy APIs
- JSON import/share endpoints
- Company, department, division, and employee list APIs

**Related Controllers**

- `AuthController`
- `WeightController`
- `DoLiveWeightCalculationController`
- `MenuController`
- `UserController`
- `AdminUserController`
- `CompanyController`
- `DepartmentController`
- `DivisionController`
- `EmployeeController`
- `JsonImportController`
- Market setup API controllers

**Business Rules**

- External clients must authenticate before accessing protected business data.
- Weight sync should be retry-safe and duplicate-safe.
- Machine and desktop identifiers should be treated as operational metadata, not
  public documentation data.
- Import APIs should validate incoming shared data before persistence.
