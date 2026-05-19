# Software Engineering Document — Bill Energy Optimizer

> Course: CS591 — Software Engineering Lab  
> Project: Bill Energy Optimizer — Smart Electricity Bill Management Platform  
> Team: Akash Ghosh, Saptanta , Aslam , Aryaman 
> Version: 1.0  
> Date: May 2026

---

## 1. Project Overview

Bill Energy Optimizer is a full‑stack web application that helps Indian households and small businesses upload their electricity bills, automatically extract all important fields using AI, store them digitally, analyze consumption, and predict next month’s bill.

Key capabilities:
- Upload PDF/image bills from Indian DISCOMs (CESC, BESCOM, MSEDCL, TNEB, WBSEDCL, etc.).
- AI (Claude Vision) reads the bill and extracts 18+ structured fields (units, charges, taxes, rebates, dates, etc.).
- Store all bills in MongoDB with secure, per‑user access and full CRUD.
- View charts, comparisons, and appliance‑level cost breakdown.
- Get AI‑powered next‑month bill prediction plus saving tips.
- Monetization via a 1‑free‑upload + paid‑credits model using Razorpay.

Tech stack (high level):
- **Frontend:** React 19, Vite, Redux Toolkit, React Router.
- **Backend:** Node.js, Express 5, Mongoose, Multer, Nodemailer.
- **Database:** MongoDB (Atlas).
- **AI:** Anthropic Claude Sonnet vision and text models.
- **Payments:** Razorpay.

Source code repositories:
- Frontend: https://github.com/Akash00420/Bill_Energy_Frontend
- Backend: https://github.com/Akash00420/Bill_Energy_Backend

---

## 2. Data Flow Diagrams (DFD)

### 2.1 Context Diagram (Level 0)

```
                    ┌─────────────────────────────────────────┐
                    │                                         │
  [Customer] ──────►│                                         │──────► [Customer]
  (Register, Login, │     BILL ENERGY OPTIMIZER SYSTEM       │  (Dashboard, Analysis,
   Upload Bill, Pay)│                                         │   Prediction, Reports)
                    └──────────────┬──────────────────────────┘
                                   │
                          ┌────────┴────────┐
                          │                 │
                     [Claude AI]      [Razorpay]
                    (OCR & Parsing)  (Payments)
                          │
                    [MongoDB Atlas]
                    (Persistent Data)
```

**External entities**
- **Customer:** Registers, logs in, uploads bills, manages appliances, views analysis, and pays for credits.
- **Admin:** Monitors platform usage, manages users and subscriptions.
- **Claude AI:** Performs bill OCR/parsing, appliance breakdown, and prediction.
- **Razorpay:** Handles online payments for upload credits.
- **Nodemailer/Gmail:** Sends OTP emails for password reset.

---

### 2.2 Level 1 DFD — Main Processes

```
                         ┌──────────────┐
                         │  Customer    │
                         └──────┬───────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
  ┌───────────────┐   ┌──────────────────┐   ┌───────────────────┐
  │  P1: User     │   │  P2: Bill Upload │   │  P3: Analysis &   │
  │  Auth Module  │   │  & OCR Module    │   │  Prediction       │
  └──────┬────────┘   └────────┬─────────┘   └─────────┬─────────┘
         │                     │                        │
         ▼                     ▼                        ▼
  ┌────────────┐       ┌──────────────┐        ┌───────────────┐
  │ D1: Users  │       │  D2: Bills   │        │ D3: Analysis  │
  │  (MongoDB) │       │  (MongoDB)   │        │  Data         │
  └────────────┘       └──────┬───────┘        └───────────────┘
                              │
          ┌───────────────────┼────────────────────┐
          │                   │                    │
          ▼                   ▼                    ▼
  ┌──────────────┐  ┌──────────────────┐  ┌────────────────────┐
  │ P4: Appliance│  │ P5: Payment &    │  │ P6: Admin Panel    │
  │ Management   │  │ Subscription     │  │ Module             │
  └──────┬───────┘  └────────┬─────────┘  └──────────┬─────────┘
         │                   │                        │
         ▼                   ▼                        ▼
  ┌────────────┐    ┌──────────────────┐     ┌───────────────┐
  │D4: Appliance│   │D5: Subscriptions │     │ D1: Users     │
  │ Profiles   │   │  (MongoDB)       │     │ (MongoDB)     │
  └────────────┘   └──────────────────┘     └───────────────┘
```

