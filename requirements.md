# SnapBill - Software Requirements Specification (SRS)

**Version:** 1.0  
**Date:** February 15, 2026  
**Status:** Draft

---

## 1. Introduction

### 1.1 Purpose
This document specifies the functional and non-functional requirements for SnapBill, an AI-powered Hybrid Retail Intelligence Platform designed for small and medium Indian retailers. This specification serves as the foundation for system design, development, testing, and validation.

### 1.2 Scope
SnapBill is a comprehensive retail management system that combines AI-powered voice billing, offline-first architecture, inventory management, GST compliance, and analytics capabilities. The system consists of:

- **Flutter Mobile Frontend**: Cross-platform mobile application for Android devices
- **FastAPI Backend**: Python-based REST API server
- **PostgreSQL Database**: Supabase-hosted relational database
- **Google Gemini AI Integration**: Natural language processing for voice commands
- **Bluetooth Thermal Printer Support**: ESC/POS compatible printing

### 1.3 Document Conventions
- **SHALL**: Indicates mandatory requirements
- **SHOULD**: Indicates recommended requirements
- **MAY**: Indicates optional requirements
- **WHEN**: Precondition for requirement execution

### 1.4 Intended Audience
- Product Managers
- Software Architects
- Backend Developers
- Mobile Developers
- QA Engineers
- DevOps Engineers
- Compliance Officers

---

## 2. Vision

SnapBill aims to revolutionize retail operations for Indian SMEs by providing an intelligent, voice-enabled, offline-capable point-of-sale system that simplifies billing, ensures GST compliance, and delivers actionable business insights. The platform will evolve into a B2B procurement marketplace and extend into healthcare prescription management.

---

## 3. Glossary


| Term | Definition |
|------|------------|
| **SKU** | Stock Keeping Unit - unique identifier for inventory items |
| **HSN** | Harmonized System of Nomenclature - standardized product classification |
| **GSTIN** | Goods and Services Tax Identification Number |
| **CGST** | Central Goods and Services Tax |
| **SGST** | State Goods and Services Tax |
| **IGST** | Integrated Goods and Services Tax (inter-state) |
| **GSTR-1** | Monthly/Quarterly return of outward supplies |
| **GSTR-3B** | Monthly summary return |
| **RAG** | Retrieval-Augmented Generation - AI technique using context |
| **ESC/POS** | Standard printer command language |
| **JWT** | JSON Web Token - authentication mechanism |
| **OTP** | One-Time Password |
| **UPI** | Unified Payments Interface |
| **Hinglish** | Mix of Hindi and English languages |

---

## 4. System Actors

### 4.1 Primary Actors
- **Owner**: Full system access, business analytics, configuration
- **Manager**: Inventory management, reporting, user management
- **Cashier**: Billing operations, customer interactions
- **Customer**: Receives invoices, makes payments

### 4.2 External Systems
- **Google Gemini AI**: Voice command processing
- **Supabase**: Database and authentication services
- **Bluetooth Printer**: Invoice printing device
- **WhatsApp API**: Invoice sharing
- **Email Service**: Invoice delivery
- **Payment Gateway**: UPI/digital payments

---

## 5. Assumptions

1. Retailers have Android devices (version 8.0+) with minimum 2GB RAM
2. Internet connectivity is intermittent but available periodically
3. Retailers have basic smartphone operation knowledge
4. Bluetooth thermal printers support ESC/POS protocol
5. Retailers have valid GSTIN for GST compliance
6. Voice commands are spoken in Hindi, English, or Hinglish
7. Retailers maintain product catalogs with accurate pricing
8. Mobile devices have microphone access for voice input

---

## 6. Constraints

### 6.1 Technical Constraints
- Must support low-end Android devices (2GB RAM, Android 8+)
- Must function with intermittent internet connectivity
- Voice processing latency must not exceed 2 seconds
- Local storage limited to device capacity
- Bluetooth printer range limited to 10 meters

### 6.2 Regulatory Constraints
- Must comply with Indian GST regulations
- Must maintain transaction records for 6 years
- Must follow Indian data protection guidelines
- Invoice numbering must be sequential and non-repeating

### 6.3 Business Constraints
- Target market: Small and medium Indian retailers
- Initial deployment: Android platform only
- Portrait orientation optimized for handheld use
- Must support regional languages (Hindi/English)

---

## 7. Functional Requirements - Phase 1 (Core System)

### 7.1 Voice-Based Billing Module

#### 7.1.1 Voice Command Processing

**FR-VB-001**: Voice Input Capture  
WHEN a user activates voice billing mode, THE SYSTEM SHALL capture audio input through the device microphone and transmit it to the Google Gemini AI service for processing.

**User Story**: As a cashier, I want to create bills using voice commands so that I can serve customers faster without typing.

**Acceptance Criteria**:
- Voice recording starts within 500ms of activation
- Audio quality sufficient for speech recognition (16kHz minimum)
- Visual feedback indicates recording status
- Support for push-to-talk and continuous listening modes
- Background noise cancellation enabled

---

**FR-VB-002**: Multi-Language Support  
WHEN processing voice commands, THE SYSTEM SHALL support Hindi, English, and Hinglish (mixed Hindi-English) languages with equivalent accuracy.

**User Story**: As a retailer in a Hindi-speaking region, I want to use voice commands in my native language so that I can operate the system naturally.

**Acceptance Criteria**:
- Recognize Hindi voice commands with >85% accuracy
- Recognize English voice commands with >90% accuracy
- Recognize Hinglish commands with >80% accuracy
- Support language switching within same session
- Handle code-mixing (switching between languages mid-sentence)

---

**FR-VB-003**: Item Matching with RAG  
WHEN voice input is received, THE SYSTEM SHALL send the retailer's complete inventory context to Google Gemini AI using Retrieval-Augmented Generation (RAG) to match spoken items with inventory SKUs.

**User Story**: As a cashier, I want the system to understand product names even when I use local nicknames so that billing is accurate.

**Acceptance Criteria**:
- Send inventory catalog (SKU, name, aliases, category) as context
- Match items within 2 seconds of voice input completion
- Support fuzzy matching for pronunciation variations
- Handle multi-alias item names (e.g., "cold drink", "Coke", "Coca-Cola")
- Return confidence score for each match

---

**FR-VB-004**: Quantity and Rate Extraction  
WHEN processing voice commands, THE SYSTEM SHALL extract quantity, unit price, and calculate line item totals automatically.

**User Story**: As a cashier, I want to say "2 kg rice at 50 rupees" and have the system calculate the total so that I don't need to do mental math.

**Acceptance Criteria**:
- Extract numeric quantities from voice input
- Extract unit prices when specified
- Use inventory default price when not specified
- Support units: pieces, kg, liters, dozens, packets
- Calculate line total (quantity × rate)
- Handle decimal quantities (e.g., "1.5 kg")

---

**FR-VB-005**: Ambiguous Match Resolution  
WHEN multiple inventory items match a voice command, THE SYSTEM SHALL present disambiguation options to the user for selection.

**User Story**: As a cashier, when I say "soap" and there are multiple soap brands, I want to see options so that I can select the correct one.

**Acceptance Criteria**:
- Display top 5 matching items with confidence scores
- Show item name, SKU, price, and stock quantity
- Allow selection via touch or voice ("option 1", "first one")
- Timeout after 10 seconds with default selection
- Learn from user selections to improve future matches

---

**FR-VB-006**: Voice Command Operations  
WHEN a user issues operational commands, THE SYSTEM SHALL support delete, cancel, undo, and confirm operations via voice.

**User Story**: As a cashier, I want to say "delete last item" or "cancel bill" so that I can correct mistakes without touching the screen.

**Acceptance Criteria**:
- "Delete last item" removes most recent line item
- "Delete [item name]" removes specific item
- "Cancel bill" clears entire bill with confirmation
- "Undo" reverses last action
- "Confirm" or "done" finalizes bill
- "Change quantity" allows modification of last item

---

**FR-VB-007**: Performance Requirements  
WHEN processing voice commands, THE SYSTEM SHALL complete item matching and display results within 2 seconds of voice input completion.

**Acceptance Criteria**:
- Voice-to-text conversion < 1 second
- AI matching with RAG < 1 second
- Total end-to-end latency < 2 seconds
- Display loading indicator during processing
- Graceful degradation if AI service is slow

---

### 7.2 Offline Frequent Billing Module

#### 7.2.1 Offline Operation

**FR-OF-001**: Local Product Cache  
WHEN the application initializes, THE SYSTEM SHALL cache the top 100 most frequently sold products locally for offline billing.

**User Story**: As a retailer with unreliable internet, I want to create bills for my popular items even when offline so that I never lose a sale.

**Acceptance Criteria**:
- Identify top 100 products by transaction frequency (last 30 days)
- Store product details: SKU, name, price, GST rate, HSN code
- Update cache daily when online
- Cache size < 5MB for performance
- Persist cache across app restarts

---

**FR-OF-002**: Offline Transaction Creation  
WHEN internet connectivity is unavailable, THE SYSTEM SHALL allow users to create and complete transactions using cached products without requiring network access.

**User Story**: As a cashier during a power outage, I want to continue billing customers so that business operations are not disrupted.

**Acceptance Criteria**:
- Create bills using cached products only
- Store transactions locally in SQLite database
- Generate sequential invoice numbers offline
- Calculate GST and totals without server
- Display "Offline Mode" indicator clearly
- Prevent access to non-cached products

---

**FR-OF-003**: Cloud Synchronization  
WHEN internet connectivity is restored, THE SYSTEM SHALL automatically synchronize all offline transactions to the cloud backend within 15 minutes.

