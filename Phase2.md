# Apartment Management System - Part 2: Database Design & Data Models

## 1. Database Architecture Overview

### 1.1 Multi-Database Strategy
```
┌─────────────────────────────────────────────────────────────┐
│                    Database Layer Design                    │
├─────────────────────────────────────────────────────────────┤
│ Primary Database: PostgreSQL 15+                           │
│ • ACID compliance for critical transactions                │
│ • Advanced indexing and query optimization                 │
│ • JSON support for flexible schema requirements            │
│ • Full-text search capabilities                            │
│ • Spatial data support for location features               │
├─────────────────────────────────────────────────────────────┤
│ Cache Database: Redis 7+                                   │
│ • Session storage and user authentication tokens           │
│ • Real-time data caching (visitor status, facility status) │
│ • Pub/Sub for real-time notifications                      │
│ • Rate limiting data and temporary locks                   │
│ • Queue management for background jobs                     │
├─────────────────────────────────────────────────────────────┤
│ Search Engine: Elasticsearch 8+                            │
│ • Full-text search across announcements and documents      │
│ • Log aggregation and analysis                             │
│ • Business intelligence and reporting                      │
│ • Audit trail search and compliance reporting              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Data Consistency & ACID Properties
- **Transaction Boundaries**: Clearly defined transaction scopes for business operations
- **Isolation Levels**: Read Committed for most operations, Serializable for financial transactions
- **Concurrency Control**: Optimistic locking for high-concurrency operations
- **Data Integrity**: Foreign key constraints, check constraints, and trigger-based validation
- **Backup Strategy**: Point-in-time recovery with 15-minute granularity

## 2. Core Entity Relationship Design

### 2.1 User Management Entities
```
┌─────────────────────────────────────────────────────────────┐
│                     User Hierarchy                          │
│                                                             │
│    ┌─────────────┐                                          │
│    │    Users    │ (Base entity for all user types)        │
│    │             │                                          │
│    ├─────────────┤                                          │
│    │ Residents   │ (Extends Users)                          │
│    │             │                                          │
│    ├─────────────┤                                          │
│    │   Workers   │ (Extends Users)                          │
│    │             │                                          │
│    ├─────────────┤                                          │
│    │  Visitors   │ (References Residents)                   │
│    │             │                                          │
│    └─────────────┘                                          │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Building & Property Management
```
┌─────────────────────────────────────────────────────────────┐
│              Building Structure Hierarchy                   │
│                                                             │
│  Buildings (1) ←──── (M) Units ←──── (M) Residents         │
│      │                   │                                  │
│      │                   └──── (M) Lease_Agreements        │
│      │                                                      │
│      ├──── (M) Facilities                                   │
│      │         │                                            │
│      │         └──── (M) Facility_Bookings                 │
│      │                                                      │
│      ├──── (M) Workers                                      │
│      │         │                                            │
│      │         ├──── (M) Work_Shifts                       │
│      │         ├──── (M) Worker_Schedules                  │
│      │         └──── (M) Worker_Attendance                 │
│      │                                                      │
│      └──── (M) Service_Provider_Bookings                   │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 Access Control & Security Entities
```
┌─────────────────────────────────────────────────────────────┐
│                 Access Control Design                       │
│                                                             │
│  Visitors ──────┐                                          │
│  Vendors ───────┼──→ Access_Logs ←──── Buildings           │
│  Deliveries ────┘         │                                │
│  Workers ─────────────────┘                                │
│                                                             │
│  Access_Logs contains:                                      │
│  • Entry/exit timestamps                                    │
│  • Authentication method used                               │
│  • Security checkpoints passed                             │
│  • Items brought in/taken out                              │
│  • Photos and biometric data                               │
└─────────────────────────────────────────────────────────────┘
```

## 3. Detailed Entity Specifications

### 3.1 Core Building Management Tables

#### Buildings Table
```sql
Purpose: Central registry of all managed apartment buildings
Key Features:
• Multi-tenant architecture support
• Configurable settings per building
• Geographic information for location services
• Amenity management and facility tracking

