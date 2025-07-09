# Apartment Management System - Part 3: API Design & Service Architecture

## 1. Microservices Architecture Overview

### 1.1 Service Decomposition Strategy
```
┌─────────────────────────────────────────────────────────────┐
│                 Service Architecture Map                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │    Auth     │    │    User     │    │  Resident   │      │
│  │  Service    │    │  Service    │    │  Service    │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │   Worker    │    │   Vendor    │    │  Delivery   │      │
│  │  Service    │    │  Service    │    │  Service    │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │  Facility   │    │ Maintenance │    │  Financial  │      │
│  │  Service    │    │  Service    │    │  Service    │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │Communication│    │Notification │    │ Access Ctrl │      │
│  │  Service    │    │  Service    │    │  Service    │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Service Boundaries & Responsibilities

#### Core Business Services
1. **Authentication Service**
   - User authentication and authorization
   - JWT token management and validation
   - Multi-factor authentication
   - Session management and security

2. **User Management Service**
   - User profile management
   - Role and permission management
   - Account lifecycle management
   - User preferences and settings

3. **Resident Service**
   - Resident registration and onboarding
   - Lease management and tracking
   - Family member and dependent management
   - Resident communication preferences

4. **Worker Service**
   - Employee registration and management
   - Skills and certification tracking
   - Performance evaluation
   - Training record management

#### Operational Services
5. **Visitor Service**
   - Visitor registration and approval
   - Check-in/check-out processing
   - QR code generation and validation
   - Visitor history and analytics

6. **Vendor Service**
   - Vendor registration and verification
   - Access request management
   - Contractor and service provider tracking
   - Vendor performance and rating

7. **Delivery Service**
   - Package and delivery tracking
   - Delivery person verification
   - Resident notification system
   - Delivery completion confirmation

8. **Access Control Service**
   - Building access management
   - Security checkpoint processing
   - Emergency access protocols
   - Access audit and logging

#### Facility & Maintenance Services
9. **Facility Service**
   - Facility booking and scheduling
   - Resource availability management
   - Booking conflict resolution
   - Usage analytics and reporting

10. **Maintenance Service**
    - Work order creation and tracking
    - Asset management
    - Preventive maintenance scheduling
    - Vendor assignment and coordination

11. **Financial Service**
    - Billing and invoice generation
    - Payment processing
    - Financial reporting
    - Late fee calculation and management

#### Communication Services
12. **Communication Service**
    - Announcement broadcasting
    - Document sharing and management
    - Community forum moderation
    - Event management

13. **Notification Service**
    - Multi-channel notification delivery
    - Notification preferences management
    - Delivery confirmation tracking
    - Emergency alert system

## 2. API Gateway Design

### 2.1 Gateway Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway Layer                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                Rate Limiting                         │    │
│  │  • Per-user quotas: 1000 req/hour                  │    │
│  │  • Per-IP quotas: 10000 req/hour                   │    │
│  │  • Burst protection: 100 req/min                   │    │
│  │  • Service-specific limits                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │             Authentication Layer                     │    │
│  │  • JWT token validation                             │    │
│  │  • Role-based access control                       │    │
│  │  • API key validation for external services        │    │
│  │  • OAuth 2.0 integration                           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │               Request Routing                       │    │
│  │  • Service discovery integration                   │    │
│  │  • Load balancing algorithms                       │    │
│  │  • Circuit breaker patterns                        │    │
│  │  • Retry logic with exponential backoff            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Request/Response Processing               │    │
│  │  • Request validation and sanitization             │    │
│  │  • Response transformation and filtering           │    │
│  │  • Compression and caching                         │    │
│  │  • Audit logging and monitoring                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 API Versioning Strategy
```
┌─────────────────────────────────────────────────────────────┐
│                   API Versioning Design                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Version Strategy: URI-based versioning                    │
│  • /api/v1/... (Current stable version)                    │
│  • /api/v2/... (Next version in development)               │
│  • /api/beta/... (Beta features for testing)               │
│                                                             │
│  Backward Compatibility:                                   │
│  • Minimum 12 months support for deprecated versions       │
│  • Gradual migration tools and documentation               │
│  • Feature flags for smooth transitions                    │
│                                                             │
│  Breaking Change Management:                               │
│  • Major version increments for breaking changes           │
│  • Minor version increments for new features               │
│  • Patch version increments for bug fixes                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 3. Service-Specific API Specifications