**User Story**: As a store owner, I want offline sales to automatically sync to the cloud so that my reports are always accurate.

**Acceptance Criteria**:
- Detect network connectivity restoration
- Upload pending transactions in chronological order
- Sync every 15 minutes when online
- Immediate sync for critical transactions (>₹10,000)
- Display sync status and progress
- Retry failed syncs with exponential backoff

---

**FR-OF-004**: Conflict Resolution  
WHEN synchronizing offline transactions, THE SYSTEM SHALL detect and resolve conflicts such as duplicate invoice numbers or inventory discrepancies.

**User Story**: As a manager, I want the system to handle sync conflicts intelligently so that data integrity is maintained.

**Acceptance Criteria**:
- Detect duplicate invoice numbers across devices
- Resolve conflicts using timestamp priority
- Adjust inventory quantities based on all transactions
- Log all conflict resolutions for audit
- Notify user of critical conflicts requiring manual review
- Maintain transaction immutability (no data loss)

---

**FR-OF-005**: Transaction Integrity  
WHEN operating in offline mode, THE SYSTEM SHALL ensure zero transaction loss through persistent local storage and guaranteed sync mechanisms.

**User Story**: As a retailer, I want absolute confidence that no sale is ever lost so that my revenue tracking is accurate.

**Acceptance Criteria**:
- Persist transactions to local database immediately
- Implement write-ahead logging for crash recovery
- Mark transactions as "synced" or "pending"
- Prevent deletion of unsynced transactions
- Recover incomplete transactions after app crash
- Maintain sync queue across app restarts

---

**FR-OF-006**: Poor Network Handling  
WHEN network conditions are poor (high latency, packet loss), THE SYSTEM SHALL gracefully degrade functionality while maintaining core billing operations.

**User Story**: As a cashier with slow internet, I want the app to remain responsive so that I can continue serving customers.

**Acceptance Criteria**:
- Timeout network requests after 10 seconds
- Fall back to offline mode automatically
- Queue operations for later sync
- Display network quality indicator
- Compress sync payloads to reduce bandwidth
- Support resumable uploads for large data

---

### 7.3 Inventory Management Module

#### 7.3.1 Product Management

**FR-IM-001**: SKU-Based Tracking  
WHEN adding products to inventory, THE SYSTEM SHALL assign unique SKU identifiers and track all product attributes including name, category, price, GST rate, and HSN code.

**User Story**: As a store manager, I want each product to have a unique identifier so that I can track inventory accurately.

**Acceptance Criteria**:
- Generate unique SKU automatically or accept manual entry
- Validate SKU uniqueness across inventory
- Store: name, description, category, price, cost, GST rate, HSN code
- Support barcode/QR code association with SKU
- Prevent duplicate SKU creation
- Allow SKU search and filtering

---

**FR-IM-002**: Multi-Alias Item Names  
WHEN creating or editing products, THE SYSTEM SHALL support multiple alias names for each product to improve voice recognition and search.

**User Story**: As a retailer, I want to add nicknames for products so that customers and staff can refer to items in different ways.

**Acceptance Criteria**:
- Add unlimited aliases per product
- Store aliases in searchable format
- Use aliases in voice command matching
- Display primary name on invoices
- Support regional language aliases
- Prevent duplicate aliases across products

---

**FR-IM-003**: Category Hierarchy  
WHEN organizing inventory, THE SYSTEM SHALL support multi-level category hierarchies (e.g., Food > Snacks > Chips) for better organization and reporting.

**User Story**: As a store manager, I want to organize products into categories so that I can analyze sales by product type.

**Acceptance Criteria**:
- Support up to 5 levels of category nesting
- Create, edit, and delete categories
- Assign products to categories
- Move products between categories
- Generate category-wise sales reports
- Display category tree in navigation

---

**FR-IM-004**: GST Rate and HSN Code Management  
WHEN configuring products, THE SYSTEM SHALL support Indian GST rates (0%, 5%, 12%, 18%, 28%) and HSN code assignment for tax compliance.

**User Story**: As a store owner, I want to set correct GST rates for products so that my tax calculations are compliant.

**Acceptance Criteria**:
- Support standard GST rates: 0%, 5%, 12%, 18%, 28%
- Validate HSN code format (4-8 digits)
- Auto-suggest HSN codes based on category
- Calculate CGST/SGST or IGST based on transaction type
- Store GST rate history for audit
- Generate GST summary reports

---

**FR-IM-005**: Low Stock Alerts  
WHEN inventory quantity falls below defined threshold, THE SYSTEM SHALL generate low stock alerts and notify relevant users.

**User Story**: As a store manager, I want to be notified when products are running low so that I can reorder in time.

**Acceptance Criteria**:
- Set minimum stock threshold per product
- Generate alert when quantity < threshold
- Display alerts in dashboard
- Send push notifications to managers/owners
- Support bulk threshold configuration
- Track alert history and resolution

---

**FR-IM-006**: Inventory Audit Trail  
WHEN inventory changes occur, THE SYSTEM SHALL maintain a complete audit trail of all stock movements including user, timestamp, and reason.

**User Story**: As a store owner, I want to see who changed inventory levels so that I can prevent theft and errors.

**Acceptance Criteria**:
- Log all inventory transactions: sales, purchases, adjustments
- Record: timestamp, user, action type, quantity change, reason
- Support manual stock adjustments with mandatory notes
- Generate audit reports by date range
- Prevent audit log modification or deletion
- Export audit trail for external review

---

**FR-IM-007**: Product Variants  
WHEN managing products with variations, THE SYSTEM SHALL support variant tracking (e.g., size, color, flavor) under a single parent product.

**User Story**: As a retailer selling shirts in multiple sizes, I want to track each size separately so that inventory is accurate.

**Acceptance Criteria**:
- Create parent product with variant attributes
- Define variant options (e.g., Size: S, M, L, XL)
- Track inventory per variant
- Price variants independently or use base price
- Display variants in billing interface
- Generate variant-wise sales reports

---

**FR-IM-008**: Batch Inventory Updates  
WHEN updating multiple products, THE SYSTEM SHALL support bulk operations via CSV import/export for efficient inventory management.

**User Story**: As a manager with 1000+ products, I want to update prices in bulk so that I don't waste time on manual entry.

**Acceptance Criteria**:
- Export inventory to CSV format
- Import inventory updates from CSV
- Validate CSV format and data types
- Preview changes before applying
- Support bulk price updates, stock adjustments
- Log all bulk operations in audit trail
- Handle errors gracefully with detailed reports

---

### 7.4 Analytics Module (Integrated in History Page)

#### 7.4.1 Sales Analytics

**FR-AN-001**: Analytics Dashboard Integration  
WHEN viewing the History page, THE SYSTEM SHALL display a comprehensive analytics dashboard at the top showing key business metrics before the bill history list.

**User Story**: As a store owner, I want to see sales performance at a glance so that I can make informed business decisions.

**Acceptance Criteria**:
- Display analytics section above bill history
- Show metrics in card-based layout
- Support time period filters: Today, Weekly, Monthly
- Refresh analytics on page load
- Cache analytics for offline viewing
- Smooth scrolling between analytics and history

---

**FR-AN-002**: Total Sales Metrics  
WHEN displaying analytics, THE SYSTEM SHALL show total sales revenue segmented by Today, Weekly, and Monthly periods with comparison to previous periods.

**User Story**: As a store owner, I want to see daily, weekly, and monthly sales so that I can track revenue trends.

**Acceptance Criteria**:
- Calculate total sales for today (midnight to current time)
- Calculate weekly sales (last 7 days)
- Calculate monthly sales (current calendar month)
- Show percentage change vs previous period
- Display currency in INR format (₹)
- Include GST breakdown in totals

---

**FR-AN-003**: Top Selling Products  
WHEN analyzing sales performance, THE SYSTEM SHALL identify and display top-selling products ranked by both quantity sold and revenue generated.

**User Story**: As a manager, I want to know which products sell the most so that I can optimize inventory.

**Acceptance Criteria**:
- Display top 10 products by quantity sold
- Display top 10 products by revenue
- Show product name, quantity, revenue, profit margin
- Support filtering by time period
- Visual representation (bar chart or list)
- Drill-down to product details

---

**FR-AN-004**: Profit Margin Analysis  
WHEN calculating profitability, THE SYSTEM SHALL compute and display profit margins at product, category, and overall business levels.

**User Story**: As a store owner, I want to see profit margins so that I can identify which products are most profitable.

**Acceptance Criteria**:
- Calculate profit: (selling price - cost price) × quantity
- Display profit margin percentage: (profit / revenue) × 100
- Show gross profit and net profit
- Category-wise profit analysis
- Product-wise profit ranking
- Highlight low-margin products

---

**FR-AN-005**: Peak Sales Hours  
WHEN analyzing transaction patterns, THE SYSTEM SHALL identify peak sales hours and display hourly sales distribution.

**User Story**: As a store owner, I want to know when customers shop most so that I can optimize staffing.

**Acceptance Criteria**:
- Aggregate sales by hour of day
- Display hourly sales chart (24-hour format)
- Identify top 3 peak hours
- Show transaction count and revenue per hour
- Support day-of-week analysis
- Compare weekday vs weekend patterns

---

**FR-AN-006**: Slow-Moving Items  
WHEN monitoring inventory health, THE SYSTEM SHALL identify slow-moving items that haven't sold within defined time periods.

**User Story**: As a manager, I want to identify products that aren't selling so that I can plan clearance sales.