Fields Specification:
- id: UUID (Primary Key, Auto-generated)
- name: VARCHAR(255) NOT NULL (Building display name)
- address: TEXT NOT NULL (Full postal address)
- coordinates: POINT (GPS coordinates for mapping)
- total_units: INTEGER NOT NULL (Total residential units)
- total_floors: INTEGER NOT NULL (Number of floors)
- amenities: JSONB (Flexible amenity list storage)
- contact_info: JSONB (Management contact details)
- settings: JSONB (Building-specific configurations)
- operating_hours: JSONB (Service hours and restrictions)
- emergency_contacts: JSONB (Emergency response contacts)
- insurance_details: JSONB (Building insurance information)
- construction_year: INTEGER (Building age for maintenance)
- last_renovation: DATE (Major renovation tracking)
- building_code: VARCHAR(50) UNIQUE (Government building identifier)
- management_company: VARCHAR(255) (Property management company)
```

#### Units Table
```sql
Purpose: Individual residential/commercial unit registry
Key Features:
• Flexible unit typing and categorization
• Occupancy tracking and lease management
• Utility and maintenance fee calculations
• Parking allocation management

Fields Specification:
- id: UUID (Primary Key)
- building_id: UUID NOT NULL (Foreign Key to Buildings)
- unit_number: VARCHAR(50) NOT NULL (Unit identifier)
- floor_number: INTEGER NOT NULL (Physical floor location)
- unit_type: VARCHAR(50) (1BHK, 2BHK, Studio, Commercial, etc.)
- carpet_area: DECIMAL(10,2) (Floor area in square meters)
- balcony_area: DECIMAL(10,2) (Balcony area if applicable)
- monthly_maintenance: DECIMAL(10,2) (Base maintenance fee)
- parking_slots: INTEGER DEFAULT 0 (Allocated parking spaces)
- storage_unit: BOOLEAN DEFAULT FALSE (Storage space allocation)
- is_occupied: BOOLEAN DEFAULT FALSE (Current occupancy status)
- move_in_ready: BOOLEAN DEFAULT TRUE (Available for occupancy)
- last_inspection: DATE (Safety/maintenance inspection)
- utility_meters: JSONB (Electricity, water, gas meter numbers)
- special_features: JSONB (Elevator access, garden access, etc.)
```

### 3.2 User Management & Authentication

#### Users Table (Base Entity)
```sql
Purpose: Universal user registry for all system participants
Key Features:
• Role-based access control foundation
• Multi-factor authentication support
• Privacy compliance and data protection
• Comprehensive profile management

Fields Specification:
- id: UUID (Primary Key)
- email: VARCHAR(255) UNIQUE NOT NULL (Primary identifier)
- phone: VARCHAR(20) UNIQUE (Mobile number with country code)
- password_hash: VARCHAR(255) NOT NULL (Bcrypt hashed password)
- first_name: VARCHAR(100) NOT NULL (Given name)
- last_name: VARCHAR(100) NOT NULL (Family name)
- middle_name: VARCHAR(100) (Middle name if applicable)
- role: user_role ENUM NOT NULL (System role assignment)
- status: user_status ENUM (Account status tracking)
- profile_picture_url: TEXT (Avatar/photo URL)
- date_of_birth: DATE (Age verification and demographics)
- gender: VARCHAR(20) (Optional demographic information)
- nationality: VARCHAR(100) (Document and compliance tracking)
- preferred_language: VARCHAR(10) DEFAULT 'en' (Localization)
- timezone: VARCHAR(50) (User timezone for scheduling)
- emergency_contact: JSONB (Emergency contact information)
- preferences: JSONB (User preferences and settings)
- privacy_settings: JSONB (Privacy and consent management)
- mfa_enabled: BOOLEAN DEFAULT FALSE (Two-factor authentication)
- mfa_secret: VARCHAR(255) (TOTP secret key)
- email_verified_at: TIMESTAMP (Email verification timestamp)
- phone_verified_at: TIMESTAMP (Phone verification timestamp)
- last_login: TIMESTAMP (Last successful login)
- login_attempts: INTEGER DEFAULT 0 (Failed login tracking)
- account_locked_until: TIMESTAMP (Temporary account lockout)
```

#### Residents Table (Extends Users)
```sql
Purpose: Resident-specific information and building association
Key Features:
• Multi-unit resident support
• Family member management
• Vehicle and pet registration
• Lease agreement tracking