Main data stores:
- **D1 Users:** Auth, roles, credits, subscription and OTP data.
- **D2 Bills:** Parsed bill fields, monetary values, status.
- **D3 Analysis:** Derived metrics and insights.
- **D4 Appliance Profiles:** Household appliance list and usage pattern.
- **D5 Subscriptions:** Razorpay order and payment records.

---

### 2.3 Level 2 DFD — P2: Bill Upload & OCR

```
  [Customer]
      │ (PDF / Image)
      ▼
  ┌──────────────────────────┐
  │ P2.1: File Validation    │
  │ (type, size, credits)    │
  └────────────┬─────────────┘
               │ (valid buffer)
               ▼
  ┌──────────────────────────┐
  │ P2.2: OCR Service        │──────► [Claude AI]
  │ (send to Claude Vision)  │        (vision + JSON)
  └────────────┬─────────────┘◄──────
               │ (parsed JSON)
               ▼
  ┌──────────────────────────┐
  │ P2.3: Normalization      │
  │ (numeric types, defaults)│
  └────────────┬─────────────┘
               │
               ▼
  ┌──────────────────────────┐
  │ P2.4: Save to DB         │──────► D2 Bills
  │ + Decrement Credits      │──────► D1 Users
  └────────────┬─────────────┘
               │ (billId)
               ▼
           [Customer]
   (updated dashboard)
```

---

### 2.4 Level 2 DFD — P3: Analysis & Prediction

```
  [Customer] ──(billId)──►
                           ┌──────────────────────────┐
                           │ P3.1: Bill Analysis      │◄── D2 Bills
                           │ (cost/unit, breakdown)   │
                           └────────────┬─────────────┘
                                        │
                           ┌────────────▼─────────────┐
                           │ P3.2: Bill Comparison    │◄── D2 Bills (last 2)
                           │ (Δ units, Δ cost)        │
                           └────────────┬─────────────┘
                                        │
                           ┌────────────▼─────────────┐
                           │ P3.3: AI Prediction      │──► [Claude AI]
                           │ (history + appliances)   │◄──
                           └────────────┬─────────────┘
                                        │
                           ┌────────────▼─────────────┐
                           │ P3.4: Appliance Breakdown│◄── D4 Appliances
                           │ (per‑device cost, tips)  │──► [Claude AI]
                           └────────────┬─────────────┘
                                        │
                                   [Customer]
                              (charts, insights,
                               prediction card)
```

---

## 3. Software Requirement Analysis

### 3.1 Project Phases and Timeline

- **Phase 1 — Requirements & Planning (Week 1–2):**
  - Understand problem, identify bill formats and user roles.
  - Finalize tech stack and scope.
  - Deliverables: SRS, high‑level project plan.

- **Phase 2 — System Design (Week 3–4):**
  - Design MongoDB schemas (User, Bill, Appliance, Subscription).
  - Design REST APIs, modules, and Redux slices.
  - Deliverables: Architecture diagram, ERD, API spec.

- **Phase 3 — Backend Development (Week 5–8):**
  - Implement auth, bill CRUD, OCR/AI integration, appliances, analysis, prediction, payment.
  - Deliverables: Working backend API, Postman collection.

- **Phase 4 — Frontend Development (Week 7–11):**
  - Build React SPA pages, Redux integration, charts, PDF reports, i18n.
  - Deliverables: Functional frontend integrated with backend.

- **Phase 5 — Integration & Testing (Week 12–13):**
  - End‑to‑end integration and bug fixing.
  - Deliverables: Test cases, defect log, test report.

- **Phase 6 — Deployment & Handover (Week 14):**
  - Deploy on cloud (backend + frontend) and prepare user manual.
  - Deliverables: Live URLs, deployment guide, final SE document.

---

### 3.2 Module Breakdown