### 3.1 Authentication Service API

#### Authentication Endpoints
```
POST /api/v1/auth/register
Purpose: User registration with email verification
Request Body:
{
  "email": "user@example.com",
  "phone": "+1234567890",
  "firstName": "John",
  "lastName": "Doe",
  "password": "securePassword123",
  "role": "resident",
  "buildingId": "uuid",
  "unitId": "uuid"
}

Response:
{
  "success": true,
  "message": "Registration successful. Please verify your email.",
  "userId": "uuid",
  "verificationRequired": true
}

POST /api/v1/auth/login
Purpose: User authentication with MFA support
Request Body:
{
  "email": "user@example.com",
  "password": "securePassword123",
  "mfaCode": "123456" // Optional if MFA enabled
}

Response:
{
  "success": true,
  "accessToken": "jwt_token",
  "refreshToken": "refresh_token",
  "expiresIn": 900, // 15 minutes
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "resident",
    "permissions": ["read:profile", "write:bookings"]
  }
}

POST /api/v1/auth/refresh
Purpose: Token refresh mechanism
Request Body:
{
  "refreshToken": "refresh_token"
}

Response:
{
  "accessToken": "new_jwt_token",
  "expiresIn": 900
}

POST /api/v1/auth/logout
Purpose: Session termination
Headers: Authorization: Bearer {access_token}
Response:
{
  "success": true,
  "message": "Logged out successfully"
}

POST /api/v1/auth/forgot-password
Purpose: Password reset initiation
Request Body:
{
  "email": "user@example.com"
}

Response:
{
  "success": true,
  "message": "Password reset email sent if account exists"
}
```

#### Security Features
- **Password Requirements**: Minimum 12 characters, mixed case, numbers, symbols
- **Account Lockout**: 5 failed attempts trigger 15-minute lockout
- **Session Management**: JWT with 15-minute expiry, refresh tokens with 7-day expiry
- **MFA Support**: TOTP-based two-factor authentication
- **Password History**: Prevent reuse of last 5 passwords

### 3.2 Visitor Management Service API

#### Visitor Registration & Management
```
POST /api/v1/visitors/register
Purpose: Register a new visitor with resident approval
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "residentId": "uuid",
  "visitorName": "Jane Smith",
  "visitorPhone": "+1234567890",
  "visitorEmail": "jane@example.com",
  "purposeOfVisit": "Family visit",
  "expectedCheckIn": "2024-01-15T10:00:00Z",
  "expectedCheckOut": "2024-01-15T18:00:00Z",
  "vehicleNumber": "ABC123",
  "numberOfPersons": 2,
  "specialRequirements": "Wheelchair access needed"
}

Response:
{
  "success": true,
  "visitorId": "uuid",
  "status": "pending_approval",
  "approvalRequired": true,
  "estimatedApprovalTime": "2024-01-15T09:00:00Z"
}

GET /api/v1/visitors?status=pending&buildingId=uuid&page=1&limit=10
Purpose: Retrieve visitor list with filtering
Headers: Authorization: Bearer {access_token}
Query Parameters:
- status: pending|approved|checked_in|checked_out|rejected
- buildingId: Building UUID filter
- residentId: Resident UUID filter
- dateFrom: Start date filter (ISO 8601)
- dateTo: End date filter (ISO 8601)
- page: Pagination page number
- limit: Items per page (max 100)

Response:
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "visitorName": "Jane Smith",
      "residentName": "John Doe",
      "unitNumber": "A-101",
      "status": "pending",
      "expectedCheckIn": "2024-01-15T10:00:00Z",
      "purposeOfVisit": "Family visit",
      "createdAt": "2024-01-14T15:30:00Z"
    }
  ],
  "pagination": {
    "total": 50,
    "page": 1,
    "limit": 10,
    "totalPages": 5
  }
}

PUT /api/v1/visitors/{visitorId}/approve
Purpose: Approve visitor access request
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "approved": true,
  "conditions": "Must be accompanied by resident",
  "validUntil": "2024-01-15T20:00:00Z",
  "accessAreas": ["lobby", "elevator", "unit_floor"],
  "parkingAllowed": true,
  "notes": "Approved for family visit"
}

Response:
{
  "success": true,
  "qrCode": "base64_encoded_qr_code",
  "accessCode": "123456",
  "validFrom": "2024-01-15T09:00:00Z",
  "validUntil": "2024-01-15T20:00:00Z"
}

POST /api/v1/visitors/{visitorId}/check-in
Purpose: Process visitor check-in
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "checkInMethod": "qr_code|manual|facial_recognition",
  "securityOfficerId": "uuid",
  "actualArrivalTime": "2024-01-15T10:15:00Z",
  "temperatureReading": 36.5,
  "healthDeclaration": {
    "symptoms": false,
    "recentTravel": false,
    "contactWithCovid": false
  },
  "photoUrl": "s3://bucket/visitor-photos/uuid.jpg",
  "accompanyingPersons": 1,
  "itemsBroughtIn": ["laptop bag", "gift box"]
}

Response:
{
  "success": true,
  "checkInTime": "2024-01-15T10:15:00Z",
  "accessGranted": true,
  "exitByTime": "2024-01-15T18:00:00Z",
  "accessAreas": ["lobby", "elevator", "unit_floor"],
  "emergencyContact": "+1234567890"
}
```