**Acceptance Criteria**:
- Identify items with zero sales in last 30 days
- Identify items with sales below threshold
- Display: product name, last sale date, current stock
- Calculate inventory holding cost
- Suggest discount strategies
- Export slow-moving items list

---

**FR-AN-007**: GST Collection Breakdown  
WHEN reviewing tax compliance, THE SYSTEM SHALL display GST collection breakdown by tax rate and type (CGST/SGST/IGST).

**User Story**: As a store owner, I want to see GST collected so that I can prepare tax returns accurately.

**Acceptance Criteria**:
- Total GST collected (all rates)
- Breakdown by rate: 0%, 5%, 12%, 18%, 28%
- CGST/SGST vs IGST separation
- Taxable amount and tax amount per rate
- Support date range filtering
- Export GST summary for filing

---

**FR-AN-008**: Customer Insights  
WHEN analyzing customer behavior, THE SYSTEM SHALL track repeat customers, lifetime value, and purchase frequency.

**User Story**: As a store owner, I want to identify loyal customers so that I can reward them.

**Acceptance Criteria**:
- Identify repeat customers (>3 purchases)
- Calculate customer lifetime value (CLV)
- Average transaction value per customer
- Purchase frequency analysis
- Customer segmentation (high/medium/low value)
- Display top 20 customers by revenue

---

**FR-AN-009**: Export Capability  
WHEN generating reports, THE SYSTEM SHALL support exporting analytics data in CSV and PDF formats for external analysis and record-keeping.

**User Story**: As a store owner, I want to export sales reports so that I can share them with my accountant.

**Acceptance Criteria**:
- Export analytics to CSV format
- Export analytics to PDF format
- Include all metrics and charts in export
- Support date range selection for export
- Email export directly from app
- Save export to device storage

---

**FR-AN-010**: Backend Analytics Queries  
WHEN computing analytics, THE SYSTEM SHALL execute optimized database queries on the backend to ensure fast performance even with large datasets.

**Acceptance Criteria**:
- Analytics computed on backend (not mobile)
- Response time < 3 seconds for all queries
- Use database indexing for performance
- Cache frequently accessed analytics
- Support pagination for large result sets
- Handle concurrent analytics requests

---

### 7.5 GST Compliance Engine

#### 7.5.1 Tax Calculation

**FR-GST-001**: HSN-Based GST Application  
WHEN calculating taxes, THE SYSTEM SHALL apply GST rates based on HSN code classification according to Indian GST regulations.

**User Story**: As a store owner, I want GST calculated automatically based on product type so that I remain compliant.

**Acceptance Criteria**:
- Map HSN codes to GST rates
- Apply correct rate: 0%, 5%, 12%, 18%, 28%
- Validate HSN code format (4-8 digits)
- Support HSN code updates when rates change
- Display HSN code on invoices
- Generate HSN-wise sales summary

---

**FR-GST-002**: CGST/SGST vs IGST Logic  
WHEN generating invoices, THE SYSTEM SHALL apply CGST+SGST for intra-state transactions and IGST for inter-state transactions based on customer and retailer GSTIN.

**User Story**: As a retailer, I want the system to automatically choose between CGST/SGST and IGST so that my invoices are correct.

**Acceptance Criteria**:
- Extract state code from GSTIN (first 2 digits)
- Compare retailer state vs customer state
- Apply CGST+SGST if states match (each = GST rate / 2)
- Apply IGST if states differ (= full GST rate)
- Display tax breakdown on invoice
- Handle unregistered customers (default to intra-state)

---

**FR-GST-003**: Sequential Invoice Numbering  
WHEN creating invoices, THE SYSTEM SHALL generate sequential, non-repeating invoice numbers that comply with GST regulations.

**User Story**: As a store owner, I want invoice numbers to be sequential so that I comply with GST audit requirements.

**Acceptance Criteria**:
- Generate invoice numbers in format: INV-YYYY-NNNNNN
- Ensure strict sequential ordering
- No gaps or duplicates in sequence
- Reset sequence annually (optional)
- Handle offline invoice numbering without conflicts
- Prevent manual invoice number editing

---

**FR-GST-004**: GSTIN Support  
WHEN processing B2B transactions, THE SYSTEM SHALL capture and validate customer GSTIN for tax credit eligibility.

**User Story**: As a cashier serving business customers, I want to record their GSTIN so that they can claim input tax credit.

**Acceptance Criteria**:
- Capture customer GSTIN (15 characters)
- Validate GSTIN format: 2-digit state + 10-digit PAN + 3-digit code
- Verify GSTIN checksum digit
- Store GSTIN with customer profile
- Display GSTIN on B2B invoices
- Mark invoices as B2B or B2C

---

**FR-GST-005**: GSTR-1 and GSTR-3B Data Compatibility  
WHEN preparing tax returns, THE SYSTEM SHALL maintain transaction data in formats compatible with GSTR-1 (outward supplies) and GSTR-3B (summary return) filing requirements.

**User Story**: As a store owner, I want to export data for GST filing so that I can file returns easily.

**Acceptance Criteria**:
- Store all fields required for GSTR-1
- Store all fields required for GSTR-3B
- Generate GSTR-1 JSON export
- Generate GSTR-3B summary report
- Support B2B, B2C, and export classifications
- Include HSN-wise summary

---

**FR-GST-006**: Credit Note Support  
WHEN processing returns or cancellations, THE SYSTEM SHALL generate credit notes that adjust GST liability and maintain audit trail.

**User Story**: As a cashier processing a return, I want to issue a credit note so that GST is adjusted correctly.

**Acceptance Criteria**:
- Create credit note linked to original invoice
- Reverse GST calculation from original invoice
- Generate unique credit note number
- Display reason for credit note
- Update inventory on credit note issuance
- Include credit notes in GST reports

---

**FR-GST-007**: Record Retention  
WHEN storing transaction data, THE SYSTEM SHALL maintain all GST-related records for a minimum of 6 years as required by Indian tax law.

**User Story**: As a store owner, I want historical records preserved so that I can respond to tax audits.

**Acceptance Criteria**:
- Store invoices for 6 years minimum
- Store credit notes for 6 years minimum
- Prevent deletion of GST records
- Support archival of old records
- Enable retrieval of historical data
- Maintain data integrity over time

---

### 7.6 Invoice System

#### 7.6.1 Invoice Generation

**FR-INV-001**: Professional Invoice Format  
WHEN generating invoices, THE SYSTEM SHALL produce professionally formatted invoices containing all mandatory GST fields and business information.

**User Story**: As a store owner, I want invoices to look professional so that my business appears credible.

**Acceptance Criteria**:
- Include: business name, address, GSTIN, contact
- Include: invoice number, date, time
- Include: customer name, phone, GSTIN (if B2B)
- Line items: product, HSN, quantity, rate, amount
- Tax breakdown: CGST/SGST or IGST
- Grand total in words and figures
- Terms and conditions section

---

**FR-INV-002**: QR Code for UPI Payments  
WHEN generating invoices, THE SYSTEM SHALL include UPI QR codes for digital payment collection.

**User Story**: As a cashier, I want to show a QR code so that customers can pay digitally.

**Acceptance Criteria**:
- Generate UPI QR code with payment details
- Include: UPI ID, amount, transaction reference
- Display QR code on digital invoice
- Print QR code on thermal receipt
- Support multiple UPI apps (GPay, PhonePe, Paytm)
- Validate QR code generation

---

**FR-INV-003**: Multi-Payment Method Support  
WHEN recording payments, THE SYSTEM SHALL support multiple payment methods including cash, UPI, card, and credit.

**User Story**: As a cashier, I want to record different payment types so that my cash reconciliation is accurate.

**Acceptance Criteria**:
- Support payment methods: Cash, UPI, Card, Credit
- Record payment method with transaction
- Support split payments (partial cash + partial UPI)
- Track outstanding credit amounts
- Generate payment method-wise reports
- Display payment method on invoice

---

**FR-INV-004**: Invoice Reprint Capability  
WHEN requested, THE SYSTEM SHALL allow reprinting of historical invoices with "DUPLICATE" watermark.

**User Story**: As a cashier, I want to reprint lost invoices so that customers can get copies.

**Acceptance Criteria**:
- Search invoices by number, date, customer
- Display invoice preview before printing
- Add "DUPLICATE" watermark on reprints
- Log all reprint actions in audit trail
- Maintain original invoice data unchanged
- Support bulk reprinting

---

**FR-INV-005**: Partial Payment Support  
WHEN customers make partial payments, THE SYSTEM SHALL track outstanding balances and payment history.

**User Story**: As a cashier, I want to accept partial payments so that customers can pay in installments.

**Acceptance Criteria**:
- Record partial payment amount
- Calculate outstanding balance
- Track payment history per invoice
- Send payment reminders for pending amounts
- Generate partial payment receipts
- Update customer credit status

---

**FR-INV-006**: Invoice Audit Trail  
WHEN invoice operations occur, THE SYSTEM SHALL maintain complete audit trail of creation, modifications, reprints, and cancellations.

**User Story**: As a store owner, I want to track all invoice activities so that I can prevent fraud.

**Acceptance Criteria**:
- Log invoice creation with user and timestamp
- Log all reprints with reason
- Log cancellations with authorization
- Log modifications (if allowed)
- Prevent unauthorized invoice deletion
- Generate audit reports

---

**FR-INV-007**: Invoice Sharing  
WHEN customers request digital copies, THE SYSTEM SHALL support sharing invoices via WhatsApp and Email.