Fields Specification:
- id: UUID (Primary Key)
- user_id: UUID NOT NULL (Foreign Key to Users)
- building_id: UUID NOT NULL (Foreign Key to Buildings)
- unit_id: UUID NOT NULL (Foreign Key to Units)
- resident_type: VARCHAR(50) (Owner, Tenant, Guest, etc.)
- is_primary_resident: BOOLEAN DEFAULT FALSE (Main contact person)
- lease_start_date: DATE (Occupancy start date)
- lease_end_date: DATE (Lease expiration date)
- lease_type: VARCHAR(50) (Rental, Owned, Corporate, etc.)
- move_in_date: DATE (Actual move-in date)
- move_out_date: DATE (Move-out date if applicable)
- security_deposit: DECIMAL(10,2) (Deposit amount)
- monthly_rent: DECIMAL(10,2) (Rent amount for tenants)
- family_members: JSONB (Family member details)
- vehicles: JSONB (Registered vehicle information)
- pets: JSONB (Pet registration details)
- special_needs: JSONB (Accessibility requirements)
- communication_preferences: JSONB (Preferred contact methods)
- billing_preferences: JSONB (Payment methods and schedules)
```

### 3.3 Worker Management System

#### Workers Table
```sql
Purpose: Employee registry and workforce management
Key Features:
• Comprehensive employee lifecycle management
• Skills and certification tracking
• Performance monitoring integration
• Compliance and document management

Fields Specification:
- id: UUID (Primary Key)
- user_id: UUID NOT NULL (Foreign Key to Users)
- building_id: UUID NOT NULL (Primary workplace assignment)
- employee_id: VARCHAR(50) UNIQUE NOT NULL (Company employee number)
- worker_type: worker_type ENUM (Job category classification)
- employment_type: employment_type ENUM (Full-time, part-time, etc.)
- department: VARCHAR(100) (Organizational department)
- position: VARCHAR(100) NOT NULL (Job title)
- supervisor_id: UUID (Foreign Key to Workers - reporting hierarchy)
- hire_date: DATE NOT NULL (Employment start date)
- probation_end_date: DATE (Probationary period end)
- termination_date: DATE (Employment end date)
- salary: DECIMAL(10,2) (Annual salary for salaried employees)
- hourly_rate: DECIMAL(8,2) (Hourly rate for hourly employees)
- overtime_rate: DECIMAL(8,2) (Overtime hourly rate)
- contract_end_date: DATE (Contract worker end date)
- skills: JSONB (Skill set and competencies)
- certifications: JSONB (Professional certifications)
- work_location: VARCHAR(255) (Primary work area)
- shift_preference: shift_type ENUM (Preferred working hours)
- emergency_contact: JSONB (Emergency contact information)
- bank_details: JSONB (Payroll bank account details)
- documents: JSONB (HR documents and compliance records)
- performance_rating: DECIMAL(3,2) (Latest performance score)
- background_check_status: VARCHAR(50) (Security clearance status)
- uniform_size: JSONB (Uniform and equipment sizing)
- training_records: JSONB (Training completion records)
- disciplinary_records: JSONB (Disciplinary action history)
```

#### Worker Schedules & Attendance
```sql
Purpose: Workforce scheduling and time tracking
Key Features:
• Flexible shift management
• Attendance tracking with GPS verification
• Overtime calculation and management
• Leave integration and coverage planning