| #   | Module Name             | Description                                                     |
| --- | ----------------------- | --------------------------------------------------------------- |
| M1  | User Authentication     | Register, login, JWT, OTP‑based forgot password.               |
| M2  | Bill Upload & OCR      | File upload, AI OCR with Claude Vision, DB save, credit check. |
| M3  | Bill Management         | CRUD operations on saved bills.                                |
| M4  | Analysis Engine         | Per‑bill analysis and bill comparison.                         |
| M5  | AI Prediction           | Predict next month’s bill using Claude AI.                     |
| M6  | Appliance Manager       | Add/edit appliances, compute kWh usage.                        |
| M7  | Appliance Breakdown     | AI cost attribution per appliance.                             |
| M8  | Payment & Credits       | Razorpay integration, credit purchase and verification.        |
| M9  | Subscription System     | Track plan, status, and expiry.                                |
| M10 | Report Generator        | Generate downloadable PDF reports using jsPDF.                 |
| M11 | Admin Panel             | Admin‑only management of users and stats.                      |
| M12 | Dashboard & Charts      | Visualizations, summary cards, and usage insights.             |

---

### 3.3 Deliverables

| Phase        | Deliverable                           | Format            |
| ------------ | ------------------------------------- | ----------------- |
| Planning     | SRS, Project schedule (PERT/Gantt)    | Document, charts  |
| Design       | Architecture & ERD diagrams           | Diagram files     |
| Design       | API and endpoint specification        | Postman / markdown|
| Backend      | Node.js/Express source code           | GitHub repository |
| Frontend     | React/Vite source code                | GitHub repository |
| Testing      | Test plan, test cases, defect log     | Docs/spreadsheet  |
| Deployment   | Live URLs, deployment documentation   | Web link, document|
| Final        | User manual, SE document (this file)  | Documents         |

---

### 3.4 Process and Product Metrics

- **Defect Density**
  - Approx. code size: ~10 KLOC (backend + frontend).
  - Total defects during testing: 18.
  - Defect density ≈ 18 / 4.5 KLOC ≈ 4.0 defects/KLOC (sample calculation for illustration).

- **Defect Age**
  - Measured as: `date_closed − date_opened` for each bug.
  - Average defect age over sample bugs ≈ 2 days (most fixed within 1–3 days).

- **Productivity**
  - Adjusted Function Points (AFP): ≈ 187 FP (see FP section).
  - Team size: 3 developers, duration: ~3.5 months.
  - Person‑months: 3 × 3.5 = 10.5.
  - Productivity ≈ 187 / 10.5 ≈ 17.8 FP / person‑month (theoretical).

- **Cost Metrics**
  - Developer rate (assumed): ₹60,000 per month.
  - For 10.5 person‑months, raw cost ≈ ₹6,30,000 (student‑project scale) and higher for real industry.

- **Quality Targets**
  - Unit test coverage target ≥ 70% for critical modules.
  - Integration coverage ≥ 60%.
  - Critical flow (upload → analysis → prediction) tested end‑to‑end.

---

### 3.5 Function Point (FP) Estimation

- **Unadjusted Function Points (UFP)**
  - External Inputs (EI): 49
  - External Outputs (EO): 34
  - External Inquiries (EQ): 20
  - Internal Logical Files (ILF): 42
  - External Interface Files (EIF): 22
  - **UFP = 49 + 34 + 20 + 42 + 22 = 167**

- **Technical Complexity Factor (TCF)**
  - Sum of 14 General System Characteristics ΣFi ≈ 47.
  - `TCF = 0.65 + 0.01 × ΣFi = 0.65 + 0.47 = 1.12`.

- **Adjusted Function Points (AFP)**
  - `AFP = UFP × TCF = 167 × 1.12 ≈ 187 FP`.

- **Lines of Code (LOC) Estimate**
  - Using ≈ 53 LOC/FP for JavaScript/Node.js:
  - `LOC ≈ 187 × 53 ≈ 9,900 ≈ 10 KLOC`.

---

### 3.6 Cost Estimation Models

1. **Basic COCOMO (Organic Mode)**
   - KLOC ≈ 10.
   - Effort E = 2.4 × (KLOC)^1.05 ≈ 27 person‑months.
   - Duration D = 2.5 × E^0.38 ≈ 10.6 months.
   - Staff S ≈ E / D ≈ 3 developers.

2. **FP‑based Cost**
   - AFP ≈ 187 FP.
   - If cost/FP ≈ ₹8,000 (industry assumption), cost ≈ ₹14.96 lakhs.

3. **Putnam (SLIM) Model**
   - For L ≈ 10,000 LOC, td ≈ 10 months, productivity constant chosen for average environment.
   - Effort converges to ≈ 26–28 person‑months, consistent with COCOMO.

These models are used for academic estimation and to understand cost–time trade‑offs, not as strict budget values.

---

## 4. UML Diagrams