**User Story**: As a cashier, I want to send invoices on WhatsApp so that customers have digital records.

**Acceptance Criteria**:
- Generate PDF invoice
- Share via WhatsApp with customer phone number
- Send via email with customer email address
- Include business branding in PDF
- Compress PDF for faster sharing
- Track sharing status and delivery

---

### 7.7 User Roles and Authentication

#### 7.7.1 Authentication

**FR-AUTH-001**: OTP-Based Login  
WHEN users log in, THE SYSTEM SHALL authenticate using One-Time Password (OTP) sent to registered mobile numbers.

**User Story**: As a user, I want to log in with OTP so that my account is secure without remembering passwords.

**Acceptance Criteria**:
- Send 6-digit OTP to mobile number
- OTP valid for 5 minutes
- Maximum 3 OTP requests per hour
- Resend OTP option after 30 seconds
- Verify OTP before granting access
- Block account after 5 failed attempts

---

**FR-AUTH-002**: JWT Token Management  
WHEN users authenticate successfully, THE SYSTEM SHALL issue JWT tokens for session management and API authorization.

**User Story**: As a developer, I want secure token-based authentication so that API calls are protected.

**Acceptance Criteria**:
- Generate JWT token on successful login
- Include user ID, role, and expiry in token
- Token validity: 7 days
- Refresh token mechanism for extended sessions
- Validate token on every API request
- Revoke tokens on logout

---

**FR-AUTH-003**: Role-Based Access Control  
WHEN users access system features, THE SYSTEM SHALL enforce role-based permissions for Owner, Manager, and Cashier roles.

**User Story**: As a store owner, I want to control what each employee can do so that sensitive operations are protected.

**Acceptance Criteria**:
- Define three roles: Owner, Manager, Cashier
- Owner: Full access to all features
- Manager: Inventory, reports, user management (no financial settings)
- Cashier: Billing, customer management only
- Enforce permissions at API and UI levels
- Display only authorized features in UI
- Log unauthorized access attempts

---

**FR-AUTH-004**: Account Lockout Policy  
WHEN multiple failed login attempts occur, THE SYSTEM SHALL temporarily lock accounts to prevent brute force attacks.

**User Story**: As a security administrator, I want accounts locked after failed attempts so that unauthorized access is prevented.

**Acceptance Criteria**:
- Lock account after 5 failed OTP attempts
- Lockout duration: 30 minutes
- Send notification to account owner on lockout
- Allow manual unlock by owner
- Log all lockout events
- Display lockout reason to user

---

**FR-AUTH-005**: Session Timeout  
WHEN users remain inactive, THE SYSTEM SHALL automatically log out sessions after 30 minutes of inactivity for security.

**User Story**: As a store owner, I want automatic logout so that unattended devices don't pose security risks.

**Acceptance Criteria**:
- Track user activity (taps, navigation)
- Logout after 30 minutes of inactivity
- Show warning 2 minutes before timeout
- Allow session extension
- Clear sensitive data on logout
- Require re-authentication after timeout

---

### 7.8 Data Sync and Backup

#### 7.8.1 Synchronization

**FR-SYNC-001**: Automatic Sync Schedule  
WHEN the application is online, THE SYSTEM SHALL automatically synchronize data with the cloud backend every 15 minutes.

**User Story**: As a store owner, I want data synced regularly so that I can access it from anywhere.

**Acceptance Criteria**:
- Sync every 15 minutes when online
- Sync on app launch and close
- Sync after critical operations (billing, inventory update)
- Display last sync timestamp
- Show sync progress indicator
- Allow manual sync trigger

---

**FR-SYNC-002**: Critical Transaction Sync  
WHEN high-value transactions occur (>₹10,000), THE SYSTEM SHALL immediately synchronize to cloud regardless of sync schedule.

**User Story**: As a store owner, I want large transactions synced immediately so that I can monitor them in real-time.

**Acceptance Criteria**:
- Detect transactions above ₹10,000 threshold
- Trigger immediate sync on completion
- Retry sync if network fails
- Notify user of sync status
- Log critical transaction syncs
- Support configurable threshold

---

**FR-SYNC-003**: Local Backup Retention  
WHEN storing data locally, THE SYSTEM SHALL maintain a rolling 30-day backup of all transactions and inventory data.

**User Story**: As a store owner, I want local backups so that I can recover data if the cloud is unavailable.

**Acceptance Criteria**:
- Store 30 days of transaction history locally
- Store complete inventory snapshot
- Compress backups to save space
- Automatic backup daily at midnight
- Restore from local backup option
- Delete backups older than 30 days

---

**FR-SYNC-004**: Encryption at Rest  
WHEN storing data locally, THE SYSTEM SHALL encrypt all sensitive data using AES-256 encryption.

**User Story**: As a store owner, I want data encrypted on my device so that it's protected if my phone is stolen.

**Acceptance Criteria**:
- Encrypt local database with AES-256
- Encrypt backup files
- Store encryption keys securely (Android Keystore)
- Decrypt data only when app is unlocked
- Prevent data access from rooted devices
- Comply with data protection standards

---

**FR-SYNC-005**: Encryption in Transit  
WHEN transmitting data to cloud, THE SYSTEM SHALL use TLS 1.3 encryption for all network communications.

**User Story**: As a security administrator, I want encrypted communication so that data cannot be intercepted.

**Acceptance Criteria**:
- Use TLS 1.3 for all API calls
- Validate SSL certificates
- Prevent man-in-the-middle attacks
- Use certificate pinning for critical APIs
- Reject insecure connections
- Log security violations

---

### 7.9 Customer Management

#### 7.9.1 Customer Profiles

**FR-CUST-001**: Credit Tracking  
WHEN customers purchase on credit, THE SYSTEM SHALL track outstanding balances, payment history, and credit limits.

**User Story**: As a store owner, I want to track customer credit so that I can manage receivables effectively.

**Acceptance Criteria**:
- Record credit transactions with customer
- Track total outstanding balance per customer
- Set credit limit per customer
- Alert when credit limit is exceeded
- Record payment history
- Generate credit aging reports

---

**FR-CUST-002**: Customer Lifetime Value  
WHEN analyzing customer data, THE SYSTEM SHALL calculate and display customer lifetime value (CLV) based on purchase history.

**User Story**: As a store owner, I want to identify valuable customers so that I can provide better service.

**Acceptance Criteria**:
- Calculate total revenue per customer
- Calculate average transaction value
- Count total transactions per customer
- Display CLV in customer profile
- Rank customers by CLV
- Track CLV trends over time

---

**FR-CUST-003**: Customer Search  
WHEN looking up customers, THE SYSTEM SHALL support search by name, phone number, and customer ID.

**User Story**: As a cashier, I want to quickly find customers so that I can apply their credit or discounts.

**Acceptance Criteria**:
- Search by full or partial name
- Search by phone number
- Search by customer ID
- Display search results instantly (<1 second)
- Show customer details in results
- Support fuzzy matching for names

---

**FR-CUST-004**: Purchase History Retrieval  
WHEN viewing customer profiles, THE SYSTEM SHALL display complete purchase history including dates, amounts, and products.

**User Story**: As a store owner, I want to see what customers bought so that I can recommend products.

**Acceptance Criteria**:
- Display all transactions for customer
- Show: date, invoice number, amount, payment status
- Filter by date range
- Show product-level details per transaction
- Calculate purchase frequency
- Export purchase history

---

### 7.10 Printer Integration

#### 7.10.1 Bluetooth Printing

**FR-PRINT-001**: Bluetooth Thermal Printer Support  
WHEN printing invoices, THE SYSTEM SHALL connect to Bluetooth thermal printers using ESC/POS protocol.

**User Story**: As a cashier, I want to print receipts on thermal printer so that customers get physical copies.

**Acceptance Criteria**:
- Discover Bluetooth printers in range
- Pair with printer using PIN/passkey
- Send print commands via ESC/POS
- Support 58mm and 80mm paper widths
- Maintain printer connection across sessions
- Handle printer disconnection gracefully

---

**FR-PRINT-002**: Print Formatting Integrity  
WHEN printing invoices, THE SYSTEM SHALL maintain proper formatting including alignment, fonts, and special characters.

**User Story**: As a store owner, I want printed receipts to look professional so that my brand image is maintained.

**Acceptance Criteria**:
- Preserve text alignment (left, center, right)
- Support bold, underline, and double-height text
- Print business logo (if supported)
- Handle special characters (₹, %, etc.)
- Maintain column alignment in tables
- Print QR codes clearly

---

**FR-PRINT-003**: Print Performance  
WHEN initiating print jobs, THE SYSTEM SHALL complete printing within 3 seconds of user action.

**User Story**: As a cashier, I want fast printing so that customers don't wait.

**Acceptance Criteria**:
- Generate print data < 1 second
- Transmit to printer < 1 second
- Complete printing < 3 seconds total
- Display print progress indicator
- Queue multiple print jobs
- Handle print errors gracefully

---

## 8. Functional Requirements - Phase 2 (B2B Direct Commerce Engine)

**Status**: Future Enhancement

### 8.1 B2B Procurement Platform

**FR-B2B-001**: Sales Data Analysis for Procurement  
WHEN analyzing retailer sales patterns, THE SYSTEM SHALL identify top-selling SKUs and predict optimal reorder quantities.

**User Story**: As a retailer, I want the system to suggest what to reorder so that I never run out of popular items.

**Acceptance Criteria**:
- Analyze sales velocity per SKU
- Calculate average daily sales rate
- Predict stockout date based on current inventory
- Suggest reorder quantity based on lead time
- Consider seasonal trends
- Display confidence level for predictions