Work_Shifts Table:
- id: UUID (Primary Key)
- building_id: UUID NOT NULL (Foreign Key to Buildings)
- name: VARCHAR(100) NOT NULL (Shift identifier)
- shift_type: shift_type ENUM (Morning, night, etc.)
- start_time: TIME NOT NULL (Shift start time)
- end_time: TIME NOT NULL (Shift end time)
- days_of_week: INTEGER[] (Operating days [1,2,3,4,5])
- required_workers: INTEGER DEFAULT 1 (Minimum staffing)
- worker_type: worker_type ENUM (Required worker category)
- break_duration_minutes: INTEGER DEFAULT 30 (Paid break time)
- overtime_threshold_minutes: INTEGER (Overtime trigger)

Worker_Schedules Table:
- id: UUID (Primary Key)
- worker_id: UUID NOT NULL (Foreign Key to Workers)
- shift_id: UUID NOT NULL (Foreign Key to Work_Shifts)
- schedule_date: DATE NOT NULL (Specific work date)
- start_time: TIME NOT NULL (Scheduled start time)
- end_time: TIME NOT NULL (Scheduled end time)
- is_overtime: BOOLEAN DEFAULT FALSE (Overtime shift flag)
- coverage_for: UUID (Foreign Key to Workers if covering)
- notes: TEXT (Schedule-specific notes)

Worker_Attendance Table:
- id: UUID (Primary Key)
- worker_id: UUID NOT NULL (Foreign Key to Workers)
- schedule_id: UUID (Foreign Key to Worker_Schedules)
- attendance_date: DATE NOT NULL (Attendance date)
- actual_check_in: TIME (Actual clock-in time)
- actual_check_out: TIME (Actual clock-out time)
- check_in_location: POINT (GPS coordinates of check-in)
- check_out_location: POINT (GPS coordinates of check-out)
- status: attendance_status ENUM (Present, absent, late, etc.)
- total_hours_worked: DECIMAL(4,2) (Calculated work hours)
- overtime_hours: DECIMAL(4,2) (Overtime hours earned)
- break_duration: INTEGER (Break time taken in minutes)
- late_minutes: INTEGER (Minutes late to work)
- early_departure_minutes: INTEGER (Early departure time)
```

### 3.4 Vendor & Service Provider Management

#### Vendors Table
```sql
Purpose: External service provider and contractor registry
Key Features:
• Vendor qualification and verification
• Service category management
• Performance tracking and rating system
• Compliance and insurance tracking

Fields Specification:
- id: UUID (Primary Key)
- company_name: VARCHAR(255) NOT NULL (Business name)
- contact_person_name: VARCHAR(255) NOT NULL (Primary contact)
- contact_phone: VARCHAR(20) NOT NULL (Business phone)
- contact_email: VARCHAR(255) (Business email)
- business_registration_number: VARCHAR(100) (Legal registration ID)
- tax_identification: VARCHAR(100) (Tax ID number)
- category: vendor_category ENUM (Service classification)
- subcategory: VARCHAR(100) (Detailed service type)
- description: TEXT (Business description)
- website: VARCHAR(255) (Company website)
- business_address: TEXT (Registered business address)
- service_areas: JSONB (Geographic service coverage)
- certifications: JSONB (Professional certifications)
- insurance_details: JSONB (Liability insurance information)
- license_information: JSONB (Business licenses)
- documents: JSONB (Legal and compliance documents)
- rating: DECIMAL(3,2) DEFAULT 0 (Average customer rating)
- total_reviews: INTEGER DEFAULT 0 (Review count)
- total_jobs_completed: INTEGER DEFAULT 0 (Job completion count)
- is_verified: BOOLEAN DEFAULT FALSE (Verification status)
- verification_date: DATE (Verification completion date)
- is_blacklisted: BOOLEAN DEFAULT FALSE (Blacklist status)
- blacklist_reason: TEXT (Reason for blacklisting)
- preferred_working_hours: JSONB (Availability schedule)
- emergency_contact: JSONB (Emergency contact information)
- payment_terms: TEXT (Payment terms and conditions)
- service_guarantee: TEXT (Service warranty information)
```

#### Vendor Access Requests
```sql
Purpose: Vendor access control and approval workflow
Key Features:
• Multi-step approval process
• Duration-based access control
• Worker and equipment tracking
• Safety and insurance compliance

