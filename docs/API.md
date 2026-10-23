# CivicPulse Nexus — REST API Documentation

This document provides a comprehensive specification of the REST API endpoints exposed across the **CivicPulse Nexus** platform. All client requests from the frontend enter via the unified **Spring Cloud API Gateway** running on port `8084`.

---

## Global Ingress & Authentication

- **Gateway Base URL**: `http://localhost:8084`
- **Authentication**: Bearer Token (Keycloak OAuth2 / OpenID Connect JWT)
- **Header**:
  ```http
  Authorization: Bearer <ACCESS_TOKEN>
  Content-Type: application/json
  ```
- **Error Response Standard (`ApiError`)**:
  ```json
  {
    "timestamp": "2026-10-23T10:00:00",
    "status": 404,
    "error": "Not Found",
    "message": "Resource not found with id : 1",
    "path": "/api/citizens/1"
  }
  ```

---

## 1. User Service (`/api/users/**`)

Downstream Service: `user-service` (Port: `8081`, Database: `user_db`)

### 1.1 Register User
- **Method**: `POST`
- **Path**: `/api/users/register`
- **Auth Required**: No (Public)
- **Request Body**:
  ```json
  {
    "email": "citizen.user@example.com",
    "password": "SecurePassword123!",
    "firstName": "John",
    "lastName": "Doe",
    "mobile": "9876543210",
    "status": "ACTIVE"
  }
  ```
- **Success Response** (`201 Created`):
  ```json
  {
    "id": 1,
    "email": "citizen.user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "CITIZEN",
    "status": "ACTIVE",
    "mobile": "9876543210",
    "keycloakId": "3f4a2b1c-9988-4433-2211-abcdefabcdef"
  }
  ```

### 1.2 User Login
- **Method**: `POST`
- **Path**: `/api/users/login`
- **Auth Required**: No (Public)
- **Request Body**:
  ```json
  {
    "email": "citizen.user@example.com",
    "password": "SecurePassword123!"
  }
  ```
- **Success Response** (`200 OK`):
  ```json
  {
    "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer"
  }
  ```

### 1.3 Get User by ID
- **Method**: `GET`
- **Path**: `/api/users/{id}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`):
  ```json
  {
    "id": 1,
    "email": "citizen.user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "CITIZEN",
    "status": "ACTIVE",
    "mobile": "9876543210",
    "keycloakId": "3f4a2b1c-9988-4433-2211-abcdefabcdef"
  }
  ```