---

**FR-B2B-002**: Personalized Wholesale Deals  
WHEN displaying procurement options, THE SYSTEM SHALL show personalized wholesale deals based on retailer's purchase history and preferences.

**User Story**: As a retailer, I want to see deals relevant to my business so that I can save money on purchases.

**Acceptance Criteria**:
- Display deals for products retailer sells
- Show bulk pricing tiers (e.g., 10% off for 100 units)
- Highlight time-limited offers
- Calculate potential savings
- Filter deals by category
- Bookmark favorite deals

---

**FR-B2B-003**: Direct Supplier Connection  
WHEN sourcing products, THE SYSTEM SHALL connect retailers directly with verified suppliers, eliminating intermediaries.

**User Story**: As a retailer, I want to buy directly from suppliers so that I get better prices.

**Acceptance Criteria**:
- Display verified supplier profiles
- Show supplier ratings and reviews
- Display supplier location and delivery areas
- Show supplier product catalog
- Enable direct communication with suppliers
- Verify supplier credentials (GSTIN, business license)

---

**FR-B2B-004**: Bulk Pricing Tiers  
WHEN viewing products, THE SYSTEM SHALL display tiered pricing based on order quantity.

**User Story**: As a retailer, I want to see bulk discounts so that I can decide optimal order quantities.

**Acceptance Criteria**:
- Display price tiers (e.g., 1-10: ₹100, 11-50: ₹95, 51+: ₹90)
- Calculate total cost for different quantities
- Show savings compared to retail price
- Highlight best value tier
- Support custom quote requests for large orders
- Display minimum order quantity (MOQ)

---

**FR-B2B-005**: Purchase Order Generation  
WHEN placing bulk orders, THE SYSTEM SHALL automatically generate purchase orders with all necessary details.

**User Story**: As a retailer, I want automated purchase orders so that I can track my procurement.

**Acceptance Criteria**:
- Generate PO with unique number
- Include: supplier details, items, quantities, prices
- Calculate total amount including GST
- Support partial deliveries
- Track PO status (pending, confirmed, delivered)
- Link PO to inventory receipts

---

**FR-B2B-006**: Supplier Rating System  
WHEN evaluating suppliers, THE SYSTEM SHALL display ratings based on delivery time, product quality, and service.

**User Story**: As a retailer, I want to see supplier ratings so that I can choose reliable partners.

**Acceptance Criteria**:
- Collect ratings after each transaction
- Rate on: delivery time, product quality, service, pricing
- Display average rating (1-5 stars)
- Show number of reviews
- Display recent reviews
- Flag problematic suppliers

---

**FR-B2B-007**: Inventory Auto-Update on Delivery  
WHEN bulk orders are delivered, THE SYSTEM SHALL automatically update inventory quantities based on purchase order.

**User Story**: As a retailer, I want inventory updated automatically so that I don't have to enter data manually.

**Acceptance Criteria**:
- Link delivery to purchase order
- Update inventory quantities on confirmation
- Record cost price from PO
- Update product details if changed
- Generate goods received note (GRN)
- Handle partial deliveries

---

**FR-B2B-008**: Demand Aggregation  
WHEN multiple retailers need same products, THE SYSTEM SHALL aggregate demand to negotiate better bulk pricing.

**User Story**: As a small retailer, I want to benefit from collective buying power so that I get competitive prices.

**Acceptance Criteria**:
- Identify common product needs across retailers
- Aggregate demand by product and region
- Negotiate group pricing with suppliers
- Distribute orders to participating retailers
- Calculate individual retailer shares
- Handle logistics coordination

---

**FR-B2B-009**: Transaction Tracking  
WHEN orders are placed, THE SYSTEM SHALL provide real-time tracking of order status, payment, and delivery.

**User Story**: As a retailer, I want to track my orders so that I can plan inventory accordingly.

**Acceptance Criteria**:
- Display order status: placed, confirmed, shipped, delivered
- Show estimated delivery date
- Track payment status
- Send notifications on status changes
- Display delivery partner details
- Support order cancellation before shipment

---

## 9. Functional Requirements - Phase 3 (Digital Prescription Mode)

**Status**: Healthcare Extension Module

### 9.1 Digital Prescription System

**FR-RX-001**: Structured Prescription Creation  
WHEN doctors create prescriptions, THE SYSTEM SHALL provide structured forms for entering patient and medication details.

**User Story**: As a doctor, I want to create digital prescriptions so that pharmacists can read them clearly.

**Acceptance Criteria**:
- Structured form with required fields
- Auto-complete for common medications
- Dosage templates (e.g., "1-0-1 after meals")
- Duration selection (days/weeks)
- Special instructions field
- Prescription preview before saving

---

**FR-RX-002**: Doctor Profile Management  
WHEN doctors use the system, THE SYSTEM SHALL maintain doctor profiles including name, registration number, and specialization.

**User Story**: As a doctor, I want my credentials displayed on prescriptions so that they are legally valid.

**Acceptance Criteria**:
- Store doctor name, registration number, specialization
- Validate medical registration number format
- Display credentials on prescription
- Support multiple doctors per clinic
- Verify doctor credentials (optional integration)
- Include clinic address and contact

---

**FR-RX-003**: Patient Profile Management  
WHEN creating prescriptions, THE SYSTEM SHALL capture patient details including name, age, gender, and contact information.

**User Story**: As a doctor, I want to maintain patient records so that I can track treatment history.

**Acceptance Criteria**:
- Store patient name, age, gender, contact
- Assign unique patient ID
- Search patients by name or ID
- Display patient history
- Support patient photo (optional)
- Maintain patient confidentiality

---

**FR-RX-004**: Medicine Database  
WHEN prescribing medications, THE SYSTEM SHALL provide searchable medicine database with generic and brand names.

**User Story**: As a doctor, I want to search medicines quickly so that prescription creation is fast.

**Acceptance Criteria**:
- Comprehensive medicine database
- Search by generic or brand name
- Display medicine composition
- Show available strengths
- Support custom medicine entry
- Update database periodically

---

**FR-RX-005**: Dosage and Frequency Specification  
WHEN prescribing medicines, THE SYSTEM SHALL support standard dosage formats and frequency patterns.

**User Story**: As a doctor, I want to specify dosage clearly so that patients take medicines correctly.

**Acceptance Criteria**:
- Support formats: 1-0-1, 1-1-1, 0-0-1, etc.
- Specify timing: before/after meals, morning/night
- Set duration: days, weeks, months
- Add special instructions
- Support PRN (as needed) medications
- Display dosage clearly on prescription

---

**FR-RX-006**: PDF Prescription Generation  
WHEN prescriptions are finalized, THE SYSTEM SHALL generate professional PDF documents with all details.

**User Story**: As a doctor, I want to provide PDF prescriptions so that patients can print or share them.

**Acceptance Criteria**:
- Generate PDF with doctor and patient details
- Include all prescribed medicines with dosage
- Display clinic logo and letterhead
- Include prescription date and validity
- Add doctor's signature (digital or scanned)
- Optimize PDF size for sharing

---

**FR-RX-007**: Prescription-Invoice Linking  
WHEN pharmacies dispense medicines, THE SYSTEM SHALL link prescriptions to invoices for tracking and compliance.

**User Story**: As a pharmacist, I want to link prescriptions to bills so that I can track medicine sales.

**Acceptance Criteria**:
- Scan or enter prescription ID
- Display prescribed medicines
- Add medicines to invoice
- Mark prescription as fulfilled
- Track partial fulfillment
- Generate prescription-wise sales reports

---

**FR-RX-008**: Prescription History Storage  
WHEN prescriptions are created, THE SYSTEM SHALL maintain complete prescription history per patient for future reference.

**User Story**: As a doctor, I want to see past prescriptions so that I can track treatment progress.

**Acceptance Criteria**:
- Store all prescriptions per patient
- Display chronological prescription history
- Search prescriptions by date or medicine
- Show prescription details on click
- Export prescription history
- Maintain history for minimum 5 years

---

**FR-RX-009**: Digital Signature Support  
WHEN finalizing prescriptions, THE SYSTEM SHALL support digital signatures for legal validity.

**User Story**: As a doctor, I want to sign prescriptions digitally so that they are legally valid.

**Acceptance Criteria**:
- Upload scanned signature image
- Apply signature to prescriptions
- Support digital signature certificates (optional)
- Verify signature authenticity
- Display signature on PDF
- Comply with digital signature regulations

---

**FR-RX-010**: Medical Data Privacy  
WHEN handling prescription data, THE SYSTEM SHALL ensure strict privacy and access controls for sensitive medical information.

**User Story**: As a patient, I want my medical data protected so that my privacy is maintained.

**Acceptance Criteria**:
- Encrypt prescription data at rest and in transit
- Restrict access to authorized doctors only
- Audit all prescription access
- Support patient consent for data sharing
- Allow patient data deletion requests
- Comply with medical data protection laws

---

## 10. Non-Functional Requirements

### 10.1 Performance Requirements

**NFR-PERF-001**: System Uptime  
THE SYSTEM SHALL maintain 99.5% uptime for cloud services, measured monthly, excluding planned maintenance windows.

**Acceptance Criteria**:
- Maximum downtime: 3.6 hours per month
- Scheduled maintenance during low-traffic hours
- Advance notification for planned downtime
- Automatic failover for critical services
- Monitor uptime with alerting

---

**NFR-PERF-002**: Response Time  
THE SYSTEM SHALL complete billing operations within 3 seconds from user action to invoice generation.