Fields Specification:
- id: UUID (Primary Key)
- vendor_id: UUID (Foreign Key to Vendors)
- building_id: UUID NOT NULL (Foreign Key to Buildings)
- unit_id: UUID (Foreign Key to Units if unit-specific)
- resident_id: UUID (Foreign Key to Residents if resident-requested)
- access_type: access_type ENUM (Service classification)
- purpose: VARCHAR(255) NOT NULL (Access purpose)
- detailed_description: TEXT (Detailed work description)
- scheduled_start_date: DATE NOT NULL (Planned start date)
- scheduled_end_date: DATE NOT NULL (Planned completion date)
- scheduled_start_time: TIME (Daily start time)
- scheduled_end_time: TIME (Daily end time)
- duration_type: access_duration_type ENUM (Access duration type)
- recurring_days: INTEGER[] (Recurring access days)
- workers_count: INTEGER DEFAULT 1 (Number of workers)
- worker_details: JSONB (Worker information and identification)
- vehicles: JSONB (Vehicle details and parking requirements)
- equipment_list: JSONB (Equipment and tools list)
- safety_requirements: TEXT (Safety protocols and requirements)
- insurance_required: BOOLEAN DEFAULT FALSE (Insurance requirement)
- insurance_amount: DECIMAL(12,2) (Required insurance coverage)
- deposit_amount: DECIMAL(10,2) DEFAULT 0 (Security deposit)
- estimated_cost: DECIMAL(10,2) (Project cost estimate)
- special_instructions: TEXT (Special requirements)
- status: visitor_status DEFAULT 'pending' (Approval status)
- approved_by: UUID (Foreign Key to Users - approver)
- approved_at: TIMESTAMP (Approval timestamp)
- approval_conditions: TEXT (Conditional approval terms)
- rejection_reason: TEXT (Rejection explanation)
- qr_code_access: TEXT (QR code for access)
- access_card_numbers: JSONB (Physical access card numbers)
```

### 3.5 Delivery Management System

#### Deliveries Table
```sql
Purpose: Package and delivery tracking system
Key Features:
• Multi-modal delivery support
• Real-time tracking integration
• Resident notification system
• Security verification and photo capture