### 3.3 Worker Management Service API

#### Worker Registration & Scheduling
```
POST /api/v1/workers
Purpose: Register new worker
Headers: Authorization: Bearer {access_token}
Required Role: building_admin|super_admin
Request Body:
{
  "email": "worker@company.com",
  "firstName": "Mike",
  "lastName": "Johnson",
  "phone": "+1234567890",
  "employeeId": "EMP001",
  "workerType": "security_guard",
  "employmentType": "full_time",
  "buildingId": "uuid",
  "department": "Security",
  "position": "Security Guard",
  "supervisorId": "uuid",
  "hireDate": "2024-01-01",
  "salary": 45000,
  "skills": ["first_aid", "cctv_monitoring", "emergency_response"],
  "shiftPreference": "night",
  "emergencyContact": {
    "name": "Jane Johnson",
    "phone": "+1234567891",
    "relationship": "spouse"
  }
}

Response:
{
  "success": true,
  "workerId": "uuid",
  "userId": "uuid",
  "employeeId": "EMP001",
  "temporaryPassword": "generated_password",
  "onboardingRequired": true
}

GET /api/v1/workers/{workerId}/schedule?startDate=2024-01-01&endDate=2024-01-31
Purpose: Retrieve worker schedule
Headers: Authorization: Bearer {access_token}
Required Permission: read:worker_schedule

Response:
{
  "success": true,
  "workerId": "uuid",
  "schedules": [
    {
      "id": "uuid",
      "date": "2024-01-15",
      "shiftType": "morning",
      "startTime": "08:00",
      "endTime": "16:00",
      "isOvertime": false,
      "location": "Main Building",
      "status": "scheduled"
    }
  ],
  "totalHours": 160,
  "overtimeHours": 8
}

POST /api/v1/workers/{workerId}/attendance/check-in
Purpose: Worker clock-in
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "checkInTime": "2024-01-15T08:00:00Z",
  "location": {
    "latitude": 3.1390,
    "longitude": 101.6869,
    "accuracy": 10
  },
  "biometricVerification": true,
  "deviceId": "mobile_device_uuid",
  "notes": "On time arrival"
}

Response:
{
  "success": true,
  "attendanceId": "uuid",
  "checkInTime": "2024-01-15T08:00:00Z",
  "status": "present",
  "scheduledShift": {
    "startTime": "08:00",
    "endTime": "16:00",
    "breakTime": "12:00-13:00"
  },
  "lateMinutes": 0
}

POST /api/v1/workers/{workerId}/leave-request
Purpose: Submit leave request
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "leaveType": "annual_leave",
  "startDate": "2024-02-01",
  "endDate": "2024-02-05",
  "reason": "Family vacation",
  "emergencyContact": {
    "name": "Jane Johnson",
    "phone": "+1234567891"
  },
  "supportingDocuments": ["s3://bucket/leave-docs/medical_cert.pdf"],
  "replacementWorkerId": "uuid"
}

Response:
{
  "success": true,
  "leaveRequestId": "uuid",
  "status": "pending",
  "totalDays": 5,
  "approvalRequired": true,
  "approver": "supervisor_name"
}
```

