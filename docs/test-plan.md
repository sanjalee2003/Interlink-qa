# InterLink Test Plan

| | |
|---|---|
| **Prepared by** | Sanjalee Dassanayake |
| **Date** | 2026-10-20 |
| **Version** | 1.1 |
| **Status** | Draft for review |

### Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| 1.0 | 2026-10-20 | Sanjalee Dassanayake | Initial version |
| 1.1 | 2026-10-21 | Sanjalee Dassanayake | Added full risk list with mitigations, environment details, suspension criteria, expanded defect lifecycle, traceability |

---

## 1. Objective

InterLink is a multi-tenant recruitment solution designed to provide a seamless, transparent, and trustworthy hiring experience. For InterLink, quality means:

- **Correct access control:** every role sees and does only what it is permitted to, and no company can see another company's data.
- **Reliable CV processing:** uploaded CVs are validated, parsed, and analysed by AI consistently.
- **Correct interview scheduling:** interview requests, panels, scheduling, and lifecycle changes behave as specified.

## 2. Scope

### 2.1 In Scope

- **Authentication:** registration, OTP verification, login (all four roles), logout, password reset, session refresh.
- **Authorization:** role-based access control for Candidate, Interviewer, Company Admin, and Super Admin, including tenant isolation.
- **Recruitment:** job posting, searching, filtering, applications, and subscription limits.
- **AI features:** resume upload, CV analysis, interview question generation, and chatbot message limits.
- **Interview management:** interview requests, panel formation, availability, scheduling, scoring, and lifecycle management.
- **Administration:** Super Admin controls.

### 2.2 Out of Scope

- Internal functionality of Supabase and OpenAI.
- Load and performance testing (covered in Project 5).
- Native mobile application testing.
- Cross-device compatibility testing.

### 2.3 Known Limitations (logged as observations)

The following limitations are documented in the project report. They are **recorded as observations, not counted as failed test cases**:

- Scanned (image-only) CVs are not read (no OCR).
- OTPs are held in server memory and are lost if the backend restarts.
- The platform supports English only.

## 3. Test Strategy

Testing combines manual and automated approaches, supported by structured test design techniques. Each test case is traced to a requirement or user story in the test case spreadsheet.

### 3.1 Test Types

| Type | Purpose |
|---|---|
| Functional | Verify core features such as login, job posting, CV upload, and interview scheduling. |
| Negative | Validate handling of invalid inputs, incorrect passwords, expired OTPs, and unsupported files. |
| Boundary | Verify system limits: OTP expiry, file size, subscription limits, chatbot usage. |
| Role-based access | Verify role permissions and data isolation between companies. |
| Regression | Ensure existing functionality remains stable after changes and bug fixes. |
| API (basic checks) | Validate API authorization, input validation, and tenant isolation. |
| Exploratory | Find unexpected issues through unscripted testing of user interactions. |

### 3.2 Approach

| Approach | Coverage |
|---|---|
| Manual | Registration, OTP verification, password reset, AI features, exploratory scenarios. |
| Automated | 5 to 7 critical workflows using Selenium and TestNG. |
| Database verification | Scheduled lifecycle jobs and related database updates. |

### 3.3 Test Design Techniques

- **Equivalence Partitioning:** group valid and invalid inputs into test categories.
- **Boundary Value Analysis:** test values at and around defined system limits.
- **Decision Table Testing:** validate access permissions across user roles.
- **State Transition Testing:** verify valid changes between interview statuses.

## 4. Environment

### 4.1 Configuration

| Item | Detail |
|---|---|
| Application under test | InterLink web application (React frontend, Spring Boot backend), commit `3f9a2c1` |
| Frontend | React, run on localhost |
| Backend | Spring Boot (Java 21), run locally |
| Database | Separate Supabase staging environment |
| Operating system | macOS 15.6.1 (Apple Silicon, arm64) |
| Browsers | Chrome 155.0.8059.85 (arm64) (automation), Firefox 152.0.4 (64-bit) (manual) |
| Email | Gmail SMTP with a dedicated test inbox |
| Tools | Jira, spreadsheet, Selenium, TestNG, Allure, Postman |