**Acceptance Criteria**:
- Voice command processing: < 2 seconds
- Invoice generation: < 1 second
- Product search: < 500ms
- Analytics loading: < 3 seconds
- API response time: < 1 second (95th percentile)

---

**NFR-PERF-003**: Inventory Capacity  
THE SYSTEM SHALL support up to 50,000 SKUs per retailer without performance degradation.

**Acceptance Criteria**:
- Database indexing for fast queries
- Pagination for large product lists
- Efficient search algorithms
- Maintain response times with 50K SKUs
- Support bulk operations on large datasets

---

**NFR-PERF-004**: Concurrent Users  
THE SYSTEM SHALL support 100 concurrent users per retailer account without performance impact.

**Acceptance Criteria**:
- Handle 100 simultaneous API requests
- Maintain response times under load
- Queue requests if necessary
- Prevent race conditions in inventory updates
- Load testing validated

---

**NFR-PERF-005**: Crash Recovery  
WHEN the application crashes, THE SYSTEM SHALL recover gracefully and restore user session without data loss.

**Acceptance Criteria**:
- Auto-save draft bills every 10 seconds
- Restore incomplete transactions on restart
- Preserve user session state
- Display recovery notification
- Log crash details for debugging

---

**NFR-PERF-006**: Low-End Device Support  
THE SYSTEM SHALL function smoothly on Android devices with 2GB RAM running Android 8.0 or higher.

**Acceptance Criteria**:
- App size < 50MB
- Memory usage < 200MB during operation
- Optimize image and asset sizes
- Lazy loading for heavy components
- Test on low-end devices (2GB RAM)
- Smooth UI with 60fps target

---

**NFR-PERF-007**: Portrait Orientation Optimization  
THE SYSTEM SHALL be optimized for portrait orientation on mobile devices for one-handed operation.

**Acceptance Criteria**:
- UI designed for portrait mode
- Key actions reachable with thumb
- Lock orientation to portrait
- Responsive layout for different screen sizes
- Test on 5-6.5 inch screens

---

### 10.2 Scalability Requirements

**NFR-SCALE-001**: Multi-Tenant Architecture  
THE SYSTEM SHALL implement multi-tenant architecture to support millions of independent retailer accounts.

**Acceptance Criteria**:
- Logical data isolation per tenant
- Shared infrastructure with tenant separation
- Tenant-specific configurations
- Prevent cross-tenant data access
- Scale tenants independently

---

**NFR-SCALE-002**: Horizontal Backend Scaling  
THE SYSTEM SHALL support horizontal scaling by adding backend server instances to handle increased load.

**Acceptance Criteria**:
- Stateless API design
- Load balancer distribution
- Auto-scaling based on CPU/memory
- Session management in distributed cache
- Database connection pooling

---

**NFR-SCALE-003**: Database Indexing  
THE SYSTEM SHALL implement appropriate database indexes to maintain query performance as data volume grows.

**Acceptance Criteria**:
- Index on frequently queried fields
- Composite indexes for complex queries
- Regular index maintenance
- Query optimization for large tables
- Monitor slow queries

---

**NFR-SCALE-004**: Million Retailer Support  
THE SYSTEM SHALL be architected to support millions of retailer accounts with billions of transactions.

**Acceptance Criteria**:
- Database sharding strategy
- Distributed caching (Redis)
- CDN for static assets
- Asynchronous processing for heavy tasks
- Microservices architecture (future)

---

**NFR-SCALE-005**: Cloud Deployment Readiness  
THE SYSTEM SHALL be deployable on cloud platforms (AWS, GCP, Azure) with containerization support.

**Acceptance Criteria**:
- Docker containerization
- Kubernetes orchestration support
- Infrastructure as Code (Terraform)
- CI/CD pipeline integration
- Environment-specific configurations

---

### 10.3 Security Requirements

**NFR-SEC-001**: JWT Authentication  
THE SYSTEM SHALL use JSON Web Tokens (JWT) for stateless authentication across all API endpoints.

**Acceptance Criteria**:
- Generate JWT on successful login
- Include user ID, role, expiry in claims
- Sign tokens with secret key (HS256 or RS256)
- Validate token signature on every request
- Reject expired or invalid tokens
- Implement token refresh mechanism

---

**NFR-SEC-002**: Role-Based Authorization  
THE SYSTEM SHALL enforce role-based access control (RBAC) at both API and UI levels.

**Acceptance Criteria**:
- Define permissions per role
- Validate permissions before operation execution
- Return 403 Forbidden for unauthorized access
- Hide unauthorized UI elements
- Log authorization failures
- Support permission inheritance

---

**NFR-SEC-003**: Data Encryption at Rest  
THE SYSTEM SHALL encrypt sensitive data at rest using AES-256 encryption.

**Acceptance Criteria**:
- Encrypt local SQLite database
- Encrypt backup files
- Use Android Keystore for key management
- Encrypt PII fields in cloud database
- Secure key rotation mechanism
- Comply with encryption standards

---

**NFR-SEC-004**: Secure API Endpoints  
THE SYSTEM SHALL protect all API endpoints with authentication, authorization, and input validation.

**Acceptance Criteria**:
- Require JWT token for all protected endpoints
- Validate input parameters (type, length, format)
- Sanitize inputs to prevent injection attacks
- Use HTTPS only (no HTTP)
- Implement CORS policies
- Return generic error messages (no sensitive info)

---

**NFR-SEC-005**: Rate Limiting  
THE SYSTEM SHALL implement rate limiting to prevent abuse and DDoS attacks.

**Acceptance Criteria**:
- Limit API requests per user: 100 requests/minute
- Limit OTP requests: 3 per hour per phone
- Limit login attempts: 5 per hour per account
- Return 429 Too Many Requests when exceeded
- Implement exponential backoff
- Whitelist trusted IPs if needed

---

**NFR-SEC-006**: Audit Logging  
THE SYSTEM SHALL maintain comprehensive audit logs for security-sensitive operations.

**Acceptance Criteria**:
- Log authentication events (login, logout, failures)
- Log authorization failures
- Log data modifications (who, what, when)
- Log configuration changes
- Store logs securely with integrity checks
- Retain logs for minimum 1 year

---

**NFR-SEC-007**: OTP Security  
THE SYSTEM SHALL implement secure OTP generation and validation with expiration and rate limiting.

**Acceptance Criteria**:
- Generate cryptographically random 6-digit OTP
- OTP validity: 5 minutes
- Single-use OTP (invalidate after use)
- Maximum 3 OTP requests per hour
- Prevent OTP enumeration attacks
- Send OTP via secure SMS gateway

---

### 10.4 Compliance Requirements

**NFR-COMP-001**: Indian GST Compliance  
THE SYSTEM SHALL comply with all Indian GST regulations including invoice format, tax calculation, and record retention.

**Acceptance Criteria**:
- Invoice contains all mandatory GST fields
- Correct CGST/SGST/IGST calculation
- Sequential invoice numbering
- HSN code on invoices
- GSTR-1 and GSTR-3B compatible data
- 6-year record retention

---

**NFR-COMP-002**: Data Protection Compliance  
THE SYSTEM SHALL comply with Indian data protection guidelines including consent, access, and deletion rights.

**Acceptance Criteria**:
- Obtain user consent for data collection
- Provide data access to users on request
- Support data deletion requests
- Maintain consent logs
- Encrypt personal data
- Data breach notification process

---

**NFR-COMP-003**: Accessibility Compliance  
THE SYSTEM SHALL follow accessibility best practices to ensure usability for users with disabilities.

**Acceptance Criteria**:
- Sufficient color contrast (WCAG AA)
- Touch targets minimum 48x48 dp
- Screen reader support for critical features
- Text scaling support
- Keyboard navigation (where applicable)
- Alternative text for images

---

**NFR-COMP-004**: Medical Data Compliance (Phase 3)  
WHEN prescription module is active, THE SYSTEM SHALL comply with medical data protection regulations.

**Acceptance Criteria**:
- Encrypt all medical records
- Restrict access to authorized personnel
- Maintain prescription audit trail
- Support patient consent management
- Comply with medical record retention laws
- Implement data anonymization for analytics

---

### 10.5 Usability Requirements

**NFR-USE-001**: Intuitive User Interface  
THE SYSTEM SHALL provide an intuitive, easy-to-learn interface requiring minimal training for new users.

**Acceptance Criteria**:
- New users can create first bill within 5 minutes
- Consistent UI patterns across screens
- Clear labels and instructions
- Contextual help available
- Minimal clicks to complete tasks
- User testing with target audience

---

**NFR-USE-002**: Multi-Language Support  
THE SYSTEM SHALL support Hindi and English languages with easy language switching.

**Acceptance Criteria**:
- Complete UI translation for Hindi and English
- Language selection in settings
- Persist language preference
- Right-to-left text support (if needed)
- Localized date and currency formats
- Cultural appropriateness of content

---

**NFR-USE-003**: Error Handling and Messaging  
THE SYSTEM SHALL provide clear, actionable error messages in user's preferred language.

**Acceptance Criteria**:
- User-friendly error messages (no technical jargon)
- Suggest corrective actions
- Translate errors to selected language
- Display errors prominently but non-intrusively
- Log technical details separately
- Prevent error message information leakage

---

**NFR-USE-004**: Offline Capability Indication  
THE SYSTEM SHALL clearly indicate offline mode status and limitations to users.

**Acceptance Criteria**:
- Display "Offline Mode" badge prominently
- Show last sync timestamp
- Indicate which features are unavailable offline
- Notify when connectivity is restored
- Display pending sync count
- Visual distinction for offline data