### 3.4 Vendor Access Service API

#### Vendor Registration & Access Management
```
POST /api/v1/vendors
Purpose: Register new vendor
Headers: Authorization: Bearer {access_token}
Required Role: building_admin|super_admin
Request Body:
{
  "companyName": "ABC Contractors Ltd",
  "contactPersonName": "David Wilson",
  "contactPhone": "+1234567890",
  "contactEmail": "david@abc-contractors.com",
  "businessRegistrationNumber": "REG123456",
  "category": "home_improvement",
  "subcategory": "Electrical Work",
  "description": "Professional electrical installation and maintenance",
  "website": "https://abc-contractors.com",
  "businessAddress": "123 Business Street, City",
  "serviceAreas": ["building_uuid_1", "building_uuid_2"],
  "certifications": [
    {
      "name": "Electrical License",
      "issuingBody": "City Electrical Board",
      "number": "EL123456",
      "issueDate": "2023-01-01",
      "expiryDate": "2025-01-01"
    }
  ],
  "insuranceDetails": {
    "provider": "Insurance Co",
    "policyNumber": "POL123456",
    "coverageAmount": 1000000,
    "expiryDate": "2024-12-31"
  }
}

Response:
{
  "success": true,
  "vendorId": "uuid",
  "verificationRequired": true,
  "documentsRequired": ["business_license", "insurance_certificate"],
  "verificationTimeframe": "3-5 business days"
}

POST /api/v1/vendor-access/request
Purpose: Request vendor access
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "vendorId": "uuid",
  "buildingId": "uuid",
  "unitId": "uuid",
  "residentId": "uuid",
  "accessType": "contractor",
  "purpose": "Kitchen renovation",
  "detailedDescription": "Complete kitchen renovation including electrical and plumbing work",
  "scheduledStartDate": "2024-02-01",
  "scheduledEndDate": "2024-02-15",
  "workingHoursStart": "08:00",
  "workingHoursEnd": "17:00",
  "durationType": "project_based",
  "workersCount": 4,
  "workerDetails": [
    {
      "name": "John Smith",
      "role": "Electrician",
      "phone": "+1234567890",
      "idNumber": "ID123456",
      "certifications": ["Electrical License"]
    }
  ],
  "vehicles": [
    {
      "type": "van",
      "licensePlate": "ABC123",
      "driverName": "John Smith"
    }
  ],
  "equipmentList": ["power tools", "electrical supplies", "safety equipment"],
  "safetyRequirements": "PPE required, electrical safety protocols",
  "insuranceRequired": true,
  "estimatedCost": 15000
}

Response:
{
  "success": true,
  "accessRequestId": "uuid",
  "status": "pending",
  "approvalRequired": true,
  "approvers": ["building_admin", "resident"],
  "estimatedApprovalTime": "2024-01-20T12:00:00Z",
  "requirements": [
    "Insurance certificate verification",
    "Background check completion",
    "Safety protocol acknowledgment"
  ]
}

PUT /api/v1/vendor-access/{requestId}/approve
Purpose: Approve vendor access request
Headers: Authorization: Bearer {access_token}
Required Permission: approve:vendor_access
Request Body:
{
  "approved": true,
  "conditions": [
    "Work only during approved hours",
    "Notify security before bringing equipment",
    "Clean up required daily"
  ],
  "approvalNotes": "Approved with conditions",
  "accessDuration": {
    "validFrom": "2024-02-01T08:00:00Z",
    "validUntil": "2024-02-15T17:00:00Z"
  },
  "parkingAllowed": true,
  "liftUsageAllowed": true,
  "specialInstructions": "Coordinate with building management for major deliveries"
}

Response:
{
  "success": true,
  "accessApproved": true,
  "qrCodeAccess": "base64_encoded_qr_code",
  "accessCardNumbers": ["CARD001", "CARD002"],
  "validFrom": "2024-02-01T08:00:00Z",
  "validUntil": "2024-02-15T17:00:00Z",
  "emergencyContact": "+1234567890"
}
```

### 3.5 Delivery Management Service API

