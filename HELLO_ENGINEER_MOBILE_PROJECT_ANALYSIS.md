# HELLO ENGINEER — COMPLETE EXISTING PROJECT ANALYSIS
**Technical Architecture, Systems Audit, Gap Analysis & Mobile Expansion Blueprint**

---

## 1. Executive Summary

This document presents a comprehensive, production-grade architectural and systems analysis of the **Hello Engineer** platform. The analysis was conducted directly against the active codebase located at `c:/Users/kille/Downloads/hello-engineer/` to establish the baseline for extending the platform into a complete multi-tier service management ecosystem comprising:

1. **Customer Mobile Application** (React Native with Expo)
2. **Admin Web Dashboard** (Built into the existing web application)
3. **Technician Mobile Application** (React Native with Expo)
4. **Unified Backend & MySQL Database** (Shared REST API + Real-time Push Notifications via FCM)

### Key Architectural Findings

1. **Frontend Architecture**: The existing frontend is a modern **Single Page Application (SPA)** built with **React 18.3.1**, **Vite 5.4.6**, **React Router DOM 6.26.2**, and a custom design system styled with bespoke CSS (`style.css` ~129 KB) augmented with Bootstrap 5.3.3 utilities and Bootstrap Icons.
2. **Current Platform Scope**: The existing website functions primarily as a **public-facing engineering showcase, service catalog, and lead generation/enquiry portal**. It contains rich UI pages for Home, All Services, AMC & Maintenance, Projects, About, Contact, and an intelligent interactive Rule-Based Chatbot (`useChatbot.js`).
3. **Absence of Customer Accounts & Authentication**: There is **no authentication system** currently active. Customers do not register or log in; all bookings submitted through modals, contact forms, or the chatbot are anonymous lead submissions.
4. **Absence of Admin Dashboard UI**: There is **no existing Admin Dashboard frontend interface**. The route table in [App.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/App.jsx) contains only public marketing and enquiry pages.
5. **Absence of Technician Management**: There are **no technician database tables, no technician credentials, no technician assignment workflows, no job status tracking, and no attendance tracking**.
6. **Critical Database & Backend Finding**: While a comprehensive relational SQL schema is written in [schema.sql](file:///c:/Users/kille/Downloads/hello-engineer/database/schema.sql), the active Node.js backend in [db.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/config/db.js) runs in **mock/in-memory mode** (`isMockMode: () => true`) using an in-memory object (`mockDb`). The `mysql2` driver is not wired into `backend/package.json`. Data submitted through the website is temporarily held in RAM and emailed via Nodemailer; it does not persist across backend restarts.
7. **Absence of File & Image Uploads**: No file handling library (such as `multer`) is installed in the backend, and no file input exists anywhere in the frontend. Problem images, before-service photos, and after-service photos cannot currently be uploaded or stored.
8. **Reusability & Migration Path**: The existing service catalog data, enquiry intake logic, email templates, and styling design tokens are high quality and can be preserved. A clean, non-disruptive migration path is viable: connect the existing Express backend to MySQL, add JWT authentication with role-based access control (RBAC), implement Multer for media uploads, build the Admin Dashboard into the existing React frontend, and build the React Native Expo mobile apps in a new `mobile/` directory sharing the same backend.

---

## 2. Existing Project Architecture

### 2.1 High-Level Component Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                        CLIENT / USER TIERS                             │
│                                                                        │
│   ┌───────────────────────────┐      ┌─────────────────────────────┐   │
│   │   Existing React Web SPA  │      │  Proposed React Native Apps │   │
│   │  (Vite + Bootstrap Icons) │      │  (Customer & Technician)    │   │
│   └─────────────┬─────────────┘      └──────────────┬──────────────┘   │
└─────────────────┼───────────────────────────────────┼──────────────────┘
                  │ HTTP / REST                       │ HTTP / REST + FCM
                  ▼                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        BACKEND SERVER TIER                             │
│                                                                        │
│   ┌────────────────────────────────────────────────────────────────┐   │
│   │                    Node.js + Express.js API                    │   │
│   │                 (backend/server.js, Port 5000)                 │   │
│   │                                                                │   │
│   │  • Middleware: CORS, express.json, validation, errorHandler   │   │
│   │  • Controllers: enquiry, service, project, maintenance        │   │
│   │  • Utilities: emailService (Nodemailer via SMTP)               │   │
│   └───────────────┬────────────────────────────────┬───────────────┘   │
└───────────────────┼────────────────────────────────┼───────────────────┘
                    │                                │
                    ▼ (Current: In-Memory)           ▼ (Existing Schema)
┌────────────────────────────────────┐ ┌─────────────────────────────────┐
│     backend/config/db.js           │ │      database/schema.sql        │
│   In-Memory Mock Store (mockDb)    │ │    MySQL Database (Unconnected) │
│ • Service Categories (14 items)    │ │ • service_categories (18 items) │
│ • Services Catalog (7 items)       │ │ • services (60+ items)          │
│ • Projects (5 items)               │ │ • projects (5 items)            │
│ • Enquiries & Maintenance (RAM)    │ │ • enquiries & maintenance_enq   │
│                                    │ │ • users (Skeleton table)        │
└────────────────────────────────────┘ └─────────────────────────────────┘
```

### 2.2 Technologies and Libraries in Use

| Tier | Component | Technology / Library | Version | Location / Config File |
|---|---|---|---|---|
| **Frontend** | Build Tool & Bundler | Vite | `^5.4.6` | [vite.config.js](file:///c:/Users/kille/Downloads/hello-engineer/frontend/vite.config.js) |
| **Frontend** | UI Framework | React & React DOM | `^18.3.1` | [package.json](file:///c:/Users/kille/Downloads/hello-engineer/frontend/package.json) |
| **Frontend** | Client-Side Routing | React Router DOM | `^6.26.2` | [App.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/App.jsx) |
| **Frontend** | Iconography | Bootstrap Icons | `^1.11.3` | [main.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/main.jsx) |
| **Frontend** | CSS Framework & Fonts | Bootstrap 5.3.3 (CDN) + Google Fonts (`Outfit`, `Plus Jakarta Sans`) | 5.3.3 | [index.html](file:///c:/Users/kille/Downloads/hello-engineer/frontend/index.html) |
| **Frontend** | Bespoke Styling | Custom Design System CSS | Bespoke | [style.css](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/styles/style.css), [responsive.css](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/styles/responsive.css) |
| **Backend** | Runtime Environment | Node.js | v16+ compatible | [server.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/server.js) |
| **Backend** | Web Framework | Express.js | `^4.19.2` | [package.json](file:///c:/Users/kille/Downloads/hello-engineer/backend/package.json) |
| **Backend** | Cross-Origin Handling | CORS | `^2.8.5` | [server.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/server.js) |
| **Backend** | Environment Config | dotenv | `^16.4.5` | [server.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/server.js) |
| **Backend** | Email Dispatcher | Nodemailer | `^10.0.10` | [emailService.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/utils/emailService.js) |
| **Database** | SQL Engine Target | MySQL 5.7+ / 8.0+ / MariaDB | N/A | [schema.sql](file:///c:/Users/kille/Downloads/hello-engineer/database/schema.sql) |
| **Database** | Active Data Store | In-Memory Object (`mockDb`) | Custom JS | [db.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/config/db.js) |

### 2.3 Frontend, Backend, and Database Communication Flow

1. **Client-to-Backend Transport**:
   - The frontend communicates with the backend exclusively via native browser `fetch()` API calls defined in [api.js](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/utils/api.js).
   - In development mode (when accessing `localhost` or `127.0.0.1`), the frontend targets the relative base path `/api` (proxied by Vite to `http://localhost:5000`).
   - In production, it targets `VITE_BACKEND_URL` (configured as `https://backend.helloengineer.in/api`).
2. **Current Data Flow**:
   - **Step 1**: User enters details into a booking modal, contact form, or chatbot dialog.
   - **Step 2**: Frontend sends a JSON payload via `POST /api/enquiries` or `POST /api/contact`.
   - **Step 3**: Express routes the request through [validation.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/middleware/validation.js) to validate the name, Indian mobile number (`/^(\+91|0)?[6-9]\d{9}$/`), and email.
   - **Step 4**: [enquiryController.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/controllers/enquiryController.js) pushes the record into `mockDb.enquiries` array.
   - **Step 5**: [emailService.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/utils/emailService.js) asynchronously formats a rich HTML email and dispatches it via Nodemailer to the configured destination mailbox.
   - **Step 6**: Backend returns HTTP 201 with `{ success: true, message: "...", data: { enquiry_id: ... } }`.
3. **Database Communication Status**:
   - The database does **not** currently communicate with the backend. [db.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/config/db.js) exports `getPool: () => null` and `isMockMode: () => true`.
   - The SQL schema in [schema.sql](file:///c:/Users/kille/Downloads/hello-engineer/database/schema.sql) is completely decoupled from runtime execution.

---

## 3. Existing Frontend Structure

### 3.1 Folder Hierarchy

```
frontend/
├── index.html                    # Root HTML template (loads Bootstrap CSS & Google Fonts)
├── vite.config.js                # Vite build config with /api proxy to localhost:5000
├── package.json                  # React 18, React Router 6, Bootstrap Icons, Vite
├── public/                       # Favicon and static branding assets
├── build/                        # Production build artifacts (compiled output)
└── src/
    ├── main.jsx                  # Entry point (ReactDOM.createRoot, Context Providers)
    ├── App.jsx                   # Central routing table (11 pages + modals + floating dock + chatbot)
    ├── components/
    │   ├── common/
    │   │   ├── Navbar.jsx        # Sticky top header, logo, navigation links, location selector button
    │   │   ├── Footer.jsx        # Comprehensive footer with links, copyright, contact info
    │   │   ├── BookingModal.jsx  # Primary service/AMC booking modal dialog
    │   │   ├── ContactModal.jsx  # Quick enquiry & callback modal dialog
    │   │   ├── LocationModal.jsx # District selection modal (Tamil Nadu & Bangalore)
    │   │   ├── FloatingDock.jsx  # Floating quick-dial (Phone call & WhatsApp links)
    │   │   └── Toast.jsx         # Custom global toast notification feedback UI
    │   └── chatbot/              # Standalone intelligent chatbot subsystem (18 files)
    │       ├── Chatbot.jsx       # Chatbot shell wrapper
    │       ├── ChatbotLauncher.jsx # Floating bot trigger pill
    │       ├── ChatbotPanel.jsx  # Main floating conversational window
    │       ├── ChatHeader.jsx    # Bot header with status indicator & close button
    │       ├── ChatMessages.jsx  # Message list container
    │       ├── ChatMessage.jsx   # Individual message component (assistant/customer)
    │       ├── ChatInput.jsx     # Text input field & submit button
    │       ├── BookingSummaryCard.jsx # Interactive summary card before bot submission
    │       ├── CategorySelector.jsx  # Category selection buttons
    │       ├── ServiceSelector.jsx   # Service selection buttons
    │       ├── LocationSelector.jsx  # District selection buttons inside chat
    │       ├── IssueChips.jsx        # Common issue suggestion chips
    │       ├── AMCSpaceSelector.jsx  # Space type selector for AMC
    │       ├── SupportCards.jsx      # Helpdesk contact cards
    │       ├── EditMenuSelector.jsx  # State correction selector
    │       ├── nlpEngine.js          # Client-side intent classifier & regex entity extractor
    │       ├── useChatbot.js         # Chatbot state machine & conversation manager (~38 KB)
    │       └── chatbot.css           # Chatbot specific stylesheet (~25 KB)
    ├── context/
    │   ├── LocationContext.jsx   # Manages active district, persists to localStorage
    │   └── ModalContext.jsx      # Manages booking, contact, location modals & toast alerts
    ├── data/
    │   ├── servicesData.js       # Hardcoded 10 categories, 100+ subservices, rate cards
    │   ├── projectsData.js       # Hardcoded case studies & statistics
    │   └── districts.js          # Tamil Nadu districts + Bangalore
    ├── pages/
    │   ├── HomePage.jsx          # Hero, category cards, most booked, why choose us
    │   ├── AllServicesPage.jsx   # Full service catalog with real-time text filter
    │   ├── AmcPage.jsx           # AMC plans, equipment vs facility care, workflow stepper
    │   ├── ProjectsPage.jsx      # Signature engineering case studies & metrics
    │   ├── AboutPage.jsx         # Company background, vision, engineering pillars
    │   ├── ContactPage.jsx       # Dispatch center details & contact form
    │   ├── PrivacyPolicyPage.jsx # Legal policy
    │   ├── TermsPage.jsx         # Terms & conditions
    │   ├── CancellationPage.jsx  # Cancellation and refund policy
    │   ├── WarrantyPage.jsx      # Warranty terms
    │   └── DisclaimerPage.jsx    # Disclaimer policy
    ├── styles/
    │   ├── style.css             # Main bespoke stylesheet (~129 KB)
    │   └── responsive.css        # Breakpoint overrides (~6.6 KB)
    └── utils/
        └── api.js                # API client functions (submitEnquiry, submitContactForm)
```

### 3.2 State Management and Context

The frontend uses two clean React Context providers wrapped in [main.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/main.jsx):
1. **`LocationContext`** ([LocationContext.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/context/LocationContext.jsx)):
   - Stores `currentLocation` (defaults to `'Chennai'`).
   - Syncs automatically to browser `localStorage` (`hello_location`).
   - Powers the district selector modal across all pages and booking flows.
2. **`ModalContext`** ([ModalContext.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/context/ModalContext.jsx)):
   - Coordinates visibility for `BookingModal`, `ContactModal`, and `Toast`.
   - Holds preselected service categories, subservices, and icons when a user clicks "Book Now" anywhere on the site.

---

## 4. Existing Backend Structure

### 4.1 Folder Hierarchy

```
backend/
├── server.js                     # Express app initialization, static serving, route mounting
├── package.json                  # Dependencies: cors, dotenv, express, nodemailer
├── config/
│   └── db.js                     # In-memory mock store (mockDb) with sample categories & services
├── controllers/
│   ├── enquiryController.js      # Handles POST /api/enquiries and GET /api/enquiries
│   ├── maintenanceController.js  # Handles POST /api/enquiries/maintenance
│   ├── serviceController.js      # Handles GET /api/categories and GET /api/services
│   └── projectController.js      # Handles GET /api/projects
├── routes/
│   ├── enquiryRoutes.js          # Routes for enquiries and maintenance
│   ├── serviceRoutes.js          # Routes for categories and services
│   ├── projectRoutes.js          # Route for featured projects
│   └── contactRoutes.js          # Route for contact form submissions
├── middleware/
│   ├── validation.js             # Form validation (name length, 10-digit mobile, email format)
│   └── errorHandler.js           # Global Express error handler returning JSON errors
└── utils/
    └── emailService.js           # Nodemailer HTML email generator and SMTP transporter
```

### 4.2 Server Entry Point Analysis

In [server.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/server.js):
- Listens on `0.0.0.0:PORT` (default 5000).
- Applies `cors()`, `express.json()`, and `express.urlencoded({ extended: true })`.
- Implements dual static serving: checks for `../frontend/build`, `../frontend/dist`, or raw `../frontend`.
- Implements an SPA fallback (`app.get('*')`) that returns `index.html` for non-API client routes.
- Central error handler via [errorHandler.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/middleware/errorHandler.js).

---

## 5. Existing Database Structure

### 5.1 Existing Schema (`database/schema.sql`) Analysis

The database script located at [schema.sql](file:///c:/Users/kille/Downloads/hello-engineer/database/schema.sql) defines 6 tables:

```sql
CREATE DATABASE IF NOT EXISTS `hello_engineer` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE `hello_engineer`;
```

#### 1. `service_categories`
- **Columns**: `id` (INT PK AI), `slug` (VARCHAR(100) UNIQUE), `name` (VARCHAR(150)), `heading` (VARCHAR(200)), `description` (TEXT), `icon` (VARCHAR(100)), `display_order` (INT), `created_at` (TIMESTAMP), `updated_at` (TIMESTAMP).
- **Primary Key**: `id`
- **Foreign Keys**: None
- **Purpose**: Stores top-level engineering disciplines (18 seeded categories: Appliance Services, Electrical, Civil, Mechanical, HVAC, Plumbing, IT & Networking, Telecom, Automation, Electronics, Fire Safety, AV Systems, Energy, Professional Services, etc.).
- **Usage**: Seeded in SQL and duplicated in `mockDb.service_categories`.

#### 2. `services`
- **Columns**: `id` (INT PK AI), `category_id` (INT FK), `name` (VARCHAR(150)), `slug` (VARCHAR(150)), `description` (TEXT), `starting_price` (DECIMAL(10,2)), `is_popular` (BOOLEAN), `is_most_booked` (BOOLEAN), `is_active` (BOOLEAN), `created_at` (TIMESTAMP), `updated_at` (TIMESTAMP).
- **Primary Key**: `id`
- **Foreign Key**: `category_id` REFERENCES `service_categories(id)` ON DELETE CASCADE
- **Purpose**: Stores individual subservices with base pricing (e.g., AC Service ₹499, Washing Machine ₹399).
- **Usage**: Seeded in SQL (60+ items), 7 items in `mockDb.services`.

#### 3. `projects`
- **Columns**: `id` (INT PK AI), `title` (VARCHAR(200)), `category` (VARCHAR(100)), `location` (VARCHAR(150)), `description` (TEXT), `image` (VARCHAR(255)), `is_featured` (BOOLEAN), `display_order` (INT), `created_at` (TIMESTAMP), `updated_at` (TIMESTAMP).
- **Primary Key**: `id`
- **Foreign Keys**: None
- **Purpose**: Showcases completed landmark commercial, industrial, and residential projects.
- **Usage**: 5 seeded projects (Business Park, Manufacturing Plant, Hospital, Luxury Apartments, Water Treatment Plant).

#### 4. `enquiries`
- **Columns**: `id` (INT PK AI), `name` (VARCHAR(150)), `mobile` (VARCHAR(20)), `email` (VARCHAR(150)), `service` (VARCHAR(150)), `location` (VARCHAR(150)), `requirement` (TEXT), `enquiry_type` (VARCHAR(50) DEFAULT 'general'), `status` (VARCHAR(50) DEFAULT 'pending'), `ip_address` (VARCHAR(45)), `created_at` (TIMESTAMP), `updated_at` (TIMESTAMP).
- **Primary Key**: `id`
- **Foreign Keys**: None
- **Purpose**: Stores incoming general enquiries, callbacks, and booking leads.
- **Usage**: Defined in schema, but backend inserts into `mockDb.enquiries` array in RAM.

#### 5. `maintenance_enquiries`
- **Columns**: `id` (INT PK AI), `name` (VARCHAR(150)), `mobile` (VARCHAR(20)), `email` (VARCHAR(150)), `service` (VARCHAR(150)), `location` (VARCHAR(150)), `maintenance_type` (VARCHAR(100)), `space_type` (VARCHAR(100)), `requirement` (TEXT), `status` (VARCHAR(50) DEFAULT 'pending'), `created_at` (TIMESTAMP), `updated_at` (TIMESTAMP).
- **Primary Key**: `id`
- **Foreign Keys**: None
- **Purpose**: Stores structured AMC and periodic maintenance enquiries.
- **Usage**: Defined in schema, but backend inserts into `mockDb.maintenance_enquiries` array in RAM.

#### 6. `users`
- **Columns**: `id` (INT PK AI), `name` (VARCHAR(150)), `email` (VARCHAR(150) UNIQUE), `mobile` (VARCHAR(20)), `password_hash` (VARCHAR(255)), `role` (VARCHAR(50) DEFAULT 'customer'), `created_at` (TIMESTAMP), `updated_at` (TIMESTAMP).
- **Primary Key**: `id`
- **Foreign Keys**: None
- **Purpose**: Skeleton table annotated in SQL with `-- 6. Users Table (for future portal/admin capabilities)`.
- **Usage**: Never referenced by any backend controller or route.

### 5.2 Critical Gap: Missing Database Requirements

The existing schema lacks all operational tables required for mobile apps, technician tracking, and an interactive admin dashboard:

1. **`customers` table missing**: No customer profiles, addresses, or block/unblock flags.
2. **`customer_addresses` table missing**: No saved addresses or GPS coordinates.
3. **`technicians` table missing**: No technician profiles, skillsets, active status, or block status.
4. **`technician_attendance` table missing**: No On Duty / Off Duty records or daily locking.
5. **`bookings` table missing**: Existing `enquiries` table is an unnormalized text-lead table. There is no structured booking lifecycle table with customer reference, technician reference, status enums, scheduled dates, pricing quotes, or completion records.
6. **`service_photos` table missing**: No storage for problem images, before-service photos, or after-service photos.
7. **`booking_status_logs` table missing**: No audit trail of job status transitions.
8. **`fcm_tokens` table missing**: No push notification token registry.
9. **`notifications` table missing**: No in-app notification center table.

---

## 6. Customer System Analysis

Evaluation of customer features specified for the proposed system:

| Customer Feature | Classification | Existing Files / Endpoints / Tables | Gap & Detailed Analysis |
|---|---|---|---|
| **Customer Registration** | **Not Implemented** | None | No register screen, no controller, no DB insert. No self-registration or SMS OTP exists. |
| **Customer Login** | **Not Implemented** | None | No login screen, no auth controller, no JWT or session mechanism exists. |
| **Customer Profile** | **Not Implemented** | None | No profile screen, no GET profile endpoint, no user profile data model exists. |
| **Profile Image** | **Not Implemented** | None | No avatar storage, no file upload endpoint, no DB avatar column exists. |
| **Customer Name & Phone** | **Partially Implemented** | [BookingModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/BookingModal.jsx), [validation.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/middleware/validation.js), `enquiries` table | Name & phone are collected as transient fields in booking/contact forms, but are not linked to a persisted user account. |
| **Saved Addresses** | **Not Implemented** | [BookingModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/BookingModal.jsx) | Address is typed as free text per booking modal. No saved addresses list, default address, or address management exists. |
| **Service Booking** | **Partially Implemented** | [BookingModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/BookingModal.jsx), `POST /api/enquiries`, `mockDb.enquiries` | Exists as an anonymous lead form submission. Does not generate a confirmed order, trackable ID, or technician assignment. |
| **Maintenance Booking** | **Partially Implemented** | [AmcPage.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/pages/AmcPage.jsx), `POST /api/enquiries/maintenance`, `mockDb.maintenance_enquiries` | Exists as an enquiry lead submission for 4 space care domains. Does not create an active maintenance contract. |
| **AMC Plans & Requests** | **Partially Implemented** | [AmcPage.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/pages/AmcPage.jsx), `POST /api/enquiries/maintenance` | Marketing cards and request modal exist. No subscription tracking, period duration, or AMC visit schedules exist. |
| **Problem Description** | **Partially Implemented** | [ContactModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/ContactModal.jsx), [useChatbot.js](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/chatbot/useChatbot.js), field `requirement` | Captured in Chatbot and Contact forms as text. Missing from standard [BookingModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/BookingModal.jsx). |
| **Problem Image Upload** | **Not Implemented** | None | No file input in UI, no multipart parsing, no local/cloud image storage exists. |
| **Visit Scheduling** | **Partially Implemented** | [enquiryController.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/controllers/enquiryController.js) (lines 30-31), [emailService.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/utils/emailService.js) | Backend controller accepts `preferred_date` and `preferred_time` if passed, but frontend modals have no date/time picker inputs. |
| **Alternative Contact Details** | **Not Implemented** | None | No fields for alternative contact name, phone, or address exist in frontend, backend, or database. |
| **Booking History** | **Not Implemented** | None | No customer dashboard, no GET `/api/my-bookings` endpoint, no customer booking query exists. |
| **Booking Status Tracking** | **Not Implemented** | None | Status is hardcoded to `'pending'` in backend. Customer cannot track status in real-time or view assigned technician. |
| **Total Services Booked** | **Not Implemented** | None | No aggregation query or counter exists for customer service history. |
| **Customer Account Blocking** | **Not Implemented** | None | No customer accounts exist; therefore no account blocking mechanism exists. |

---

## 7. Admin Dashboard Analysis

Evaluation of Admin Dashboard requirements against the existing codebase:

| Admin Feature | Classification | Existing Implementation | Required Development |
|---|---|---|---|
| **Total Service Requests** | **Partially Implemented** | Backend endpoint `GET /api/enquiries` returns `mockDb.enquiries.length`, but has no frontend dashboard UI and no authentication. | Build Admin Web Dashboard overview page with KPI cards connected to persistent MySQL database. |
| **Pending Requests** | **Not Implemented** | Schema has column `status = 'pending'`, but no endpoint filters by pending and no UI displays it. | Implement filtered query `GET /api/admin/bookings?status=pending` and Admin status board. |
| **Assigned Requests** | **Not Implemented** | Neither database nor backend contains any assignment fields or logic. | Add `technician_id` foreign key, assignment endpoints, and Admin assigned filter. |
| **Completed Services** | **Not Implemented** | Schema has status enum, but no controller transitions or filters completed services. | Implement `status = 'completed'` lifecycle and completed service metrics in Admin. |
| **Service & Maintenance Management** | **Not Implemented** | No UI exists to review, filter, update, or cancel service and maintenance requests. | Build full Admin Booking Management table with filters, search, detail modals, and status dropdowns. |
| **Customer List & Account Details** | **Not Implemented** | No customer accounts or customer management endpoints exist. | Build Admin Customer Management module with customer directory, contact info, and booking history. |
| **Customer Block / Unblock** | **Not Implemented** | No `is_blocked` flag exists in database. | Add `is_blocked` column to `customers` table and toggle endpoint `PUT /api/admin/customers/:id/block`. |
| **Technician List & Details** | **Not Implemented** | No technician data models or UI exist. | Build Admin Technician Management module displaying all registered technicians and active status. |
| **Technician Account Creation** | **Not Implemented** | No endpoint or UI exists to create technician credentials. | Implement `POST /api/admin/technicians` (Admin creates technician username/phone and temporary password). |
| **Technician Credential Management** | **Not Implemented** | No password reset or credential management exists. | Implement password update/reset endpoints in Admin. |
| **Technician Block / Unblock** | **Not Implemented** | No technician status flag exists. | Add `is_active` / `is_blocked` flag and toggle endpoint `PUT /api/admin/technicians/:id/block`. |
| **Job Assignment & Reassignment** | **Not Implemented** | No assignment controller or UI exists. | Implement `POST /api/admin/bookings/:id/assign` and reassignment logic with push notification dispatch. |
| **Technician Attendance Monitoring** | **Not Implemented** | No attendance tables or tracking exist. | Implement attendance table and Admin attendance monitor showing On Duty / Off Duty status and punch times. |
| **Before & After Service Photos** | **Not Implemented** | No image upload or photo gallery exists. | Build photo display modal inside Admin booking detail drawer showing before/after images with timestamps. |
| **Service Completion Records** | **Not Implemented** | No completion audit table exists. | Implement completion record with service charges, technician notes, completion timestamp, and approval log. |
| **Completed Services per Technician** | **Not Implemented** | No aggregation query exists. | Add query `COUNT(bookings.id) WHERE technician_id = ? AND status = 'completed'` displayed in Admin. |

---

## 8. Technician System Analysis

Analysis of the technician subsystem against all required capabilities:

1. **Technician Database Tables**:
   - **Status**: **Not Implemented**.
   - **Finding**: Neither [schema.sql](file:///c:/Users/kille/Downloads/hello-engineer/database/schema.sql) nor [db.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/config/db.js) contains a `technicians` or `technician_attendance` table.
2. **Technician Account Creation (Admin-Only)**:
   - **Status**: **Not Implemented**.
   - **Finding**: Technicians must not self-register; accounts must be created by the Admin. No backend route (`POST /api/admin/technicians`) or frontend modal currently exists.
3. **Technician Login Credentials & Authentication**:
   - **Status**: **Not Implemented**.
   - **Finding**: No authentication route (`POST /api/auth/technician/login`) or JWT role validation exists.
4. **Technician Profile Information**:
   - **Status**: **Not Implemented**.
   - **Finding**: Required fields (Full Name, Mobile, Skills/Specializations, Experience, Profile Photo, Active Status) are completely absent.
5. **Assigned Jobs View**:
   - **Status**: **Not Implemented**.
   - **Finding**: No query exists to filter jobs by `WHERE technician_id = :id AND status IN ('assigned', 'in_progress')`.
6. **Customer Contact and Address Visibility**:
   - **Status**: **Not Implemented**.
   - **Finding**: In the proposed workflow, once assigned, the technician must see the customer's full name, phone number, location address, and reported issue. Currently, this data is only stored in raw lead emails.
7. **Job Status Updates (Inspection → Start → Finish)**:
   - **Status**: **Not Implemented**.
   - **Finding**: No state machine or endpoint exists for technicians to update job status (`assigned` → `inspected` → `in_progress` → `completed`).
8. **Technician Attendance (Strict Daily Rules)**:
   - **Status**: **Not Implemented**.
   - **Requirement Verification**:
     - Status: `On Duty` / `Off Duty`.
     - Rule 1: Prevent repeated On Duty / Off Duty toggling within the same day.
     - Rule 2: Once `Off Duty` is recorded, prevent another `On Duty` check-in on that same calendar day.
     - Finding: This requires a dedicated `technician_attendance` table with unique constraint on `(technician_id, attendance_date)` and validation in the backend attendance controller.
9. **Before & After Service Image Uploads**:
   - **Status**: **Not Implemented**.
   - **Finding**: No file upload endpoint (`POST /api/bookings/:id/photos`) exists.
10. **Service Completion & Statistics**:
    - **Status**: **Not Implemented**.
    - **Finding**: Technicians have no view to inspect their total completed services.

---

## 9. Existing Service Booking Workflow

### 9.1 Step-by-Step Trace of the Current Booking Process

```
[1. User Browses Services]
   ├── HomePage.jsx (Category cards / Most booked pills)
   ├── AllServicesPage.jsx (Live text search / Category filter)
   └── Chatbot Launcher (NLP conversational assistant)
            │
            ▼
[2. Booking Trigger]
   ├── User clicks "Book Now" on a card -> Opens BookingModal.jsx
   └── User types in Chatbot -> Initiates booking state in useChatbot.js
            │
            ▼
[3. Details Collection]
   ├── Full Name (Text input)
   ├── Mobile Number (10-digit text input)
   ├── District/City (From LocationContext, e.g., 'Chennai')
   ├── Address (Text input, e.g., '123 Anna Nagar')
   └── Selected Subservices (Checkboxes synced with servicesData.js)
            │
            ▼
[4. Frontend Submission]
   ├── submitEnquiry(payload) in api.js
   └── HTTP POST /api/enquiries (JSON body)
            │
            ▼
[5. Backend Validation & Storage]
   ├── validation.js validates name (>= 2 chars) and 10-digit Indian phone
   ├── enquiryController.js pushes object to mockDb.enquiries array (RAM)
   └── (Critical: No database INSERT occurs; no customer profile is linked)
            │
            ▼
[6. Notification Dispatch]
   ├── emailService.js builds HTML email with booking details
   ├── Nodemailer sends email via SMTP to configured admin address
   └── (Critical: No push notification, SMS, or WhatsApp is dispatched)
            │
            ▼
[7. Customer Feedback]
   ├── Backend returns HTTP 201 Created with mock enquiry_id
   └── BookingModal closes; Toast notification displays confirmation message
```

### 9.2 Key Workflow Gaps Identified

- **Step 1 (Service Selection)**: Works well on the web frontend. The service catalog is rich and categorized.
- **Step 2 (Booking Details)**: No problem description input in `BookingModal.jsx` (only exists in Chatbot and Contact forms).
- **Step 3 (Customer Information)**: Completely anonymous. Customer must re-type name and phone every time.
- **Step 4 (Service Images)**: Completely absent. No option to attach photos of broken equipment.
- **Step 5 (Visit Scheduling)**: `BookingModal.jsx` has no date or time selection fields.
- **Step 6 (Reaching Admin)**: The Admin only discovers the booking by checking their email inbox. The `GET /api/enquiries` endpoint is unauthenticated and not connected to any frontend dashboard.
- **Step 7 (Status Maintenance)**: Status remains `'pending'` indefinitely in RAM.
- **Step 8 (Technician Assignment)**: Does not exist.
- **Step 9 (Service Completion)**: Does not exist.

---

## 10. Maintenance and AMC Analysis

### 10.1 Actual Existing Behavior vs Regular Services

The codebase explicitly distinguishes between **Regular Services**, **Facility/Space Maintenance**, and **Annual Maintenance Contracts (AMC)**:

```
                            ┌────────────────────────┐
                            │    HELLO ENGINEER      │
                            │    SERVICE DOMAINS     │
                            └───────────┬────────────┘
                                        │
             ┌──────────────────────────┼──────────────────────────┐
             ▼                          ▼                          ▼
    [Regular Services]        [Maintenance Care]             [AMC Contracts]
    • On-demand repair        • Routine space upkeep         • Annual contracts
    • Fixed starting price    • Home / Workplace /           • Equipment AMC
      (e.g., AC ₹499)           Facility / Industrial        • System AMC
    • Single visit            • Periodic check-ups           • Facility AMC
    • POST /api/enquiries     • POST /api/enquiries/maint    • POST /api/enquiries/maint
```

1. **Frontend Presentation**:
   - Regular services live in [AllServicesPage.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/pages/AllServicesPage.jsx) and the Home page with individual rate cards.
   - Maintenance and AMC have a dedicated page in [AmcPage.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/pages/AmcPage.jsx). It features:
     - **6 Popular Maintenance Services**: AC & HVAC, Electrical, Appliance, Plumbing, IT & Network, Mechanical.
     - **4 Space Care Domains**: Home Care, Workplace Care, Facility Care, Industrial Care.
     - **4 Maintenance Modes**: Regular Care, Scheduled Care, Technical Care, Annual Care.
     - **3 AMC Coverage Tiers**: Equipment AMC (individual asset), System AMC (technical system), Facility AMC (multi-system property).
     - **5-Step Process Stepper**: Inspect → Assess → Plan → Maintain → Support.
2. **Backend Routing Distinction**:
   - Regular service requests are routed to `POST /api/enquiries` via [enquiryRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/enquiryRoutes.js).
   - Maintenance and AMC requests are routed to `POST /api/enquiries/maintenance` via [maintenanceController.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/controllers/maintenanceController.js).
   - The controller checks regex `/amc/i` on incoming `maintenance_type` and `service` fields, dynamically classifying requests as either `'AMC'` or `'Maintenance'`.
   - Stored in a separate array `mockDb.maintenance_enquiries` and mapped to a distinct SQL table `maintenance_enquiries`.
3. **Email Formatting Distinction**:
   - [emailService.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/utils/emailService.js) renders different HTML banners and metadata fields for AMC vs Maintenance vs Standard Service requests.
4. **Current Functional Limitation**:
   - Despite rich marketing presentations, AMC requests are currently handled strictly as lead enquiries. There is no contract lifecycle management, agreement duration tracking, periodic visit scheduling (e.g., quarterly visits), or contract renewal tracking in the backend or database.

---

## 11. Existing API Inventory

### 11.1 Complete Active API Endpoint Matrix

| Method | Endpoint | Source File | Purpose | Auth Required | Existing Frontend Caller | Reusable in Mobile App? |
|---|---|---|---|---|---|---|
| `GET` | `/` | [server.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/server.js) | Server online status & health link | None | None (browser / curl) | Diagnostic only |
| `GET` | `/api` | [server.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/server.js) | API version and available endpoint list | None | None | Diagnostic only |
| `GET` | `/api/health` | [server.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/server.js) | Liveness probe returning status `healthy` | None | None | Reusable (Health checks) |
| `GET` | `/api/categories` | [serviceRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/serviceRoutes.js) | Returns all 14 service categories from `mockDb` | None | None (Frontend uses `servicesData.js`) | **Yes** (Once backed by MySQL) |
| `GET` | `/api/services` | [serviceRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/serviceRoutes.js) | Returns services filtered by `?popular=`, `?most_booked=`, `?search=` | None | None (Frontend uses `servicesData.js`) | **Yes** (Once backed by MySQL) |
| `GET` | `/api/projects` | [projectRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/projectRoutes.js) | Returns 5 featured engineering projects | None | None (Frontend uses `projectsData.js`) | Optional (Showcase) |
| `POST` | `/api/enquiries` | [enquiryRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/enquiryRoutes.js) | Validates and receives service/callback booking lead; sends email | None | [BookingModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/BookingModal.jsx), [ContactModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/ContactModal.jsx), [useChatbot.js](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/chatbot/useChatbot.js) | **No** (Needs replacement with authenticated `/api/bookings`) |
| `GET` | `/api/enquiries` | [enquiryRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/enquiryRoutes.js) | Returns all stored enquiries from RAM | None | None | **No** (Unprotected; must be secured for Admin) |
| `POST` | `/api/enquiries/maintenance` | [enquiryRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/enquiryRoutes.js) | Validates and receives AMC/maintenance lead; sends email | None | None directly (Modals submit to `/api/enquiries` with type AMC) | **No** (Needs integration into `/api/bookings`) |
| `POST` | `/api/contact` | [contactRoutes.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/routes/contactRoutes.js) | Receives general contact form enquiry; sends email | None | [ContactPage.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/pages/ContactPage.jsx) | Reusable (General Contact) |

### 11.2 Inventory of Missing APIs Required for the Platform

To support the Customer Mobile App, Technician Mobile App, and Admin Web Dashboard, the following endpoints must be developed:

#### Authentication & Profile APIs
- `POST /api/auth/customer/register` (Customer registration with phone OTP or password)
- `POST /api/auth/customer/login` (Customer login returning JWT)
- `GET /api/auth/customer/profile` (Get customer profile, saved addresses, booking count)
- `PUT /api/auth/customer/profile` (Update customer profile & upload avatar)
- `POST /api/auth/technician/login` (Technician login using Admin-created credentials)
- `GET /api/auth/technician/profile` (Technician profile, duty status, completed count)
- `POST /api/auth/admin/login` (Admin login returning Admin JWT)
- `POST /api/auth/fcm-token` (Register/refresh FCM push notification token)

#### Customer Booking APIs
- `POST /api/bookings` (Create service or maintenance booking with problem description, image uploads, scheduled date/time, alternative contact)
- `GET /api/bookings/my-bookings` (Get authenticated customer's booking history and active tracking)
- `GET /api/bookings/:id` (Get booking status, assigned technician name & phone)
- `POST /api/bookings/:id/cancel` (Customer cancels pending booking)

#### Technician Operational APIs
- `GET /api/technician/jobs` (Get jobs assigned to authenticated technician)
- `GET /api/technician/jobs/:id` (Get job detail with customer name, address, contact, issue)
- `POST /api/technician/jobs/:id/before-photo` (Upload before-service photo)
- `POST /api/technician/jobs/:id/start` (Mark service as started)
- `POST /api/technician/jobs/:id/after-photo` (Upload after-service photo)
- `POST /api/technician/jobs/:id/finish` (Mark service as finished with final notes)
- `GET /api/technician/attendance/status` (Check today's On Duty / Off Duty status)
- `POST /api/technician/attendance/punch` (Punch On Duty or Off Duty with daily restriction enforcement)

#### Admin Management APIs
- `GET /api/admin/dashboard/stats` (Overview KPI statistics: total, pending, assigned, completed)
- `GET /api/admin/bookings` (List all bookings with status, date, and technician filters)
- `PUT /api/admin/bookings/:id/assign` (Assign or reassign technician to a booking)
- `PUT /api/admin/bookings/:id/status` (Update booking status or service charges)
- `GET /api/admin/customers` (List customers, details, and booking counts)
- `PUT /api/admin/customers/:id/block` (Toggle customer account block status)
- `GET /api/admin/technicians` (List technicians, completed job counts, active status)
- `POST /api/admin/technicians` (Admin creates technician account with credentials)
- `PUT /api/admin/technicians/:id` (Update technician credentials, skills, or status)
- `PUT /api/admin/technicians/:id/block` (Toggle technician block status)
- `GET /api/admin/attendance` (View daily technician attendance records)

---

## 12. Feature Gap Analysis

| Feature Area | Required System Capability | Existing State in Codebase | Gap Assessment |
|---|---|---|---|
| **Database Persistence** | Production relational storage in MySQL for all entities | In-memory `mockDb` object in [db.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/config/db.js). MySQL schema exists but is completely disconnected. | **Critical Gap**: Connect Express to MySQL via `mysql2/promise` and run schema migrations. |
| **Authentication & RBAC** | Secure JWT authentication for Customer, Technician, and Admin roles | Zero authentication. Skeleton `users` table exists in SQL but is unused. | **Critical Gap**: Implement JWT generation, password hashing (`bcryptjs`), and role authorization middleware. |
| **Customer App** | React Native Expo app with 5-tab navigation, saved profiles, and tracking | No mobile application folder exists. | **Critical Gap**: Build React Native Expo app in `mobile/`. |
| **Technician App** | React Native Expo app with 2-tab navigation, job lifecycle, photos, and attendance | No technician interface exists anywhere. | **Critical Gap**: Build Technician app views in `mobile/`. |
| **Admin Dashboard** | Web dashboard for booking dispatch, customer/technician management, attendance | No Admin UI exists in the web frontend. Only public marketing pages. | **Critical Gap**: Implement Admin views and navigation in existing React web app. |
| **Job Assignment** | Admin assigns technician; customer & technician receive details | No technician assignment concept exists in code. | **Critical Gap**: Build assignment controller, database foreign key, and notification triggers. |
| **File & Image Upload** | Problem photos, before-service photos, after-service photos | No file input in frontend, no `multer` in backend, no upload storage. | **Critical Gap**: Install `multer`, configure media storage directory/cloud, and add photo endpoints. |
| **Attendance Engine** | Strict On Duty / Off Duty with once-per-day lock | No attendance records or tracking exist. | **Critical Gap**: Build attendance table and daily locking business logic. |
| **Push Notifications** | FCM push notifications for booking, assignment, start, and finish events | Only Nodemailer email notifications to admin exist. | **Critical Gap**: Configure Firebase Admin SDK and device token registry. |
| **Alternative Contact** | Secondary contact notified for service updates | Not captured in UI, backend, or database. | **Critical Gap**: Add alternative contact fields and fallback notification logic. |

---

## 13. Required Mobile App Requirements

The mobile app will be developed in a new `mobile/` directory using **React Native with Expo**.

### 13.1 Customer Mobile Application

#### Bottom Navigation Structure (5 Tabs):
1. **Home**:
   - Location header with current district selector.
   - Hero banner with quick booking shortcut.
   - Grid of primary engineering disciplines (Appliance, Electrical, Plumbing, HVAC, etc.).
   - Most booked services with transparent starting prices.
   - "Why Choose Us" trust indicators and instant emergency call button.
2. **Service**:
   - Comprehensive categorized service catalog with real-time search.
   - Requirement filters (*Repair, Installation, Maintenance, Inspection*).
   - Dynamic service booking screen with:
     - Issue description text area.
     - Problem photo attachment (camera / gallery upload).
     - Preferred visit date & time slot picker.
     - Toggle: "Use saved profile address" OR "Specify alternative contact & address".
3. **AMC**:
   - Space care plans (Home Care, Workplace Care, Facility Care).
   - Equipment vs System AMC coverage tiers.
   - Interactive custom AMC request form.
4. **My Booking**:
   - Filter tabs: **Active** vs **Completed** vs **Cancelled**.
   - Real-time status badge (`Pending`, `Technician Assigned`, `In Progress`, `Completed`).
   - Assigned technician card: Technician Name, Phone Call trigger, Star rating.
   - Detailed timeline tracking.
5. **Profile**:
   - Profile avatar with photo upload.
   - Full Name, Phone Number, Email.
   - Saved Addresses list (Home, Office, Other) with "Add New Address".
   - Total services booked counter.
   - Helpdesk helpline (`94 45 60 60 60`), WhatsApp support, and Logout.

---

### 13.2 Technician Mobile Application

#### Bottom Navigation Structure (2 Tabs):
1. **My Booking**:
   - Header with **Attendance Control Widget**:
     - Status indicator: `On Duty` / `Off Duty`.
     - Punch button with strict daily lock:
       - Technicians can punch `On Duty` at the start of the shift.
       - Technicians can punch `Off Duty` at the end of the shift.
       - Repeated toggling is blocked. Once `Off Duty` is punched for the day, subsequent `On Duty` check-ins on that date are disabled.
   - Active Assigned Jobs list (filtered strictly to jobs assigned to this technician).
   - Job Card Details:
     - Customer Full Name and Contact Phone (with one-touch dial).
     - Full Service Address (with map directions button).
     - Reported issue & customer-uploaded problem photo.
     - Scheduled visit date and time window.
   - In-Job Action Workflow:
     - **Step 1: Inspect Issue**: Technician inspects the problem on-site.
     - **Step 2: Admin Charge Discussion**: Technician coordinates service charges with Admin.
     - **Step 3: Customer Approval**: Customer confirms quote.
     - **Step 4: Before-Service Photo**: Camera trigger to capture and upload photo before work begins.
     - **Step 5: Start Service**: Button enabled once before-photo is uploaded; transitions status to `In Progress`.
     - **Step 6: After-Service Photo**: Camera trigger to capture and upload photo after work is completed.
     - **Step 7: Finish Service**: Button enabled once after-photo is uploaded; records completion notes and marks job `Completed`.
2. **Profile**:
   - Technician profile photo, Name, Phone, and Assigned Engineering Domain.
   - KPI Stat: **Total Completed Services**.
   - Today's Attendance Log summary.
   - Contact Admin button and Logout.

---

## 14. Required Admin Dashboard Changes

The Admin Dashboard will be built directly into the existing React web application (accessed via a protected `/admin` route):

### Required Admin Modules:
1. **Dashboard Overview**:
   - Metric cards: Total Service Requests, Pending Requests, Assigned Requests, Completed Services.
   - Recent bookings feed and active technician attendance counter.
2. **Service & Maintenance Requests Management**:
   - Searchable, filterable data table for all incoming service and AMC requests.
   - Detailed request drawer: Customer info, address, problem description, attached problem photo.
   - Status transition controls (`Pending` → `Assigned` → `Completed` → `Cancelled`).
   - Service charge input field (for recording agreed charges during technician inspection).
3. **Job Assignment & Reassignment**:
   - Dropdown selector displaying available `On Duty` technicians matching the service category.
   - "Assign Technician" button that updates the database and triggers FCM push notifications to both Customer and Technician.
4. **Customer Management**:
   - Customer directory: Name, Phone, Email, Registered Date, Total Bookings Count.
   - Account Details modal showing complete service history.
   - **Customer Block / Unblock** action button (blocks customer from submitting future bookings).
5. **Technician Account & Credential Management**:
   - Technician directory: Photo, Name, Phone, Specialization, Active/Blocked status.
   - **Create Technician Account**: Admin inputs technician Name, Mobile Number, Skills, and assigns initial password.
   - Credential Reset: Admin can update technician credentials.
   - **Technician Block / Unblock**: Admin can toggle technician status (immediately revokes mobile app access).
6. **Technician Attendance Monitor**:
   - Daily attendance log: Shows check-in (`On Duty`) and check-out (`Off Duty`) timestamps for each technician.
   - Filter by date and technician.
7. **Service Records & History**:
   - Completed service audit log.
   - Before and after service photographs gallery with timestamps.
   - Total completed services per technician scorecard.

---

## 15. Required Backend and Database Changes

### 15.1 Database Schema Extensions

To support the complete platform, the MySQL database schema must be upgraded with the following tables:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   PROPOSED EXTENDED DATABASE SCHEMA                    │
├────────────────────────────────────────────────────────────────────────┤
│  EXISTING TABLES (Preserved):                                          │
│  • service_categories (id, slug, name, icon, display_order)            │
│  • services (id, category_id, name, slug, starting_price, is_active)   │
│  • projects (id, title, category, location, image, is_featured)        │
│                                                                        │
│  NEW / UPGRADED OPERATIONAL TABLES:                                    │
│  1. customers                                                          │
│     (id, name, phone, email, password_hash, avatar_url, is_blocked)    │
│                                                                        │
│  2. customer_addresses                                                 │
│     (id, customer_id, address_line, landmark, city, is_default)        │
│                                                                        │
│  3. technicians                                                        │
│     (id, name, phone, email, password_hash, specialization,            │
│      avatar_url, is_active, is_blocked, created_by_admin_id)           │
│                                                                        │
│  4. technician_attendance                                              │
│     (id, technician_id, attendance_date, on_duty_time, off_duty_time,  │
│      status ['on_duty', 'off_duty'], UNIQUE(technician_id, date))      │
│                                                                        │
│  5. bookings                                                           │
│     (id, booking_ref, customer_id, service_id, booking_type,           │
│      problem_description, preferred_date, preferred_time,             │
│      service_address, alt_contact_name, alt_contact_phone,             │
│      technician_id, agreed_charge, status, created_at, updated_at)     │
│      [Status ENUM: 'pending', 'assigned', 'in_progress',               │
│                    'completed', 'cancelled']                           │
│                                                                        │
│  6. service_photos                                                     │
│     (id, booking_id, photo_type ['problem', 'before', 'after'],        │
│      photo_url, uploaded_by_role, created_at)                          │
│                                                                        │
│  7. fcm_tokens                                                         │
│     (id, user_type ['customer', 'technician', 'admin'], user_id,       │
│      token, device_type, updated_at)                                   │
│                                                                        │
│  8. notifications                                                      │
│     (id, recipient_type, recipient_id, title, body, booking_id, read)  │
└────────────────────────────────────────────────────────────────────────┘
```

### 15.2 Backend Architectural Upgrades

1. **Activate Real MySQL Connection**:
   - Install `mysql2` in [backend/package.json](file:///c:/Users/kille/Downloads/hello-engineer/backend/package.json).
   - Refactor [backend/config/db.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/config/db.js) to initialize a connection pool (`mysql.createPool`) using environment variables (`DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_PORT`).
2. **Implement File Upload Service (Multer)**:
   - Install `multer`.
   - Create `backend/middleware/upload.js` configuring disk storage under `backend/uploads/` with validation for MIME types (`image/jpeg`, `image/png`, `image/webp`) and file size limits (max 5 MB).
   - Serve uploads statically via `app.use('/uploads', express.static(path.join(__dirname, 'uploads')))`.
3. **Implement Authentication & RBAC Middleware**:
   - Install `jsonwebtoken` and `bcryptjs`.
   - Create `backend/middleware/auth.js` with helpers:
     - `verifyToken`: Validates Authorization Bearer JWT.
     - `requireRole(['customer', 'technician', 'admin'])`: Enforces RBAC permissions.
4. **Implement Notification Dispatcher**:
   - Install `firebase-admin`.
   - Create `backend/utils/notificationService.js` to handle FCM push dispatches and fallback alerts.

---

## 16. Notification System Requirements

### 16.1 Notification Architecture & Channel Distinction

A critical architectural distinction must be maintained: **A phone number alone cannot receive Firebase Cloud Messaging (FCM) push notifications**.

```
┌────────────────────────────────────────────────────────────────────────┐
│                     NOTIFICATION DELIVERY MATRIX                       │
├──────────────────────────┬───────────────────┬─────────────────────────┤
│ Target Recipient         │ Primary Channel   │ Fallback Channel        │
├──────────────────────────┼───────────────────┼─────────────────────────┤
│ Registered Customer      │ FCM Push (Mobile) │ SMS / WhatsApp Alert    │
│ Assigned Technician      │ FCM Push (Mobile) │ SMS Notification        │
│ Admin Dispatch Team      │ In-App Web Alert  │ Email (Nodemailer)      │
│ Alternative Contact Person│ SMS / WhatsApp    │ Voice Call (Operations) │
└──────────────────────────┴───────────────────┴─────────────────────────┘
```

### 16.2 Required Event Triggers and Payloads

1. **Event: Customer Booking Submitted**:
   - **Target**: Admin Dashboard & Registered Customer.
   - **Customer Message**: *"Your Hello Engineer service request #HE-XXXX has been received. Our dispatch team is assigning an engineer."*
   - **Admin Message**: *"New service request #HE-XXXX received for [Service Name] in [City]."*
2. **Event: Technician Assigned**:
   - **Target**: Customer & Assigned Technician.
   - **Customer Push Notification**:
     - *Title*: "Technician Assigned!"
     - *Body*: "Engineer [Technician Name] ([Technician Phone]) has been assigned to your service request #HE-XXXX."
   - **Technician Push Notification**:
     - *Title*: "New Job Assigned!"
     - *Body*: "New job #HE-XXXX: [Service Name] at [Customer Address]. Customer: [Customer Name] ([Customer Phone])."
   - **Alternative Contact Handling**: If alternative contact name & phone are provided, an automated SMS / WhatsApp message is dispatched to that number because alternative contacts do not have a registered FCM device token.
3. **Event: Service Started**:
   - **Target**: Customer.
   - **Customer Push Notification**: *"Engineer [Technician Name] has started working on your service request #HE-XXXX."*
4. **Event: Service Completed**:
   - **Target**: Customer & Admin.
   - **Customer Push Notification**: *"Service request #HE-XXXX has been completed successfully! Thank you for choosing Hello Engineer."*
   - **Admin Alert**: *"Job #HE-XXXX marked complete by [Technician Name]. After-service photo verified."*

---

## 17. Recommended Mobile App Architecture

### 17.1 Technology Choice: React Native with Expo

- **Framework**: React Native 0.74+ with **Expo SDK 51+** (Managed Workflow).
- **Rationale**:
  - Expo provides out-of-the-box support for `expo-camera`, `expo-image-picker`, and `expo-notifications` (FCM integration).
  - Rapid cross-platform development (Android & iOS) from a single unified codebase.
  - Smooth over-the-air (OTA) updates and simplified build pipelines.

### 17.2 Proposed Directory Structure for `mobile/`

```
mobile/
├── App.js                         # Root Expo app entry (NavigationContainer, Providers)
├── app.json                       # Expo configuration (bundle ID, permissions, FCM credentials)
├── package.json                   # Dependencies (react-navigation, expo-camera, etc.)
├── src/
│   ├── api/
│   │   ├── client.js              # Axios instance with JWT interceptor & base URL
│   │   ├── authApi.js             # Login, register, profile endpoints
│   │   ├── bookingApi.js          # Booking submission, tracking, history
│   │   ├── technicianApi.js       # Job fetch, status update, photos, attendance
│   │   └── notificationApi.js     # FCM token registration
│   ├── context/
│   │   ├── AuthContext.js         # User state, JWT storage (expo-secure-store), role check
│   │   └── NotificationContext.js # FCM listener & push notification handler
│   ├── navigation/
│   │   ├── AppNavigator.js        # Root switch navigator (Auth vs Customer vs Technician)
│   │   ├── AuthNavigator.js       # Login, Register, Forgot Password stack
│   │   ├── CustomerNavigator.js   # 5-tab Bottom Tab Navigator
│   │   └── TechnicianNavigator.js # 2-tab Bottom Tab Navigator
│   ├── screens/
│   │   ├── auth/
│   │   │   ├── CustomerLoginScreen.js
│   │   │   ├── CustomerRegisterScreen.js
│   │   │   └── TechnicianLoginScreen.js
│   │   ├── customer/
│   │   │   ├── CustomerHomeScreen.js
│   │   │   ├── CustomerServiceCatalogScreen.js
│   │   │   ├── CustomerBookingFormScreen.js
│   │   │   ├── CustomerAmcScreen.js
│   │   │   ├── CustomerMyBookingsScreen.js
│   │   │   ├── CustomerBookingDetailScreen.js
│   │   │   └── CustomerProfileScreen.js
│   │   └── technician/
│   │       ├── TechnicianMyBookingsScreen.js
│   │       ├── TechnicianJobDetailScreen.js
│   │       ├── TechnicianCameraScreen.js
│   │       └── TechnicianProfileScreen.js
│   ├── components/
│   │   ├── AttendanceWidget.js    # Strict On Duty / Off Duty punch widget
│   │   ├── JobCard.js             # Technician assigned job summary card
│   │   ├── BookingTracker.js      # Visual 4-step progress bar
│   │   ├── PhotoUploader.js       # Camera capture / image picker component
│   │   └── CustomButton.js        # Branded orange/blue action button
│   ├── theme/
│   │   ├── colors.js              # Brand colors (#0B192C primary, #FF6500 orange)
│   │   └── typography.js          # Outfit / Plus Jakarta Sans fonts
│   └── utils/
│       └── storage.js             # Secure token and session helpers
```

---

## 18. Phased Development Roadmap

```
┌────────────────────────────────────────────────────────────────────────┐
│                     PHASED IMPLEMENTATION TIMELINE                     │
├────────────────────────────────────────────────────────────────────────┤
│  PHASE 1: Core Foundation & Database Activation                        │
│  • Install mysql2, bcryptjs, jsonwebtoken, multer in backend           │
│  • Connect backend/config/db.js to MySQL database pool                 │
│  • Run schema migrations for customers, technicians, bookings, etc.    │
│  • Implement JWT Auth & Multer upload endpoints                        │
├────────────────────────────────────────────────────────────────────────┤
│  PHASE 2: Admin Web Dashboard (Existing React Web App)                 │
│  • Add /admin routes and protected Admin layout in React               │
│  • Build Booking Management, Technician Management, and Attendance UI  │
│  • Implement Technician Job Assignment & Customer Block/Unblock        │
├────────────────────────────────────────────────────────────────────────┤
│  PHASE 3: Customer Mobile App (React Native Expo)                      │
│  • Initialize mobile/ project with Expo                                │
│  • Build 5-tab Customer navigation (Home, Service, AMC, Bookings, User)│
│  • Connect customer registration, booking flow with photo upload       │
├────────────────────────────────────────────────────────────────────────┤
│  PHASE 4: Technician Mobile App (React Native Expo)                    │
│  • Build 2-tab Technician navigation (My Booking, Profile)             │
│  • Implement strict On Duty / Off Duty attendance with daily lock      │
│  • Implement before/after camera capture and Start/Finish lifecycle    │
├────────────────────────────────────────────────────────────────────────┤
│  PHASE 5: Notifications & Real-Time Lifecycle                          │
│  • Configure Firebase Cloud Messaging (FCM) via firebase-admin         │
│  • Wire event triggers (Booked, Assigned, Started, Completed)          │
│  • Implement SMS fallback for alternative contact numbers              │
├────────────────────────────────────────────────────────────────────────┤
│  PHASE 6: End-to-End Testing, Hardening & Production Release           │
│  • Validate complete workflow end-to-end                               │
│  • Production build & deployment                                       │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 19. Risks, Dependencies, and Open Questions

### 19.1 Identified Technical Risks

1. **In-Memory Data Volatility**:
   - *Risk*: Any booking created right now in the backend is lost on server restart because it only exists in `mockDb`.
   - *Mitigation*: Migrating to MySQL is the absolute highest priority before launching any mobile app.
2. **Alternative Contact Push Delivery**:
   - *Risk*: If a customer supplies an alternative contact phone number, that contact cannot receive Firebase push notifications without an installed app and registered FCM token.
   - *Mitigation*: Integrate an SMS / WhatsApp API gateway (e.g., MSG91, Twilio, or WhatsApp Business Cloud API) for secondary contact updates.
3. **Attendance Edge Cases**:
   - *Risk*: Server time zone discrepancies (UTC vs IST) could cause a technician's attendance punch to register on the wrong calendar day, inadvertently locking them out.
   - *Mitigation*: Enforce Indian Standard Time (IST, UTC+05:30) calculations explicitly in the backend attendance controller.
4. **App Store Distribution**:
   - *Risk*: Publishing two distinct mobile apps (Customer App and Technician App) to Google Play and Apple App Store requires two separate developer listings and review processes.
   - *Alternative Option*: Build a single unified mobile app with a clean role-selector switch at login, or proceed with two separate apps if brand separation is strictly required.

---

## 20. Comprehensive Feature Comparison Table & Final Recommendations

### 20.1 Master Feature Comparison Table

| Feature | Current Status | Existing Files / APIs | Required Action | Priority |
|---|---|---|---|---|
| **Service Catalog Display** | Fully Implemented | [servicesData.js](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/data/servicesData.js), [AllServicesPage.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/pages/AllServicesPage.jsx), `GET /api/services` | Connect backend API to MySQL table `services`; reuse in Mobile App | P1 |
| **District / Location Selection** | Fully Implemented | [LocationContext.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/context/LocationContext.jsx), [LocationModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/LocationModal.jsx) | Port location context and district picker to React Native Expo | P2 |
| **Anonymous Web Enquiry** | Fully Implemented | [BookingModal.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/common/BookingModal.jsx), [enquiryController.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/controllers/enquiryController.js), `POST /api/enquiries` | Preserve for public website visitors as a lead capture fallback | P3 |
| **Email Notifications** | Fully Implemented | [emailService.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/utils/emailService.js), Nodemailer SMTP | Keep as backup notification channel for admin team | P3 |
| **Conversational Chatbot** | Fully Implemented | [Chatbot.jsx](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/chatbot/Chatbot.jsx), [useChatbot.js](file:///c:/Users/kille/Downloads/hello-engineer/frontend/src/components/chatbot/useChatbot.js) | Maintain on web; optional future integration into mobile app | P4 |
| **MySQL Database Connection** | Partially Implemented | [schema.sql](file:///c:/Users/kille/Downloads/hello-engineer/database/schema.sql) exists; [db.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/config/db.js) uses `mockDb` | Install `mysql2`, create pool, connect backend to MySQL | **P0 (Critical)** |
| **Customer Registration & Login** | Not Implemented | Skeleton `users` table in SQL | Create `customers` table, JWT auth controller, and login/register screens | **P0 (Critical)** |
| **Customer Saved Addresses** | Not Implemented | None | Create `customer_addresses` table and address management endpoints/screens | P1 |
| **Problem Image Upload** | Not Implemented | None | Install `multer`, create `/uploads` directory, add camera/picker in app | P1 |
| **Visit Scheduling (Date/Time)** | Partially Implemented | Backend fields in [enquiryController.js](file:///c:/Users/kille/Downloads/hello-engineer/backend/controllers/enquiryController.js); no UI picker | Add date/time pickers in mobile app and booking modal | P1 |
| **Alternative Contact Details** | Not Implemented | None | Add `alt_contact_name` and `alt_contact_phone` to schema and forms | P1 |
| **Customer Booking History & Tracking** | Not Implemented | None | Build `GET /api/bookings/my-bookings` and tracking screen in mobile app | P1 |
| **Admin Dashboard UI** | Not Implemented | None | Build `/admin` dashboard modules inside existing React web app | **P0 (Critical)** |
| **Admin Booking Dispatch & Assignment** | Not Implemented | None | Build assignment endpoint, technician selector, and status manager | **P0 (Critical)** |
| **Customer Block / Unblock** | Not Implemented | None | Add `is_blocked` flag to customers and Admin toggle controls | P1 |
| **Technician Account Creation (Admin)** | Not Implemented | None | Build Admin technician creation screen and `POST /api/admin/technicians` | **P0 (Critical)** |
| **Technician Mobile Login** | Not Implemented | None | Build technician login screen in Expo using Admin-generated credentials | **P0 (Critical)** |
| **Technician Assigned Jobs View** | Not Implemented | None | Build `GET /api/technician/jobs` and technician job list in mobile app | **P0 (Critical)** |
| **Technician Daily Attendance Lock** | Not Implemented | None | Create `technician_attendance` table and build strict once-per-day lock | P1 |
| **Before & After Service Photos** | Not Implemented | None | Create `service_photos` table, camera upload screen, and Admin viewer | P1 |
| **Technician Start / Finish Workflow** | Not Implemented | None | Implement `in_progress` and `completed` status transitions with validation | P1 |
| **Firebase Cloud Messaging (FCM)** | Not Implemented | None | Configure Firebase Admin SDK, token storage, and trigger push alerts | P1 |
| **SMS / WhatsApp Gateway** | Not Implemented | None | Integrate SMS gateway for secondary/alternative contact alerts | P2 |

### 20.2 Architectural Recommendations

1. **Maintain Unified Backend Integrity**:
   - Do **not** create a separate backend for mobile. Keep the existing Express server in [backend/](file:///c:/Users/kille/Downloads/hello-engineer/backend) as the single source of truth for both the React Web Application and the React Native Mobile Apps.
2. **Preserve Existing Public Website**:
   - The current marketing pages and booking modals are rich, responsive, and functional for prospective customers. Keep them intact for web visitors while adding the Admin Dashboard under a protected `/admin` route.
3. **Database-First Execution Order**:
   - Do not write mobile code until the MySQL database connection is live and verified with the extended schema. All client apps depend on a stable data model.
4. **Strict Attendance Security**:
   - Enforce the technician attendance rules at the backend database level (unique composite index on `technician_id` and `attendance_date`) rather than relying solely on mobile UI validation.

---
*Report generated strictly based on active codebase analysis. Awaiting user approval before beginning implementation.*