---

### 10.6 Reliability Requirements

**NFR-REL-001**: Data Integrity  
THE SYSTEM SHALL ensure data integrity through validation, constraints, and transaction management.

**Acceptance Criteria**:
- Database constraints (foreign keys, unique, not null)
- Input validation at API and UI levels
- ACID transactions for critical operations
- Checksum validation for data transfers
- Regular data integrity audits
- Automated backup verification

---

**NFR-REL-002**: Fault Tolerance  
THE SYSTEM SHALL handle failures gracefully without data loss or corruption.

**Acceptance Criteria**:
- Retry failed operations with exponential backoff
- Fallback mechanisms for external service failures
- Circuit breaker pattern for unstable services
- Graceful degradation of non-critical features
- Automatic recovery from transient failures
- User notification for permanent failures

---

**NFR-REL-003**: Backup and Recovery  
THE SYSTEM SHALL implement automated backup and recovery mechanisms for disaster recovery.

**Acceptance Criteria**:
- Daily automated cloud backups
- Point-in-time recovery capability
- Backup retention: 30 days
- Test recovery procedures quarterly
- Backup encryption
- Recovery time objective (RTO): 4 hours
- Recovery point objective (RPO): 24 hours

---

### 10.7 Maintainability Requirements

**NFR-MAINT-001**: Code Quality  
THE SYSTEM SHALL maintain high code quality through standards, reviews, and automated testing.

**Acceptance Criteria**:
- Follow language-specific style guides
- Code review for all changes
- Automated linting and formatting
- Unit test coverage > 70%
- Integration test coverage for critical paths
- Documentation for complex logic

---

**NFR-MAINT-002**: Logging and Monitoring  
THE SYSTEM SHALL implement comprehensive logging and monitoring for troubleshooting and performance analysis.

**Acceptance Criteria**:
- Structured logging (JSON format)
- Log levels: DEBUG, INFO, WARN, ERROR
- Application performance monitoring (APM)
- Error tracking and alerting
- User analytics (privacy-compliant)
- Dashboard for key metrics

---

**NFR-MAINT-003**: API Versioning  
THE SYSTEM SHALL implement API versioning to support backward compatibility during updates.

**Acceptance Criteria**:
- Version in URL path (e.g., /api/v1/)
- Maintain previous version for 6 months
- Deprecation notices for old versions
- Clear migration documentation
- Automated API documentation (Swagger/OpenAPI)
- Version negotiation support

---

## 11. Integration Requirements

### 11.1 External Service Integrations

**INT-001**: Google Gemini AI Integration  
THE SYSTEM SHALL integrate with Google Gemini AI API for voice command processing and natural language understanding.

**Acceptance Criteria**:
- API key management and rotation
- Request/response handling
- Error handling for API failures
- Rate limit management
- Fallback to cached responses if available
- Monitor API usage and costs

---

**INT-002**: Supabase Database Integration  
THE SYSTEM SHALL use Supabase as the cloud database and authentication provider.

**Acceptance Criteria**:
- PostgreSQL database connection
- Supabase Auth integration
- Real-time subscriptions (optional)
- Row-level security policies
- Connection pooling
- Backup and restore via Supabase

---

**INT-003**: Bluetooth Printer Integration  
THE SYSTEM SHALL integrate with Bluetooth thermal printers using ESC/POS protocol.

**Acceptance Criteria**:
- Bluetooth device discovery and pairing
- ESC/POS command generation
- Support for common printer brands
- Handle printer errors (paper out, low battery)
- Printer status monitoring
- Fallback to PDF if printer unavailable

---

**INT-004**: WhatsApp Integration  
THE SYSTEM SHALL integrate with WhatsApp for invoice sharing via WhatsApp Business API or intent-based sharing.

**Acceptance Criteria**:
- Generate shareable invoice PDF
- Launch WhatsApp with pre-filled message
- Support WhatsApp Business API (future)
- Handle WhatsApp not installed scenario
- Track sharing success/failure
- Respect user privacy

---

**INT-005**: Email Service Integration  
THE SYSTEM SHALL integrate with email service providers for invoice delivery and notifications.

**Acceptance Criteria**:
- SMTP configuration for email sending
- HTML email templates
- Attachment support (PDF invoices)
- Delivery status tracking
- Bounce handling
- Unsubscribe mechanism for marketing emails

---

**INT-006**: Payment Gateway Integration (Future)  
THE SYSTEM SHALL support integration with Indian payment gateways for digital payment collection.

**Acceptance Criteria**:
- Support Razorpay, Paytm, or similar
- UPI payment links
- Payment status webhooks
- Refund processing
- Transaction reconciliation
- PCI DSS compliance considerations

---

**INT-007**: SMS Gateway Integration  
THE SYSTEM SHALL integrate with SMS gateway for OTP delivery and notifications.

**Acceptance Criteria**:
- Reliable SMS delivery
- Delivery status tracking
- Support for Indian mobile numbers
- Template-based messaging
- Cost optimization (DLT registration)
- Fallback to alternative gateway

---

## 12. Data Governance and Privacy

### 12.1 Data Collection and Usage

**DG-001**: Data Minimization  
THE SYSTEM SHALL collect only data necessary for business operations and explicitly consented by users.

**Acceptance Criteria**:
- Document purpose for each data field
- Avoid collecting unnecessary PII
- Obtain explicit consent before collection
- Provide clear privacy policy
- Allow users to opt-out of optional data collection

---

**DG-002**: Data Retention Policy  
THE SYSTEM SHALL define and enforce data retention periods for different data types.

**Acceptance Criteria**:
- Transaction data: 6 years (GST compliance)
- User activity logs: 1 year
- Audit logs: 2 years
- Backup data: 30 days
- Automated deletion after retention period
- Legal hold capability for investigations

---

**DG-003**: User Data Access  
THE SYSTEM SHALL provide users with access to their personal data upon request.

**Acceptance Criteria**:
- Self-service data export feature
- Export in machine-readable format (JSON/CSV)
- Include all personal data
- Deliver within 30 days of request
- Verify user identity before providing data
- Log all data access requests

---

**DG-004**: Right to Deletion  
THE SYSTEM SHALL support user requests for data deletion (right to be forgotten) subject to legal retention requirements.

**Acceptance Criteria**:
- Self-service account deletion option
- Delete personal data within 30 days
- Retain data required for legal compliance
- Anonymize data instead of deletion where appropriate
- Notify user of deletion completion
- Prevent data recovery after deletion

---

**DG-005**: Data Breach Response  
THE SYSTEM SHALL have documented procedures for detecting, responding to, and reporting data breaches.

**Acceptance Criteria**:
- Automated breach detection mechanisms
- Incident response plan documented
- Notify affected users within 72 hours
- Report to authorities as required
- Remediation and prevention measures
- Post-incident review and improvements

---

### 12.2 Data Security

**DG-006**: Encryption Standards  
THE SYSTEM SHALL use industry-standard encryption for data at rest and in transit.

**Acceptance Criteria**:
- AES-256 for data at rest
- TLS 1.3 for data in transit
- Secure key management
- Regular security audits
- Vulnerability scanning
- Penetration testing annually

---

**DG-007**: Access Control  
THE SYSTEM SHALL implement strict access controls for sensitive data.

**Acceptance Criteria**:
- Principle of least privilege
- Multi-factor authentication for admin access
- Regular access reviews
- Revoke access immediately on termination
- Audit all data access
- Separate production and development environments

---

## 13. Dependencies

### 13.1 Technology Stack Dependencies

**DEP-001**: Flutter Framework  
- **Version**: Flutter 3.x or higher
- **Purpose**: Cross-platform mobile app development
- **Critical**: Yes
- **Alternatives**: React Native, Native Android

**DEP-002**: FastAPI Framework  
- **Version**: FastAPI 0.100+ with Python 3.11+
- **Purpose**: Backend REST API development
- **Critical**: Yes
- **Alternatives**: Django REST Framework, Flask

**DEP-003**: PostgreSQL Database  
- **Version**: PostgreSQL 14+ via Supabase
- **Purpose**: Relational data storage
- **Critical**: Yes
- **Alternatives**: MySQL, MongoDB (not recommended for transactional data)

**DEP-004**: Google Gemini AI  
- **Version**: Latest Gemini API
- **Purpose**: Voice command processing and NLU
- **Critical**: Yes (for voice billing)
- **Alternatives**: OpenAI GPT, Azure OpenAI, local models

**DEP-005**: Supabase Platform  
- **Version**: Latest
- **Purpose**: Database hosting, authentication, real-time features
- **Critical**: Yes
- **Alternatives**: Firebase, AWS Amplify, self-hosted PostgreSQL

---

### 13.2 Third-Party Service Dependencies

**DEP-006**: SMS Gateway  
- **Purpose**: OTP delivery
- **Critical**: Yes
- **Providers**: Twilio, MSG91, AWS SNS

**DEP-007**: Cloud Storage  
- **Purpose**: Invoice PDFs, backups, images
- **Critical**: No (can use local storage initially)
- **Providers**: AWS S3, Google Cloud Storage, Supabase Storage

**DEP-008**: Email Service  
- **Purpose**: Invoice delivery, notifications
- **Critical**: No
- **Providers**: SendGrid, AWS SES, Mailgun

**DEP-009**: Analytics Service  
- **Purpose**: User behavior tracking, crash reporting
- **Critical**: No
- **Providers**: Firebase Analytics, Mixpanel, Sentry

---

### 13.3 Hardware Dependencies