#### Delivery Tracking & Management
```
POST /api/v1/deliveries
Purpose: Register new delivery
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "buildingId": "uuid",
  "unitId": "uuid",
  "residentId": "uuid",
  "deliveryType": "package_delivery",
  "deliveryServiceName": "FedEx",
  "trackingNumber": "TRK123456789",
  "senderName": "Amazon",
  "packageDescription": "Electronics - Laptop",
  "packageCount": 1,
  "packageDimensions": {
    "length": 40,
    "width": 30,
    "height": 10,
    "weight": 2.5
  },
  "specialHandlingInstructions": "Fragile - Handle with care",
  "requiresSignature": true,
  "requiresPhotoConfirmation": true,
  "isFragile": true,
  "estimatedDeliveryTime": "2024-01-15T14:00:00Z",
  "deliveryWindowStart": "2024-01-15T12:00:00Z",
  "deliveryWindowEnd": "2024-01-15T16:00:00Z"
}

Response:
{
  "success": true,
  "deliveryId": "uuid",
  "status": "scheduled",
  "residentNotified": true,
  "deliveryInstructions": "Contact recipient 30 minutes before delivery",
  "securityNotified": true
}

GET /api/v1/deliveries?buildingId=uuid&status=pending&page=1&limit=20
Purpose: Retrieve delivery list
Headers: Authorization: Bearer {access_token}
Query Parameters:
- buildingId: Building filter
- unitId: Unit filter
- residentId: Resident filter
- status: scheduled|out_for_delivery|arrived|delivered|failed_delivery
- deliveryType: package_delivery|food_delivery|grocery_delivery
- dateFrom: Start date filter
- dateTo: End date filter

Response:
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "trackingNumber": "TRK123456789",
      "deliveryServiceName": "FedEx",
      "recipientName": "John Doe",
      "unitNumber": "A-101",
      "status": "out_for_delivery",
      "estimatedArrival": "2024-01-15T14:00:00Z",
      "deliveryPersonName": "Mike Delivery",
      "deliveryPersonPhone": "+1234567890",
      "packageCount": 1,
      "specialHandling": ["fragile", "signature_required"]
    }
  ],
  "pagination": {
    "total": 15,
    "page": 1,
    "limit": 20,
    "totalPages": 1
  }
}

PUT /api/v1/deliveries/{deliveryId}/arrived
Purpose: Mark delivery as arrived
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "arrivalTime": "2024-01-15T14:15:00Z",
  "deliveryPersonName": "Mike Delivery",
  "deliveryPersonPhone": "+1234567890",
  "vehicleNumber": "DEL123",
  "deliveryPersonPhoto": "s3://bucket/delivery-photos/uuid.jpg",
  "packagePhotos": ["s3://bucket/packages/uuid1.jpg"],
  "requiresAssistance": false,
  "specialNotes": "Package larger than expected"
}

Response:
{
  "success": true,
  "status": "arrived",
  "residentNotified": true,
  "securityInformed": true,
  "estimatedCompletionTime": "2024-01-15T14:30:00Z"
}

PUT /api/v1/deliveries/{deliveryId}/complete
Purpose: Complete delivery
Headers: Authorization: Bearer {access_token}
Request Body:
{
  "completionTime": "2024-01-15T14:25:00Z",
  "deliveryLocation": "unit_door",
  "receivedByName": "John Doe",
  "receivedByPhone": "+1234567890",
  "signatureImage": "base64_encoded_signature",
  "deliveryPhotos": ["s3://bucket/completion/uuid.jpg"],
  "deliveryNotes": "Package delivered successfully to resident",
  "packageCondition": "good",
  "residentSatisfied": true
}

Response:
{
  "success": true,
  "status": "delivered",
  "completionTime": "2024-01-15T14:25:00Z",
  "deliveryConfirmation": "CONF123456",
  "residentNotified": true
}
```

## 4. Inter-Service Communication