### 4.2 Test Accounts

| Account | Role | Company / Plan |
|---|---|---|
| candidate01 | Candidate | N/A |
| interviewer01, interviewer02 | Interviewer | N/A |
| admin.a | Company Admin | Company A (Free) |
| admin.b | Company Admin | Company B (Paid) |
| superadmin01 | Super Admin | N/A |

Only synthetic test data is used. Credentials are stored locally and excluded from Git using `.gitignore`.

### 4.3 Test Files

- Valid PDF and DOCX files (under 5 MB)
- Boundary files: exactly 5 MB and 5 MB + 1 byte
- Unsupported formats: `.png`, `.exe`, `.txt`
- Blank PDF and image-only (scanned) PDF

## 5. Entry Criteria

Testing begins when:

- The build deploys and the frontend and backend run successfully.
- The Supabase staging database is configured.
- Test accounts, files, and data are prepared.
- The test email inbox is accessible.
- Test cases are prepared and reviewed.

## 6. Exit Criteria

Testing is complete when:

- All high-priority test cases are executed, and all of them pass.
- No Critical or High severity defects remain open.
- At least 90% of planned test cases pass.
- Failed cases are retested and regression testing is completed.
- The test summary report and release recommendation are prepared.

## 7. Suspension and Resumption Criteria

- **Suspend** testing if the application cannot be started, login fails for all roles, or the staging database is unreachable.
- **Resume** once a fix or workaround is in place and a quick smoke check of login and navigation passes.

## 8. Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | AI output varies between runs, so CV scores and generated questions are not identical. | High | Medium | Assert on ranges, structure, and required fields rather than exact text. Repeat key AI tests. |
| R2 | Email depends on external Gmail SMTP, so OTP emails may be delayed or not delivered. | Medium | High | Use a dedicated test inbox. Retry once before logging a defect. Check SMTP status first. |
| R3 | OTPs are lost if the backend restarts. | Medium | Medium | Avoid restarting mid-test. Request a new OTP after any restart. Logged as an observation. |
| R4 | The lifecycle job runs only every 15 minutes, so status changes are not immediate. | High | Medium | Schedule lifecycle tests with wait time, or trigger the job manually where possible. |
| R5 | Hosted services (Supabase, SMTP) may be slow or unavailable after idle periods. | Medium | Medium | Warm up services before each session. Retry before logging a defect. |
| R6 | The role-and-tenant matrix (4 roles across multiple companies) is large and easy to under-cover. | Medium | High | Use a decision table. Prioritise cross-tenant checks. Automate key access checks. |

## 9. Defect Process

### 9.1 Defect Tracking

All defects are recorded in Jira (project IQA) as Bug issues, linked to the related user stories. Labels used: `auth`, `rbac`, `scheduling`, `api`, `ai`.

### 9.2 Bug Report Contents

Title, environment (commit, browser, OS), reproduction steps, expected result, actual result, severity, priority, and screenshots or recordings.

### 9.3 Severity

| Severity | Definition | Example |
|---|---|---|
| Critical | System failure, data loss, or security breach | Cross-company data access |
| High | Major functionality failure | Interview cannot be finalized |
| Medium | Partial functionality failure | Incorrect filtering results |
| Low | Minor or cosmetic issue | UI misalignment |

### 9.4 Priority

| Priority | Description |
|---|---|
| P1 | Immediate fix; blocks testing or release |
| P2 | Fix before release |
| P3 | Fix when time permits |

Severity measures impact; priority determines fixing urgency.

### 9.5 Defect Lifecycle

```
To Do -> In Progress -> Fixed -> Retest -> Done
```

- A failed retest moves the bug to **Reopened**, then back to **In Progress**.
- Bugs that are not valid or already reported are closed as **Rejected** or **Duplicate**.
- If custom Jira statuses are unavailable, the `retest` label is used.

## 10. Deliverables

- Test Plan (this document)
- Test case spreadsheet (60+ test cases)
- Jira defect reports (8 to 10)
- Selenium + TestNG automation project
- Allure test report
- Test summary report (`docs/test-summary.md`)