### 1.4 Get User by Keycloak ID
- **Method**: `GET`
- **Path**: `/api/users/keycloak/{keycloakId}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Same schema as `1.3`.

### 1.5 List All Users
- **Method**: `GET`
- **Path**: `/api/users`
- **Auth Required**: Yes (`ADMIN`)
- **Success Response** (`200 OK`): Array of UserResponse objects.

### 1.6 Update User
- **Method**: `PUT`
- **Path**: `/api/users/{id}`
- **Auth Required**: Yes
- **Request Body**:
  ```json
  {
    "role": "CITIZEN",
    "status": "ACTIVE",
    "mobile": "9876543211"
  }
  ```
- **Success Response** (`200 OK`): Updated UserResponse object.

### 1.7 Delete User
- **Method**: `DELETE`
- **Path**: `/api/users/{id}`
- **Auth Required**: Yes (`ADMIN`)
- **Success Response** (`200 OK`): `"User deleted successfully"`

---

## 2. Citizen Service (`/api/citizens/**`)

Downstream Service: `citizen-service` (Port: `8082`, Database: `citizen_db`)

### 2.1 Register Citizen Demographic Profile
- **Method**: `POST`
- **Path**: `/api/citizens`
- **Auth Required**: Yes (`CITIZEN`)
- **Request Body**:
  ```json
  {
    "userId": 1,
    "aadhaarNumber": "123456789012",
    "firstName": "John",
    "lastName": "Doe",
    "gender": "MALE",
    "dateOfBirth": "1990-01-15",
    "mobile": "9876543210",
    "address": "42 Civic Lane, North Ward",
    "city": "Metropolis",
    "district": "Central",
    "state": "Maharashtra",
    "pincode": "400001",
    "occupation": "Software Engineer"
  }
  ```
- **Success Response** (`200 OK`):
  ```json
  {
    "id": 1,
    "userId": 1,
    "aadhaarNumber": "123456789012",
    "firstName": "John",
    "lastName": "Doe",
    "gender": "MALE",
    "dateOfBirth": "1990-01-15",
    "mobile": "9876543210",
    "address": "42 Civic Lane, North Ward",
    "city": "Metropolis",
    "district": "Central",
    "state": "Maharashtra",
    "pincode": "400001",
    "occupation": "Software Engineer"
  }
  ```

### 2.2 Get Citizen Profile by User ID
- **Method**: `GET`
- **Path**: `/api/citizens/user/{userId}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): CitizenResponse object.

### 2.3 Get Citizen by ID
- **Method**: `GET`
- **Path**: `/api/citizens/{id}`
- **Auth Required**: Yes (Invoked by `grievance-service` via OpenFeign)
- **Success Response** (`200 OK`): CitizenResponse object.

### 2.4 Total Citizen Count
- **Method**: `GET`
- **Path**: `/api/citizens/count`
- **Auth Required**: Yes (Invoked by `reporting-service` via OpenFeign)
- **Success Response** (`200 OK`):
  ```json
  {
    "totalCitizens": 48
  }
  ```

### 2.5 List All Citizens
- **Method**: `GET`
- **Path**: `/api/citizens`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Success Response** (`200 OK`): Array of CitizenResponse objects.

### 2.6 Update Citizen Profile
- **Method**: `PUT`
- **Path**: `/api/citizens/{id}`
- **Auth Required**: Yes
- **Request Body**: Partial demographic updates.
- **Success Response** (`200 OK`): Updated CitizenResponse object.

---

## 3. Department & Officer Service (`/api/departments/**`, `/api/officers/**`)

Downstream Service: `department-service` (Port: `8086`, Database: `department_db`)

### 3.1 Create Department
- **Method**: `POST`
- **Path**: `/api/departments`
- **Auth Required**: Yes (`ADMIN`)
- **Request Body**:
  ```json
  {
    "name": "WATER",
    "description": "Municipal Water Supply, Sanitation & Drainage Department"
  }
  ```
- **Success Response** (`201 Created`):
  ```json
  {
    "id": 1,
    "name": "WATER",
    "description": "Municipal Water Supply, Sanitation & Drainage Department"
  }
  ```

### 3.2 List All Departments
- **Method**: `GET`
- **Path**: `/api/departments`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Array of DepartmentResponse objects.

### 3.3 Get Department by ID
- **Method**: `GET`
- **Path**: `/api/departments/{id}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): DepartmentResponse object.

### 3.4 Create Officer
- **Method**: `POST`
- **Path**: `/api/officers`
- **Auth Required**: Yes (`ADMIN`)
- **Request Body**:
  ```json
  {
    "name": "Officer Smith",
    "email": "officer.smith@civicpulse.org",
    "departmentId": 1
  }
  ```
- **Success Response** (`201 Created`):
  ```json
  {
    "id": 1,
    "name": "Officer Smith",
    "email": "officer.smith@civicpulse.org",
    "departmentId": 1,
    "departmentName": "WATER"
  }
  ```

### 3.5 Get Officer by Email
- **Method**: `GET`
- **Path**: `/api/officers/by-email?email={email}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): OfficerResponse object.

### 3.6 Get Officers by Department
- **Method**: `GET`
- **Path**: `/api/officers/department/{departmentId}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Array of OfficerResponse objects.

### 3.7 Get Officer by ID
- **Method**: `GET`
- **Path**: `/api/officers/{id}`
- **Auth Required**: Yes (Invoked by `grievance-service` via OpenFeign)
- **Success Response** (`200 OK`): OfficerResponse object.

---

## 4. Grievance Service (`/api/grievances/**`, `/api/admin/**`)

Downstream Service: `grievance-service` (Port: `8083`, Database: `grievance_db`)

### 4.1 Create Grievance
- **Method**: `POST`
- **Path**: `/api/grievances`
- **Auth Required**: Yes (`CITIZEN`)
- **Request Body**:
  ```json
  {
    "citizenId": 1,
    "title": "Broken Water Main on 5th Avenue",
    "description": "Severe water leakage overflowing onto the main sidewalk.",
    "category": "WATER",
    "priority": "HIGH"
  }
  ```
- **Success Response** (`200 OK`):
  ```json
  {
    "id": 1,
    "citizenId": 1,
    "title": "Broken Water Main on 5th Avenue",
    "description": "Severe water leakage overflowing onto the main sidewalk.",
    "category": "WATER",
    "priority": "HIGH",
    "status": "NEW",
    "departmentId": 1,
    "assignedOfficerId": null,
    "remarks": null,
    "escalated": false,
    "createdAt": "2026-10-23T10:15:00",
    "updatedAt": "2026-10-23T10:15:00",
    "expectedResolution": "2026-10-24T10:15:00"
  }
  ```

### 4.2 Search & Filter Grievances (Dynamic Specification)
- **Method**: `GET`
- **Path**: `/api/grievances/search`
- **Auth Required**: Yes
- **Query Parameters**:
  - `keyword` (String, optional): Case-insensitive match on title/description
  - `status` (Enum, optional): `NEW`, `ASSIGNED`, `IN_PROGRESS`, `PENDING`, `RESOLVED`, `CLOSED`
  - `category` (Enum, optional): `ROAD`, `WATER`, `ELECTRICITY`, `SANITATION`, `DRAINAGE`, `STREET_LIGHT`, `GARBAGE`, `OTHER`
  - `priority` (Enum, optional): `LOW`, `MEDIUM`, `HIGH`, `URGENT`
  - `departmentId` (Long, optional)
  - `assignedOfficerId` (Long, optional)
  - `dueToday` (Boolean, default: `false`): Resolves grievances due within current calendar day
  - `page` (Integer, default: `0`)
  - `size` (Integer, default: `10`)
- **Success Response** (`200 OK`): Spring Data `Page<GrievanceResponse>` containing paginated content and total elements.

### 4.3 Get Grievance by ID
- **Method**: `GET`
- **Path**: `/api/grievances/{id}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): GrievanceResponse object.

### 4.4 Get Grievances by Citizen ID
- **Method**: `GET`
- **Path**: `/api/grievances/citizen/{citizenId}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Array of GrievanceResponse objects.

### 4.5 Get Grievances Assigned to Officer
- **Method**: `GET`
- **Path**: `/api/grievances/assigned/{officerId}`
- **Auth Required**: Yes (`OFFICER`)
- **Success Response** (`200 OK`): Array of GrievanceResponse objects.

### 4.6 Assign Department
- **Method**: `PUT`
- **Path**: `/api/grievances/{id}/department/{departmentId}`
- **Auth Required**: Yes (`ADMIN`)
- **Success Response** (`200 OK`): Updated GrievanceResponse with departmentId.

### 4.7 Assign Officer
- **Method**: `PUT`
- **Path**: `/api/grievances/{id}/officer/{officerId}`
- **Auth Required**: Yes (`ADMIN`)
- **Validation**: Officer must belong to assigned department (verified via Feign call).
- **Status Change**: Automatically moves status from `NEW` to `ASSIGNED`.
- **Success Response** (`200 OK`): Updated GrievanceResponse.

### 4.8 Transition Status
- **Method**: `PUT`
- **Path**: `/api/grievances/{id}/status/{status}`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Enforced Transitions**:
  - `NEW` -> `ASSIGNED`
  - `ASSIGNED` -> `IN_PROGRESS`
  - `IN_PROGRESS` -> `PENDING` or `RESOLVED`
  - `PENDING` -> `RESOLVED` or `IN_PROGRESS`
  - `RESOLVED` -> `CLOSED`
- **Success Response** (`200 OK`): Updated GrievanceResponse.
- **Error Response** (`400 Bad Request`): On invalid state transition.

### 4.9 Get Grievance Audit History Timeline
- **Method**: `GET`
- **Path**: `/api/grievances/{id}/history`
- **Auth Required**: Yes
- **Success Response** (`200 OK`):
  ```json
  [
    {
      "id": 1,
      "grievanceId": 1,
      "status": "NEW",
      "action": "GRIEVANCE_CREATED",
      "remarks": "Grievance created",
      "performedBy": 1,
      "createdAt": "2026-10-23T10:15:00"
    },
    {
      "id": 2,
      "grievanceId": 1,
      "status": "ASSIGNED",
      "action": "OFFICER_ASSIGNED",
      "remarks": "Officer assigned: 1",
      "performedBy": null,
      "createdAt": "2026-10-23T10:30:00"
    }
  ]
  ```

### 4.10 Grievance Summary Report
- **Method**: `GET`
- **Path**: `/api/grievances/report`
- **Auth Required**: No (Permitted in SecurityConfig for inter-service aggregation)
- **Success Response** (`200 OK`):
  ```json
  {
    "totalGrievances": 24,
    "resolvedGrievances": 18,
    "pendingGrievances": 6,
    "overdueGrievances": 2,
    "resolvedPercentage": 75.0
  }
  ```

### 4.11 Admin Dashboard & Reports
- **Method**: `GET`
- **Path**: `/api/admin/dashboard` & `/api/admin/reports`
- **Auth Required**: Yes (`ADMIN`)
- **Success Response** (`200 OK`): Aggregated statistics on category distribution and priority metrics.

---

## 5. Service Management Service (`/api/certificates/**`, `/api/certificate/**`)

Downstream Service: `service-management-service` (Port: `8087`, Database: `service_db`)

### 5.1 Apply for Certificate or Permit
- **Method**: `POST`
- **Path**: `/api/certificates/apply`
- **Auth Required**: Yes (`CITIZEN`)
- **Request Body**:
  ```json
  {
    "citizenId": 1,
    "applicantName": "John Doe",
    "dateOfBirth": "1990-01-15",
    "address": "42 Civic Lane",
    "phone": "9876543210",
    "email": "citizen@example.com",
    "aadhaarNumber": "123456789012",
    "serviceType": "BIRTH_CERTIFICATE",
    "department": "Revenue",
    "documents": [
      {
        "documentName": "Hospital Birth Card",
        "documentUrl": "https://docs.civicpulse.local/uploads/birth_card.pdf"
      }
    ]
  }
  ```
- **Success Response** (`201 Created`):
  ```json
  {
    "id": 1,
    "applicationNo": "APP-2026-0001",
    "citizenId": 1,
    "applicantName": "John Doe",
    "serviceType": "BIRTH_CERTIFICATE",
    "status": "SUBMITTED",
    "submissionDate": "2026-10-23T10:45:00",
    "documents": [
      {
        "id": 1,
        "documentName": "Hospital Birth Card",
        "documentUrl": "https://docs.civicpulse.local/uploads/birth_card.pdf",
        "verificationStatus": "PENDING"
      }
    ]
  }
  ```

### 5.2 Get My Applications
- **Method**: `GET`
- **Path**: `/api/certificates/my?citizenId={citizenId}`
- **Auth Required**: Yes (`CITIZEN`)
- **Success Response** (`200 OK`): Array of ApplicationResponse objects.

### 5.3 Officer Pending Queue
- **Method**: `GET`
- **Path**: `/api/certificates/pending`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Success Response** (`200 OK`): Array of applications with status `SUBMITTED`, `UNDER_VERIFICATION`, or `VERIFIED`.

### 5.4 Verify Application & Documents
- **Method**: `PUT`
- **Path**: `/api/certificates/{id}/verify`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Request Body**: `{"remarks": "Documents verified against municipal registry."}`
- **Success Response** (`200 OK`): Updated ApplicationResponse with status `VERIFIED`.

### 5.5 Approve Application
- **Method**: `PUT`
- **Path**: `/api/certificates/{id}/approve`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Request Body**: `{"remarks": "Application approved for issuance."}`
- **Success Response** (`200 OK`): Updated ApplicationResponse with status `APPROVED`.

### 5.6 Reject Application
- **Method**: `PUT`
- **Path**: `/api/certificates/{id}/reject`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Request Body**: `{"remarks": "Supporting hospital proof mismatch."}`
- **Success Response** (`200 OK`): Updated ApplicationResponse with status `REJECTED`.

### 5.7 Generate Digitally Signed Certificate
- **Method**: `POST`
- **Path**: `/api/certificate/generate/{id}`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Success Response** (`200 OK`):
  ```json
  {
    "id": 1,
    "applicationNo": "APP-2026-0001",
    "certificateNo": "BC-2026-1",
    "digitalSignature": "DIGITALLY-SIGNED-CIVICPULSE-1",
    "certificateHtml": "<html><body style='font-family:Georgia;padding:40px;border:8px solid #1e3a8a'><h1 style='text-align:center;color:#1e3a8a'>CivicPulse Municipal Corporation</h1><h2 style='text-align:center'>BIRTH CERTIFICATE</h2><p>Certificate No: <strong>BC-2026-1</strong></p><p>This is to certify that <strong>John Doe</strong> has been issued this document after due verification.</p><p>Issue Date: 2026-10-23</p><p>Digital Signature: DIGITALLY-SIGNED-CIVICPULSE-1</p></body></html>",
    "status": "CERTIFICATE_GENERATED",
    "issueDate": "2026-10-23T11:00:00"
  }
  ```

### 5.8 Certificate Report
- **Method**: `GET`
- **Path**: `/api/certificates/report`
- **Auth Required**: Yes
- **Success Response** (`200 OK`):
  ```json
  {
    "totalApplications": 12,
    "approvedApplications": 9,
    "rejectedApplications": 1,
    "certificatesIssued": 8,
    "permitsIssued": 1
  }
  ```

---

## 6. Welfare Service (`/api/welfare/**`)

Downstream Service: `welfare-service` (Port: `8088`, Database: `welfare_db`)

### 6.1 Create Welfare Scheme
- **Method**: `POST`
- **Path**: `/api/welfare/schemes`
- **Auth Required**: Yes (`ADMIN`)
- **Request Body**:
  ```json
  {
    "name": "Senior Citizen Healthcare Stipend",
    "description": "Monthly healthcare subsidy for residents over 60.",
    "department": "Social Welfare",
    "allocatedAmount": 500000.0,
    "minAge": 60,
    "maxAge": null,
    "maxIncome": 300000.0
  }
  ```
- **Success Response** (`200 OK`): Saved Scheme object with `active: true`.

### 6.2 List All Schemes
- **Method**: `GET`
- **Path**: `/api/welfare/schemes`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Array of Scheme objects.

### 6.3 Apply for Scheme (Automated Eligibility Evaluation)
- **Method**: `POST`
- **Path**: `/api/welfare/apply`
- **Auth Required**: Yes (`CITIZEN`)
- **Request Body**:
  ```json
  {
    "schemeId": 1,
    "citizenId": 1,
    "citizenName": "John Doe",
    "age": 65,
    "income": 180000.0,
    "ward": "Ward 4"
  }
  ```
- **Evaluation Rules**: Automatically evaluates `age >= minAge`, `age <= maxAge`, `income <= maxIncome`. Sets status to `ENROLLED` (eligible) or `INELIGIBLE`. Prevents duplicate registration.
- **Success Response** (`200 OK`): Saved Beneficiary record with initial `disbursedAmount: 0.0`.

### 6.4 List Beneficiaries
- **Method**: `GET`
- **Path**: `/api/welfare/beneficiaries?citizenId={citizenId}`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Array of Beneficiary objects.

### 6.5 Disburse Welfare Benefit
- **Method**: `POST`
- **Path**: `/api/welfare/disburse/{beneficiaryId}`
- **Auth Required**: Yes (`OFFICER`, `ADMIN`)
- **Request Body**: `{"amount": 2400.0}` (Optional; defaults to sanctioned amount)
- **Success Response** (`200 OK`):
  ```json
  {
    "id": 1,
    "beneficiaryId": 1,
    "schemeId": 1,
    "amount": 2400.0,
    "status": "PAID",
    "paidAt": "2026-10-23T11:15:00"
  }
  ```

### 6.6 Welfare Summary Report
- **Method**: `GET`
- **Path**: `/api/welfare/report`
- **Auth Required**: Yes
- **Success Response** (`200 OK`):
  ```json
  {
    "totalSchemes": 4,
    "totalBeneficiaries": 18,
    "allocatedAmount": 1500000.0,
    "disbursedAmount": 345600.0
  }
  ```

---

## 7. Budget Service (`/api/budget/**`)

Downstream Service: `budget-service` (Port: `8089`, Database: `budget_db`)

### 7.1 Create Department Budget
- **Method**: `POST`
- **Path**: `/api/budget`
- **Auth Required**: Yes (`ADMIN`)
- **Request Body**:
  ```json
  {
    "department": "Public Works",
    "period": "FY 2026-27",
    "allocated": 2500000.0
  }
  ```
- **Success Response** (`200 OK`): Saved Budget entity with `spent: 0.0`.

### 7.2 List All Budgets
- **Method**: `GET`
- **Path**: `/api/budget`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Array of Budget entities.

### 7.3 Log Departmental Expense
- **Method**: `POST`
- **Path**: `/api/budget/{id}/expense`
- **Auth Required**: Yes (`ADMIN`)
- **Request Body**:
  ```json
  {
    "description": "Asphalt Road resurfacing contract",
    "amount": 145000.0
  }
  ```
- **Side Effect**: Automatically increments `spent` on the parent Budget.
- **Success Response** (`200 OK`): Saved Expense entity.

### 7.4 List Expenses by Budget ID
- **Method**: `GET`
- **Path**: `/api/budget/{id}/expenses`
- **Auth Required**: Yes
- **Success Response** (`200 OK`): Array of Expense entities.

### 7.5 Budget Report
- **Method**: `GET`
- **Path**: `/api/budget/report`
- **Auth Required**: Yes
- **Success Response** (`200 OK`):
  ```json
  {
    "allocated": 5000000.0,
    "spent": 2150000.0,
    "remaining": 2850000.0,
    "budgetUtilization": 43.0,
    "revenueCollected": 903000.0
  }
  ```

---

## 8. Reporting Service (`/api/dashboard/**`, `/api/reports/**`)

Downstream Service: `reporting-service` (Port: `8090`, Stateless API Aggregator)

### 8.1 Executive Governance Dashboard
- **Method**: `GET`
- **Path**: `/api/dashboard`
- **Auth Required**: Yes (`ADMIN`)
- **Aggregation Source**: Aggregates `citizen-service::count`, `grievance-service::report`, `budget-service::report`, and `service-management-service::report` with resilient fallbacks.
- **Success Response** (`200 OK`):
  ```json
  {
    "totalCitizens": 48,
    "totalGrievances": 24,
    "resolvedGrievances": 18,
    "pendingGrievances": 6,
    "resolvedPercentage": 75.0,
    "revenueCollected": 903000.0,
    "budgetUtilization": 43.0,
    "allocated": 5000000.0,
    "spent": 2150000.0,
    "certificatesIssued": 8,
    "permitsIssued": 1,
    "citizenSatisfaction": 4.7,
    "slaCompliance": 94
  }
  ```

### 8.2 Domain Reports
- `GET /api/reports/citizens`: Demographic statistics from `citizen-service`
- `GET /api/reports/grievances`: Resolution breakdown from `grievance-service`
- `GET /api/reports/revenue`: Revenue overview from `budget-service`
- `GET /api/reports/budget`: Budget utilization breakdown
- `GET /api/reports/performance`: Combined performance metrics