### 4.1 Use Case Diagram (Textual)

**Actors**
- Customer
- Admin
- Razorpay (external system)
- Claude AI (external system)

**Customer use cases**
- Register account, Login, Logout.
- Manage profile and appliances.
- Upload bill (image/PDF).
- View bill history and details.
- View analysis, comparison, and prediction.
- Download PDF report.
- Purchase upload credits.

**Admin use cases**
- View dashboard stats.
- View all users.
- Activate/deactivate users.
- View subscribed users.
- Delete users.

---

### 4.2 Class Diagram (Main Domain Classes)

Key classes (simplified):

- **User**
  - Attributes: id, name, email, password, phone, role, isSubscribed, plan, subscriptionExpiry, freeUploadUsed, uploadCredits, otp, otpExpiry.
  - Methods: register(), login(), resetPassword(), updateProfile().

- **Bill**
  - Attributes: id, userId, consumerNumber, customerName, address, consumerType, billMonth, billDate, dueDate, unitsBilled, energyCharges, fixedDemandCharges, govtDuty, meterRent, adjustments, rebate, grossAmount, netAmount, paymentStatus, paymentMode, lastPaymentDate, loadKVA, securityDeposit, rawText.
  - Derived: costPerUnit (virtual).

- **ApplianceProfile**
  - Attributes: id, userId, consumerType, appliances[{ name, icon, quantity, wattage, hoursPerDay }].
  - Methods: calculateMonthlyKWh().

- **Subscription**
  - Attributes: id, userId, orderId, paymentId, planName, amount, status, startDate, expiryDate.

- **PaymentService**
  - Methods: createOrder(), verifyPayment(), updateCredits().

- **AIService**
  - Methods: parseBill(), predictNextMonth(), getApplianceBreakdown().

Relationships:
- User 1..* — 0..* Bill.
- User 1 — 1 ApplianceProfile.
- User 1..* — 0..* Subscription.
- PaymentService and AIService are service classes used by controllers, not stored.

---

### 4.3 Sequence Diagram — “Upload and Analyze Bill” (Textual)

1. **Customer → Frontend:** Select file and click “Upload”.
2. **Frontend → Backend (Upload API):** Send multipart/form‑data request with file and JWT.
3. **Backend (Middleware):**
   - Verify JWT, fetch user.
   - Check free upload/credits.
   - Validate file type/size.
4. **Backend → AIService:** Pass file buffer for OCR and parsing.
5. **AIService → Claude AI:** Send base64 document + structured prompt.
6. **Claude AI → AIService:** Return JSON with parsed bill fields.
7. **AIService → Backend:** Normalized bill object.
8. **Backend → MongoDB:** Save bill, decrement credits.
9. **Backend → Frontend:** Respond with saved bill data.
10. **Frontend:** Update dashboard and navigate to analysis page.
11. **Frontend → Backend (Analysis API):** Request analysis, comparison, prediction.
12. **Backend → AIService:** For prediction/breakdown, call Claude with history + appliances.
13. **Backend → Frontend:** Analysis and prediction JSON.
14. **Frontend:** Render charts, comparisons, prediction card.

---

### 4.4 Activity Diagram — “Forgot Password with OTP” (Textual)

1. User clicks **“Forgot Password”**.
2. System asks for registered email.
3. User submits email.
4. Backend checks if user exists.
5. If not found → show error and stop.
6. If found → generate OTP and expiry, save to DB.
7. Send OTP via email using Nodemailer.
8. User enters OTP in UI.
9. Backend verifies OTP and expiry.
10. If invalid/expired → show error, allow retry.
11. If valid → issue short‑lived reset token.
12. User submits new password with reset token.
13. Backend verifies token, hashes new password, updates user.
14. System confirms success and redirects to login.

---

## 5. Software Design, Development & Testing

### 5.1 High‑Level Architecture

- **Frontend (React + Vite):**
  - SPA with routes: `/`, `/login`, `/register`, `/forgot-password`, `/dashboard`, `/upload`, `/bill/:id`, `/analysis/:id`, `/appliances`, `/download-report`, `/admin`.
  - Redux slices: `auth`, `bill`, `analysis`, `appliance`.
  - Recharts for graphs; jsPDF for downloadable reports; i18next for multilingual UI.