Fields Specification:
- id: UUID (Primary Key)
- building_id: UUID NOT NULL (Foreign Key to Buildings)
- unit_id: UUID NOT NULL (Foreign Key to Units)
- resident_id: UUID NOT NULL (Foreign Key to Residents)
- delivery_type: delivery_type ENUM (Package, food, grocery, etc.)
- delivery_service_name: VARCHAR(255) NOT NULL (Delivery company)
- tracking_number: VARCHAR(255) (Package tracking number)
- order_number: VARCHAR(255) (Order reference number)
- sender_name: VARCHAR(255) (Sender information)
- sender_phone: VARCHAR(20) (Sender contact)
- delivery_person_name: VARCHAR(255) (Delivery person name)
- delivery_person_phone: VARCHAR(20) (Delivery person contact)
- delivery_person_photo_url: TEXT (Delivery person photo)
- vehicle_number: VARCHAR(20) (Delivery vehicle number)
- package_description: TEXT (Package contents description)
- package_count: INTEGER DEFAULT 1 (Number of packages)
- package_dimensions: JSONB (Length, width, height, weight)
- package_value: DECIMAL(10,2) (Declared package value)
- special_handling_instructions: TEXT (Handling requirements)
- requires_signature: BOOLEAN DEFAULT FALSE (Signature requirement)
- requires_photo_confirmation: BOOLEAN DEFAULT FALSE (Photo requirement)
- requires_id_verification: BOOLEAN DEFAULT FALSE (ID check requirement)
- is_fragile: BOOLEAN DEFAULT FALSE (Fragile item flag)
- is_perishable: BOOLEAN DEFAULT FALSE (Perishable item flag)
- temperature_requirements: TEXT (Storage temperature needs)
- estimated_delivery_time: TIMESTAMP (Expected delivery time)
- delivery_window_start: TIMESTAMP (Delivery window start)
- delivery_window_end: TIMESTAMP (Delivery window end)
- actual_arrival_time: TIMESTAMP (Actual arrival timestamp)
- delivery_completed_time: TIMESTAMP (Completion timestamp)
- status: delivery_status DEFAULT 'scheduled' (Delivery status)
- delivery_location: VARCHAR(255) (Final delivery location)
- delivery_photos: JSONB (Delivery confirmation photos)
- signature_image_url: TEXT (Digital signature capture)
- delivery_notes: TEXT (Delivery completion notes)
- resident_notified: BOOLEAN DEFAULT FALSE (Notification sent flag)
- security_received: BOOLEAN DEFAULT FALSE (Security desk receipt)
- received_by_name: VARCHAR(255) (Person who received package)
- received_by_phone: VARCHAR(20) (Receiver contact number)
- failed_delivery_reason: TEXT (Failed delivery explanation)
- rescheduled_time: TIMESTAMP (Rescheduled delivery time)
- return_tracking_number: VARCHAR(255) (Return shipment tracking)
```

## 4. Advanced Data Management Features

### 4.1 Audit Trail & Compliance
```sql
Purpose: Comprehensive activity logging for compliance and security
Key Features:
• Immutable audit records
• GDPR compliance support
• Security event tracking
• Business process auditing

Audit_Logs Table:
- id: UUID (Primary Key)
- table_name: VARCHAR(100) NOT NULL (Source table)
- record_id: UUID NOT NULL (Source record ID)
- action: VARCHAR(50) NOT NULL (INSERT, UPDATE, DELETE)
- old_values: JSONB (Previous field values)
- new_values: JSONB (Updated field values)
- changed_fields: TEXT[] (List of modified fields)
- user_id: UUID (User who made the change)
- ip_address: INET (Source IP address)
- user_agent: TEXT (Browser/app information)
- session_id: VARCHAR(255) (Session identifier)
- timestamp: TIMESTAMP NOT NULL DEFAULT NOW()
- business_context: JSONB (Business operation context)
```

### 4.2 Data Archival Strategy
```sql
Purpose: Long-term data retention and performance optimization
Key Features:
• Automated data lifecycle management
• Historical data preservation
• Performance optimization through partitioning
• Compliance-driven retention policies

Archival Rules:
- Visitor records: Archive after 2 years, delete after 7 years
- Delivery records: Archive after 1 year, delete after 5 years
- Attendance records: Archive after 3 years, delete after 10 years
- Audit logs: Archive after 1 year, retain for 10 years
- Financial records: Archive after 3 years, retain permanently
- Communication logs: Archive after 6 months, delete after 3 years

Partitioning Strategy:
- Monthly partitions for high-volume tables (attendance, logs)
- Quarterly partitions for medium-volume tables (deliveries, visitors)
- Annual partitions for low-volume tables (financial records)
```

### 4.3 Data Privacy & Protection
```sql
Purpose: Personal data protection and privacy compliance
Key Features:
• Field-level encryption for sensitive data
• Data anonymization for analytics
• Consent management and tracking
• Right to be forgotten implementation

Encrypted Fields:
- Social security numbers and government IDs
- Bank account and payment information
- Biometric data and health information
- Emergency contact details
- Child information and family details

Data Anonymization:
- Replace names with anonymized identifiers
- Mask phone numbers and email addresses
- Remove geographic precision beyond building level
- Aggregate demographic data for reporting
```

This completes Part 2 of the database design and data models specifications.