### 4.1 Event-Driven Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                Event-Driven Communication                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Event Bus: Redis Pub/Sub + Message Queue                  │
│                                                             │
│  Event Types:                                               │
│  • user.created                                             │
│  • visitor.approved                                         │
│  • visitor.checked_in                                       │
│  • delivery.arrived                                         │
│  • worker.checked_in                                        │
│  • maintenance.completed                                    │
│  • payment.received                                         │
│  • emergency.alert                                          │
│                                                             │
│  Event Processing:                                          │
│  • Asynchronous event handlers                             │
│  • Event replay capability                                 │
│  • Dead letter queue for failed events                     │
│  • Event versioning and backward compatibility             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Service Discovery & Load Balancing
```
┌─────────────────────────────────────────────────────────────┐
│              Service Discovery Pattern                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Service Registry: Kubernetes Service Discovery            │
│  • Automatic service registration                          │
│  • Health check integration                                │
│  • Service mesh integration (Istio)                        │
│                                                             │
│  Load Balancing Strategies:                                │
│  • Round-robin for stateless services                      │
│  • Least connections for database services                 │
│  • Consistent hashing for cache services                   │
│  • Geographic routing for CDN services                     │
│                                                             │
│  Circuit Breaker Pattern:                                  │
│  • Failure threshold: 5 failures in 1 minute              │
│  • Half-open state: 30-second recovery period              │
│  • Fallback mechanisms for degraded service                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 API Security & Rate Limiting
```
┌─────────────────────────────────────────────────────────────┐
│                  API Security Framework                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Rate Limiting Rules:                                       │
│  • Public APIs: 100 requests/minute                        │
│  • Authenticated APIs: 1000 requests/minute                │
│  • Admin APIs: 10000 requests/minute                       │
│  • File upload APIs: 10 requests/minute                    │
│  • Search APIs: 50 requests/minute                         │
│                                                             │
│  Security Headers:                                          │
│  • X-Content-Type-Options: nosniff                         │
│  • X-Frame-Options: DENY                                   │
│  • X-XSS-Protection: 1; mode=block                         │
│  • Content-Security-Policy: strict policy                  │
│  • Strict-Transport-Security: max-age=31536000             │
│                                                             │
│  Input Validation:                                          │
│  • Schema-based validation (JSON Schema)                   │
│  • SQL injection prevention                                │
│  • XSS attack prevention                                   │
│  • File upload restrictions                                │
│  • Request size limitations                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 5. Error Handling & Monitoring

### 5.1 Standardized Error Responses
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input parameters",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "phone",
        "message": "Phone number is required"
      }
    ],
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_uuid",
    "support": {
      "contactEmail": "support@apartmentmanagement.com",
      "documentationUrl": "https://docs.apartmentmanagement.com/errors/VALIDATION_ERROR"
    }
  }
}
```

### 5.2 Health Check Endpoints
```
GET /health
Purpose: Basic service health check
Response:
{
  "status": "healthy",
  "timestamp": "2024-01-15T10:30:00Z",
  "version": "1.2.3",
  "uptime": 3600
}

GET /health/detailed
Purpose: Comprehensive health check
Response:
{
  "status": "healthy",
  "checks": {
    "database": {
      "status": "healthy",
      "responseTime": 15,
      "lastChecked": "2024-01-15T10:30:00Z"
    },
    "cache": {
      "status": "healthy",
      "responseTime": 2,
      "lastChecked": "2024-01-15T10:30:00Z"
    },
    "external_apis": {
      "status": "degraded",
      "details": {
        "payment_gateway": "timeout",
        "sms_service": "healthy"
      }
    }
  }
}
```

### 5.3 Monitoring & Observability
```
┌─────────────────────────────────────────────────────────────┐
│                Monitoring Strategy                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Application Metrics:                                       │
│  • Request count and response times                        │
│  • Error rates and success rates                           │
│  • Database query performance                              │
│  • Cache hit/miss ratios                                   │
│  • Business-specific metrics                               │
│                                                             │
│  Infrastructure Metrics:                                   │
│  • CPU, memory, and disk usage                             │
│  • Network bandwidth and latency                           │
│  • Container and pod health                                │
│  • Load balancer performance                               │
│                                                             │
│  Alerting Rules:                                            │
│  • Error rate > 5% for 5 minutes                           │
│  • Response time > 2 seconds for 3 minutes                 │
│  • CPU usage > 80% for 10 minutes                          │
│  • Memory usage > 90% for 5 minutes                        │
│  • Service unavailable for 1 minute                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

This completes Part 3 of the API design and service architecture specifications.