**DEP-010**: Android Device  
- **Minimum**: Android 8.0 (API level 26), 2GB RAM
- **Recommended**: Android 10+, 4GB RAM
- **Critical**: Yes

**DEP-011**: Bluetooth Thermal Printer  
- **Requirement**: ESC/POS compatible, Bluetooth 4.0+
- **Critical**: No (PDF export alternative)
- **Brands**: Epson, Star Micronics, Zebra, generic Chinese printers

**DEP-012**: Internet Connectivity  
- **Requirement**: Periodic connectivity for sync
- **Critical**: No (offline mode available)
- **Minimum**: 2G/3G for basic operations

---

## 14. Future Roadmap

### 14.1 Phase 2: B2B Direct Commerce Engine (Q3-Q4 2026)

**Objective**: Transform SnapBill into a B2B procurement marketplace connecting retailers with suppliers.

**Key Features**:
- AI-powered demand forecasting
- Supplier marketplace with ratings
- Bulk ordering and pricing
- Purchase order management
- Demand aggregation for better pricing
- Automated inventory updates on delivery
- Supplier analytics and insights

**Success Metrics**:
- 10,000+ retailers using B2B features
- ₹100 crore GMV in first year
- 500+ verified suppliers onboarded
- 30% cost savings for retailers

---

### 14.2 Phase 3: Digital Prescription Mode (Q1-Q2 2027)

**Objective**: Extend SnapBill into healthcare sector with digital prescription management.

**Key Features**:
- Structured prescription creation for doctors
- Medicine database with dosage templates
- PDF prescription generation
- Prescription-invoice linking for pharmacies
- Patient history tracking
- Digital signature support
- Medical data privacy compliance

**Success Metrics**:
- 1,000+ doctors using prescription module
- 5,000+ pharmacies integrated
- 100,000+ digital prescriptions generated
- Zero prescription-related medication errors reported

---

### 14.3 Additional Future Enhancements

**FE-001**: iOS Application  
- Native iOS app for Apple device users
- Timeline: Q2 2027

**FE-002**: Web Dashboard  
- Desktop web interface for detailed analytics and management
- Timeline: Q4 2026

**FE-003**: Multi-Store Management  
- Manage multiple store locations from single account
- Timeline: Q3 2026

**FE-004**: Employee Attendance and Payroll  
- Track employee attendance and manage payroll
- Timeline: Q1 2027

**FE-005**: Customer Loyalty Program  
- Points-based loyalty and rewards system
- Timeline: Q4 2026

**FE-006**: Advanced Analytics with ML  
- Predictive analytics for sales forecasting
- Churn prediction and prevention
- Timeline: Q2 2027

**FE-007**: Barcode Scanner Integration  
- Camera-based barcode scanning for quick product addition
- Timeline: Q3 2026

**FE-008**: Multi-Currency Support  
- Support for international transactions
- Timeline: Q1 2028

**FE-009**: Offline Voice Billing  
- On-device voice processing without internet
- Timeline: Q4 2027

**FE-010**: Integration Marketplace  
- Third-party integrations (accounting software, CRM, etc.)
- Timeline: Q2 2028

---

## 15. Success Criteria

### 15.1 Phase 1 Success Metrics

**Business Metrics**:
- 10,000 active retailers within 6 months of launch
- 1 million transactions processed in first year
- 80% user retention rate (monthly active users)
- Average 50 transactions per retailer per day
- 4.5+ star rating on Google Play Store

**Technical Metrics**:
- 99.5% system uptime
- < 3 second average response time for billing
- < 0.1% transaction failure rate
- Zero critical security incidents
- < 5% crash rate

**User Satisfaction Metrics**:
- 85%+ user satisfaction score
- < 10% support ticket rate
- 70%+ feature adoption rate (voice billing)
- 90%+ invoice accuracy rate
- 60%+ users recommend to others (NPS > 50)

---

## 16. Risks and Mitigation

### 16.1 Technical Risks

**RISK-001**: Voice Recognition Accuracy  
- **Risk**: Low accuracy in noisy retail environments
- **Impact**: High - Core feature unusability
- **Mitigation**: 
  - Implement noise cancellation
  - Provide manual fallback option
  - Continuous model training with real data
  - User feedback loop for corrections

**RISK-002**: Offline Sync Conflicts  
- **Risk**: Data conflicts when multiple devices sync
- **Impact**: Medium - Data integrity issues
- **Mitigation**:
  - Implement robust conflict resolution
  - Use timestamp-based priority
  - Maintain audit trail
  - Alert users of critical conflicts

**RISK-003**: Third-Party API Dependency  
- **Risk**: Google Gemini API downtime or rate limits
- **Impact**: High - Voice billing unavailable
- **Mitigation**:
  - Implement caching and fallback
  - Queue requests during outages
  - Consider backup AI provider
  - Graceful degradation to manual entry

**RISK-004**: Database Performance  
- **Risk**: Slow queries with large datasets
- **Impact**: Medium - Poor user experience
- **Mitigation**:
  - Implement proper indexing
  - Query optimization
  - Database sharding if needed
  - Regular performance monitoring

---

### 16.2 Business Risks

**RISK-005**: Low User Adoption  
- **Risk**: Retailers prefer traditional methods
- **Impact**: High - Business failure
- **Mitigation**:
  - Extensive user research and testing
  - Gradual feature rollout
  - Strong onboarding and training
  - Incentive programs for early adopters

**RISK-006**: Competition  
- **Risk**: Established POS systems in market
- **Impact**: Medium - Market share challenges
- **Mitigation**:
  - Focus on unique features (voice, offline, AI)
  - Competitive pricing
  - Superior user experience
  - Strong customer support

**RISK-007**: Regulatory Changes  
- **Risk**: GST regulations change
- **Impact**: Medium - Compliance issues
- **Mitigation**:
  - Monitor regulatory updates
  - Flexible architecture for changes
  - Quick update deployment capability
  - Legal consultation

---

### 16.3 Security Risks

**RISK-008**: Data Breach  
- **Risk**: Unauthorized access to sensitive data
- **Impact**: Critical - Legal and reputation damage
- **Mitigation**:
  - Strong encryption (AES-256, TLS 1.3)
  - Regular security audits
  - Penetration testing
  - Incident response plan
  - Compliance with data protection laws

**RISK-009**: Payment Fraud  
- **Risk**: Fraudulent transactions or payment manipulation
- **Impact**: High - Financial loss
- **Mitigation**:
  - Transaction audit trail
  - Role-based access control
  - Anomaly detection
  - Regular reconciliation
  - Insurance coverage

---

## 17. Assumptions and Constraints Summary

### 17.1 Key Assumptions
1. Retailers have basic smartphone literacy
2. Internet connectivity available periodically (not continuous)
3. Retailers willing to adopt digital billing
4. Google Gemini API remains available and affordable
5. GST regulations remain stable
6. Bluetooth printers widely available in market
7. Target market has Android device penetration
8. Retailers maintain accurate product catalogs

### 17.2 Key Constraints
1. Android-only platform (Phase 1)
2. Portrait orientation only
3. Minimum Android 8.0 requirement
4. Dependent on third-party AI service
5. Limited by device storage capacity
6. Bluetooth range limitations for printing
7. Voice recognition accuracy limitations
8. Network bandwidth constraints in rural areas

---

## 18. Acceptance and Sign-Off

### 18.1 Document Review

This requirements document must be reviewed and approved by:

- **Product Owner**: _____________________ Date: _______
- **Technical Lead**: _____________________ Date: _______
- **Business Stakeholder**: _____________________ Date: _______
- **Compliance Officer**: _____________________ Date: _______
- **Security Lead**: _____________________ Date: _______

### 18.2 Change Management

Any changes to this requirements document must follow the change control process:

1. Submit change request with justification
2. Impact analysis (technical, business, timeline)
3. Stakeholder review and approval
4. Update document with version increment
5. Communicate changes to all stakeholders
6. Update related design and implementation documents

### 18.3 Document Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-15 | SnapBill Team | Initial draft - Complete SRS |

---

## 19. Appendices

### Appendix A: Glossary of Technical Terms

- **API**: Application Programming Interface
- **CRUD**: Create, Read, Update, Delete operations
- **ESC/POS**: Printer command language standard
- **JWT**: JSON Web Token for authentication
- **NLU**: Natural Language Understanding
- **OTP**: One-Time Password
- **POS**: Point of Sale
- **RAG**: Retrieval-Augmented Generation
- **RBAC**: Role-Based Access Control
- **REST**: Representational State Transfer
- **SKU**: Stock Keeping Unit
- **TLS**: Transport Layer Security

### Appendix B: Reference Documents

1. Indian GST Act and Rules
2. Flutter Documentation: https://flutter.dev/docs
3. FastAPI Documentation: https://fastapi.tiangolo.com
4. Google Gemini AI API Documentation
5. Supabase Documentation: https://supabase.com/docs
6. ESC/POS Command Reference
7. Indian Data Protection Guidelines

### Appendix C: Contact Information

**Product Team**:
- Product Manager: [Contact Details]
- Technical Lead: [Contact Details]
- Business Analyst: [Contact Details]

**Development Team**:
- Mobile Lead: [Contact Details]
- Backend Lead: [Contact Details]
- QA Lead: [Contact Details]

**Support**:
- Email: support@snapbill.in
- Phone: [Contact Number]
- Website: www.snapbill.in

---

**END OF DOCUMENT**

---

*This Software Requirements Specification (SRS) document is confidential and proprietary to SnapBill. Unauthorized distribution or reproduction is prohibited.*