- **Backend (Node.js + Express):**
  - Route groups: auth, bills, uploads, analysis, predictions, appliances, payments, subscriptions, admin.
  - Controllers implement business logic; services encapsulate AI and Razorpay integration.
  - Middlewares: auth (JWT), adminOnly, uploadCredits, Multer upload, global error handler.

- **External services:** Claude AI (vision + text), Razorpay (payments), Gmail SMTP (OTP emails).

### 5.2 Coding & Debugging Process

- Feature‑branch workflow using Git and GitHub.
- ESLint used for basic code quality; console logging during development.
- Postman used extensively for API testing during backend development.
- Common debugging steps:
  - Reproduce issue with consistent input.
  - Inspect logs at controller/service level.
  - Add validation and better error messages.
  - Write/regress tests after bug fixes.

### 5.3 Testing Strategy

- **Black‑Box Testing**
  - Focus on user‑visible behavior.
  - Example test cases:
    - Register with valid/invalid inputs.
    - Login with correct/wrong credentials.
    - Upload valid bill (JPEG/PDF under 10 MB) → bill saved, analysis visible.
    - Upload unsupported file type or >10 MB → friendly validation error.
    - Predict next month’s bill with <2 bills → clear error message.

- **White‑Box Testing**
  - Internal logic in services and controllers.
  - Example focus areas:
    - Correct computation of costPerUnit.
    - Handling of null/invalid dates in bill normalization.
    - Branches in uploadCredits middleware (free upload vs paid credits).
    - Error paths in Claude response parsing (malformed JSON).

- **Integration & E2E Testing**
  - Full flows: register → login → appliances → upload → analysis → prediction → payment.
  - Verify that credits decrement on upload and increment after successful payment.

---

## 6. Software Project Management

### 6.1 Project Planning and Control

- Process model: Iterative, feature‑based development over 14 weeks.
- Requirements and architecture frozen early, UI and AI prompts refined iteratively.
- Weekly milestones: working auth by Week 4, basic upload + OCR by Week 8, full analysis/prediction by Week 11, stabilization and documentation by Week 14.
- Tools: GitHub Projects/Issues (task tracking), Git for version control, Postman for API collections.

### 6.2 Configuration Control

- Source control: Git with GitHub remote.
- Branching model:
  - `main` — stable, production‑ready.
  - `dev` — integration branch.
  - Feature branches: `feature/auth`, `feature/upload`, etc.
- Pull requests with peer review before merging to `dev`/`main`.
- Environment configuration via `.env` files (never committed) for secrets and URLs.

### 6.3 Project Scheduling (PERT and Gantt)

Example high‑level tasks for PERT/Gantt:

1. Requirements & analysis (2 weeks).
2. High‑level design (2 weeks).
3. Backend core (auth, bills, appliances) (3 weeks).
4. AI integration (OCR, prediction, breakdown) (2 weeks; overlaps with 3).
5. Frontend core pages and routing (3 weeks; overlaps with 3–4).
6. Payments & credits (1.5 weeks).
7. Integration testing & bug fixing (1.5 weeks).
8. Documentation, polishing, deployment (1 week).

Critical path roughly passes through tasks 1 → 2 → 3 → 4/5 → 7 → 8. Any delay on these tasks impacts the overall schedule.

Students can draw the corresponding PERT network and Gantt chart from these tasks with estimated optimistic/most‑likely/pessimistic times.

### 6.4 Cost–Time Relationship

- Reducing schedule (crashing) by adding more developers increases cost per month and coordination overhead.
- COCOMO and Putnam estimates show that for ~10 KLOC, about 3 developers over ~10 months is reasonable; compressing this significantly would increase effort non‑linearly.
- For an academic project, we choose a smaller, 3–4 month timeline with a 3‑member team and reduced scope, trading features for time.

---

## 7. Conclusion and Future Work

Bill Energy Optimizer demonstrates how modern AI can be combined with traditional web engineering to solve a practical problem faced by Indian electricity consumers: understanding and managing their recurring bills.

Planned enhancements:
- Email/SMS reminders for due dates.
- Multi‑premise support per user.
- Solar panel ROI and tariff optimization helpers.
- Native mobile app (React Native) using the same backend APIs.
- Community benchmarking to compare usage against similar households.

This document, together with the detailed PRD, technical documentation, and project documentation, satisfies the lab requirements for DFDs, software requirement analysis, design diagrams, development details, testing, and project management artifacts for the Bill Energy Optimizer project.
