# Oryne — Architecture & Technical Design (Complete)

**Date:** 2025-11-11  
**Author:** AI Assistant (for IIT DEVELOPER)  
**Version:** 1.0  
**Status:** ✅ Finalized for Development

---

## Table of Contents
1. [Executive Summary](#1-executive-summary)
2. [High-Level Architecture](#2-high-level-architecture)
3. [Technology Stack](#3-technology-stack)
4. [Multi-Tenancy Architecture](#4-multi-tenancy-architecture)
5. [Backend Architecture](#5-backend-architecture)
6. [Database Design](#6-database-design)
7. [API Architecture](#7-api-architecture)
8. [Authentication & Authorization](#8-authentication--authorization)
9. [Notification System (GNS Integration)](#9-notification-system-gns-integration)
10. [Asynchronous Processing](#10-asynchronous-processing)
11. [Search & Analytics](#11-search--analytics)
12. [Storage Architecture](#12-storage-architecture)
13. [Deployment Architecture](#13-deployment-architecture)
14. [Security Architecture](#14-security-architecture)
15. [Scalability & Performance](#15-scalability--performance)
16. [Monitoring & Observability](#16-monitoring--observability)
17. [Disaster Recovery & Backup](#17-disaster-recovery--backup)

---

## 1. Executive Summary

Oryne is a comprehensive, AI-powered SaaS platform designed for educational institutions. The architecture is built on modern cloud-native principles with the following key decisions:

### Core Architecture Decisions
✅ **Database:** PostgreSQL with schema-per-tenant multi-tenancy  
✅ **Backend Framework:** Django + Django REST Framework (core services) + FastAPI (AI/ML microservices)  
✅ **Notifications:** Integrated with existing GNS (General Notification Service) microservice  
✅ **Timeline Granularity:** Detailed week-by-week implementation phases  
✅ **Cloud Native:** Containerized deployment with Kubernetes orchestration  
✅ **Security First:** JWT authentication, RBAC, data encryption, audit logging

### Key Architectural Principles
- **Scalability:** Horizontal scaling for all components
- **Tenant Isolation:** Schema-per-tenant for data security and compliance
- **Modularity:** Loosely coupled services with clear boundaries
- **Reliability:** High availability with redundancy and failover
- **Performance:** Caching, async processing, and optimized queries
- **Maintainability:** Clean code, comprehensive testing, documentation

---

## 2. High-Level Architecture

### 2.1 System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Client Layer                                 │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────────┐ │
│  │  Web App   │  │ Mobile App │  │  Admin UI  │  │ Partner APIs │ │
│  │ (Next.js)  │  │ (Flutter)  │  │ (React)    │  │  (REST)      │ │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └──────┬───────┘ │
└────────┼───────────────┼───────────────┼─────────────────┼─────────┘
         │               │               │                 │
         └───────────────┴───────────────┴─────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │    CDN / WAF / TLS      │
                    │   (CloudFlare/AWS)      │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │     API Gateway         │
                    │   (Nginx/Traefik)       │
                    │  - Rate Limiting        │
                    │  - Load Balancing       │
                    │  - Routing              │
                    └──┬──────────────────┬───┘
                       │                  │
         ┌─────────────▼─────┐    ┌──────▼──────────┐
         │   Django Backend  │    │ FastAPI Services│
         │   (Core Services) │    │  (AI/ML/Async)  │
         │                   │    │                  │
         │ ┌───────────────┐ │    │ ┌──────────────┐│
         │ │ Auth Service  │ │    │ │ Recommend.   ││
         │ │ ERP Modules   │ │    │ │ Plagiarism   ││
         │ │ LMS Modules   │ │    │ │ Analytics    ││
         │ │ E-Store       │ │    │ │ Predictions  ││
         │ │ Hostel Mgmt   │ │    │ └──────────────┘│
         │ │ Library       │ │    └──────┬──────────┘
         │ │ Transport     │ │           │
         │ │ Events        │ │           │
         │ └───────────────┘ │           │
         └─────────┬─────────┘           │
                   │                     │
         ┌─────────▼─────────────────────▼───────┐
         │        Message Broker Layer           │
         │   ┌──────────┐      ┌──────────┐     │
         │   │RabbitMQ  │      │  Redis   │     │
         │   │(Celery)  │      │ (Cache)  │     │
         │   └────┬─────┘      └────┬─────┘     │
         └────────┼──────────────────┼───────────┘
                  │                  │
         ┌────────▼──────────────────▼───────────┐
         │       Background Workers              │
         │   ┌──────────────────────────┐       │
         │   │   Celery Workers         │       │
         │   │  - Email/SMS Tasks       │       │
         │   │  - Report Generation     │       │
         │   │  - Data Sync             │       │
         │   │  - ML Pipeline           │       │
         │   │  - Search Indexing       │       │
         │   └──────────┬───────────────┘       │
         └──────────────┼───────────────────────┘
                        │
         ┌──────────────▼───────────────────────┐
         │         Data Layer                   │
         │  ┌──────────┐  ┌────────┐  ┌──────┐│
         │  │PostgreSQL│  │Elastic-│  │  S3  ││
         │  │(Schemas) │  │ search │  │Media ││
         │  └──────────┘  └────────┘  └──────┘│
         └──────────────────────────────────────┘
                        │
         ┌──────────────▼───────────────────────┐
         │      External Services               │
         │  ┌─────────┐  ┌──────────┐          │
         │  │   GNS   │  │ Payment  │          │
         │  │(Notify) │  │ Gateway  │          │
         │  └─────────┘  └──────────┘          │
         └──────────────────────────────────────┘

         ┌──────────────────────────────────────┐
         │    Observability & Monitoring        │
         │  ┌──────────┐  ┌──────────┐         │
         │  │Prometheus│  │ Grafana  │         │
         │  │   ELK    │  │  Sentry  │         │
         │  └──────────┘  └──────────┘         │
         └──────────────────────────────────────┘
```

### 2.2 Component Interaction Flow

**Standard API Request Flow:**
```
User → CDN/WAF → API Gateway → Django/FastAPI → Database → Response
                      ↓
                  Rate Limit
                      ↓
                 Auth Check
                      ↓
                Tenant Routing
```

**Async Task Flow:**
```
API Request → Enqueue Task → RabbitMQ → Celery Worker → Execute
                                             ↓
                                      GNS/Database/S3
                                             ↓
                                     Update Status/Notify
```

---

## 3. Technology Stack

### 3.1 Backend Technologies

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Core Framework** | Django | 5.0+ | Main application framework |
| **API Framework** | Django REST Framework | 3.14+ | RESTful API development |
| **Async API** | FastAPI | 0.104+ | AI/ML microservices |
| **Database** | PostgreSQL | 15+ | Primary data store |
| **Cache** | Redis | 7.0+ | Caching & sessions |
| **Message Broker** | RabbitMQ | 3.12+ | Task queue |
| **Task Queue** | Celery | 5.3+ | Async task processing |
| **Search Engine** | Elasticsearch | 8.10+ | Full-text search |
| **Object Storage** | AWS S3 / MinIO | Latest | Media files |

### 3.2 Frontend Technologies

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | Next.js | 14+ | React framework with SSR |
| **UI Library** | React | 18+ | Component library |
| **Styling** | Tailwind CSS | 3.4+ | Utility-first CSS |
| **State Management** | React Query | 5.0+ | Server state |
| **State Management** | Zustand | 4.4+ | Client state |
| **Forms** | React Hook Form | 7.48+ | Form management |
| **Charts** | Recharts/Chart.js | Latest | Data visualization |

### 3.3 DevOps & Infrastructure

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Container** | Docker | Containerization |
| **Orchestration** | Kubernetes (K8s) | Container orchestration |
| **CI/CD** | GitHub Actions | Automation pipeline |
| **IaC** | Terraform | Infrastructure as code |
| **Monitoring** | Prometheus + Grafana | Metrics & dashboards |
| **Logging** | ELK Stack / CloudWatch | Centralized logging |
| **APM** | Sentry / New Relic | Error tracking & APM |
| **Cloud** | AWS / GCP / Azure | Cloud provider |

---

## 4. Multi-Tenancy Architecture

### 4.1 Schema-Per-Tenant Strategy

**Why Schema-Per-Tenant?**
- ✅ Strong data isolation (compliance & security)
- ✅ Per-tenant backup and restore
- ✅ Easier to scale individual tenants
- ✅ Custom schema modifications per tenant
- ✅ Better query performance (no tenant_id filtering)

**Implementation with Django:**
```python
# Using django-tenants or custom middleware

# Database structure:
# - public schema: tenant registry, platform users, billing
# - tenant schemas: tenant_1, tenant_2, ... tenant_N
```

### 4.2 Tenant Identification

**Primary Method: Subdomain Routing**
```
tenant1.oryne.com → routes to tenant_1 schema
tenant2.oryne.com → routes to tenant_2 schema
```

**Fallback Method: Header-Based**
```
X-Tenant-ID: tenant_1
```

### 4.3 Tenant Lifecycle

```
┌──────────────┐
│  Onboarding  │ → Platform Admin creates tenant
└──────┬───────┘
       │
┌──────▼───────┐
│ Provisioning │ → Create schema, run migrations, seed data
└──────┬───────┘
       │
┌──────▼───────┐
│    Active    │ → Normal operations
└──────┬───────┘
       │
┌──────▼───────┐
│  Suspended   │ → Payment issues / policy violations
└──────┬───────┘
       │
┌──────▼───────┐
│   Archived   │ → Data retention, eventual deletion
└──────────────┘
```

### 4.4 Tenant Provisioning Process

**Automated Provisioning Flow:**
1. Platform admin creates tenant via API
2. Record created in `public.tenants` table with status='provisioning'
3. Celery task `provision_tenant(tenant_id)` triggered
4. Worker executes:
   - Create PostgreSQL schema `tenant_<id>`
   - Run Django migrations on tenant schema
   - Create S3 bucket prefix `tenants/tenant_<id>/`
   - Create Elasticsearch indices `oryne_tenant_<id>_*`
   - Seed default data (roles, permissions, settings)
   - Create admin user and send invite via GNS
   - Update status='active'
5. Tenant receives welcome email with login credentials

**Rollback Strategy:**
- If provisioning fails, mark tenant as 'failed'
- Keep schema for investigation
- Manual cleanup or automated retry

---

## 5. Backend Architecture

### 5.1 Django Application Structure

```
/backend
├── /core                    # Shared utilities
│   ├── middleware.py        # Tenant routing, auth
│   ├── models.py           # Abstract base models
│   ├── permissions.py      # RBAC permissions
│   └── utils.py            # Helper functions
│
├── /tenants                # Tenant management
│   ├── models.py          # Tenant, Domain models
│   ├── tasks.py           # Provisioning tasks
│   └── api/               # Tenant CRUD APIs
│
├── /auth                   # Authentication
│   ├── models.py          # User, Role, Permission
│   ├── serializers.py
│   ├── views.py
│   └── jwt.py             # JWT token handling
│
├── /erp                    # ERP modules
│   ├── /students
│   ├── /teachers
│   ├── /admissions
│   ├── /fees
│   ├── /attendance
│   └── /timetable
│
├── /lms                    # Learning Management
│   ├── /courses
│   ├── /assignments
│   ├── /submissions
│   ├── /grades
│   └── /live_classes
│
├── /estore                 # E-commerce
│   ├── /products
│   ├── /orders
│   ├── /cart
│   └── /payments
│
├── /hostel                 # Hostel Management
├── /library                # Library System
├── /transport              # Transport & Logistics
├── /events                 # Events & Activities
├── /notifications          # GNS Integration
│
└── /config
    ├── settings.py
    ├── urls.py
    └── celery.py
```

### 5.2 FastAPI Microservices Structure

```
/ai_services
├── /recommendation_engine
│   ├── main.py
│   ├── models.py          # ML models
│   ├── schemas.py         # Pydantic models
│   └── routes.py
│
├── /plagiarism_detection
│   ├── main.py
│   ├── nlp_pipeline.py
│   └── routes.py
│
├── /analytics_engine
│   ├── main.py
│   ├── predictions.py
│   └── reports.py
│
└── /shared
    ├── auth.py            # JWT verification
    ├── database.py
    └── utils.py
```

---

## 6. Database Design

### 6.1 PostgreSQL Architecture

**Public Schema (Shared Data):**
```sql
-- Tenant Registry
CREATE TABLE tenants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    subdomain VARCHAR(100) UNIQUE NOT NULL,
    status VARCHAR(20) DEFAULT 'provisioning',
    plan VARCHAR(50),
    paid_until TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Platform Users (Super Admins)
CREATE TABLE platform_users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    roles JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Billing
CREATE TABLE billing_invoices (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER REFERENCES tenants(id),
    amount DECIMAL(10,2),
    status VARCHAR(20),
    paid_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Tenant Schema (Per-Tenant Data):**
```sql
-- Users (Tenant-specific)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    role VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    profile_data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Students
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    enrollment_no VARCHAR(50) UNIQUE NOT NULL,
    class VARCHAR(20),
    section VARCHAR(10),
    dob DATE,
    parent_contact VARCHAR(20),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Courses
CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    code VARCHAR(20) UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    teacher_id INTEGER REFERENCES users(id),
    credits INTEGER,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- And similar tables for: teachers, assignments, submissions,
-- fees, products, orders, hostel_rooms, library_books, etc.
```

### 6.2 Indexing Strategy

```sql
-- Performance Indexes
CREATE INDEX idx_students_enrollment ON students(enrollment_no);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_courses_code ON courses(code);
CREATE INDEX idx_submissions_student ON submissions(student_id, assignment_id);

-- JSONB Indexes (for flexible queries)
CREATE INDEX idx_students_metadata ON students USING GIN(metadata);
CREATE INDEX idx_courses_metadata ON courses USING GIN(metadata);

-- Partial Indexes (for common filters)
CREATE INDEX idx_active_users ON users(id) WHERE is_active = TRUE;
```

### 6.3 Connection Pooling

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'OPTIONS': {
            'options': '-c search_path=public',
        },
        'CONN_MAX_AGE': 600,  # Connection pooling
        'CONN_HEALTH_CHECKS': True,
    }
}

# Use PgBouncer for connection pooling at infrastructure level
```

---

## 7. API Architecture

### 7.1 API Design Principles

- **RESTful conventions**: Resource-based URLs, HTTP methods
- **Versioning**: `/api/v1/`, `/api/v2/` for backward compatibility
- **Consistent response format**: Envelope pattern
- **Pagination**: Cursor-based or offset-limit
- **Filtering & Sorting**: Query parameters
- **Error handling**: Standard error codes and messages

### 7.2 API Response Envelope

```json
{
  "success": true,
  "data": {
    // Response payload
  },
  "meta": {
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 100,
      "total_pages": 5
    },
    "timestamp": "2025-11-11T10:30:00Z"
  },
  "error": null
}
```

**Error Response:**
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERR_AUTH_01",
    "message": "Invalid credentials",
    "details": {
      "field": "password",
      "reason": "incorrect"
    }
  }
}
```

### 7.3 API Rate Limiting

```python
# Django settings
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',
        'user': '1000/hour',
        'tenant_admin': '10000/hour'
    }
}
```

---

## 8. Authentication & Authorization

### 8.1 JWT Token Structure

**Access Token (short-lived, 15 minutes):**
```json
{
  "user_id": 123,
  "email": "teacher@school.com",
  "tenant_id": 45,
  "roles": ["Teacher"],
  "permissions": ["courses:read", "assignments:write"],
  "exp": 1699776000,
  "iat": 1699775100
}
```

**Refresh Token (long-lived, 7 days):**
- Stored in httpOnly cookie or secure storage
- Used to obtain new access tokens
- Blacklisted on logout

### 8.2 Role-Based Access Control (RBAC)

**Roles:**
- `PlatformAdmin`: Full system access
- `TenantAdmin`: Full tenant access
- `Teacher`: Course management, grading
- `Student`: Course access, submissions
- `Parent`: View child's progress
- `Staff`: Administrative tasks

**Permission Model:**
```python
# Example permission checks
@permission_required('courses.add_course')
def create_course(request):
    pass

# Tenant-scoped permissions
@tenant_permission_required('students.view_student')
def list_students(request):
    pass
```

---

## 9. Notification System (GNS Integration)

### 9.1 GNS Architecture

**GNS (General Notification Service)** is an existing microservice that handles all notifications:
- Email notifications
- SMS notifications
- Push notifications (mobile/web)
- In-app notifications

**Integration Pattern:**
```
Django Backend → Celery Task → GNS API/Queue → User
```

### 9.2 GNS Integration Flow

```python
# Example: Send notification via GNS
from celery import shared_task
import requests

@shared_task
def send_notification_via_gns(notification_data):
    """
    Send notification through GNS service
    """
    gns_endpoint = settings.GNS_API_URL
    payload = {
        "tenant_id": notification_data['tenant_id'],
        "user_id": notification_data['user_id'],
        "channel": notification_data['channel'],  # email, sms, push
        "template": notification_data['template'],
        "data": notification_data['data'],
        "priority": notification_data.get('priority', 'normal')
    }
    
    response = requests.post(
        f"{gns_endpoint}/api/notifications/send",
        json=payload,
        headers={"Authorization": f"Bearer {settings.GNS_API_KEY}"}
    )
    
    return response.json()
```

### 9.3 Notification Templates in GNS

**Common Templates:**
- `welcome_email`: New user onboarding
- `assignment_due`: Assignment deadline reminder
- `grade_published`: Grades available
- `fee_reminder`: Fee payment reminder
- `event_registration`: Event confirmation
- `password_reset`: Password reset link

**Template Variables:**
```json
{
  "template": "assignment_due",
  "data": {
    "student_name": "John Doe",
    "assignment_title": "Math Homework",
    "due_date": "2025-11-15",
    "course_name": "Mathematics 101"
  }
}
```

### 9.4 Notification Preferences

Users can configure notification preferences:
```python
# User notification settings
{
  "email": {
    "assignment_due": true,
    "grade_published": true,
    "announcements": false
  },
  "sms": {
    "emergency_alerts": true,
    "fee_reminders": true
  },
  "push": {
    "real_time_updates": true
  }
}
```

---

## 10. Asynchronous Processing

### 10.1 Celery Task Architecture

**Task Categories:**
1. **Notification Tasks**: Email, SMS via GNS
2. **Report Generation**: PDF reports, exports
3. **Data Processing**: Bulk imports, calculations
4. **ML Pipeline**: Trigger AI/ML jobs
5. **Search Indexing**: Elasticsearch sync
6. **Scheduled Tasks**: Cron-like periodic tasks

### 10.2 Celery Configuration

```python
# celery.py
from celery import Celery
from celery.schedules import crontab

app = Celery('oryne')
app.config_from_object('django.conf:settings', namespace='CELERY')

# Task routing
app.conf.task_routes = {
    'notifications.*': {'queue': 'notifications'},
    'reports.*': {'queue': 'reports'},
    'ml.*': {'queue': 'ml'},
}

# Periodic tasks
app.conf.beat_schedule = {
    'send-daily-reports': {
        'task': 'reports.tasks.generate_daily_reports',
        'schedule': crontab(hour=6, minute=0),
    },
    'sync-elasticsearch': {
        'task': 'search.tasks.sync_indices',
        'schedule': crontab(minute='*/30'),
    },
}
```

### 10.3 Task Priority & Retry

```python
@shared_task(bind=True, max_retries=3, default_retry_delay=60)
def process_bulk_import(self, file_path, tenant_id):
    try:
        # Process import
        pass
    except Exception as exc:
        raise self.retry(exc=exc, countdown=60)

# High priority task
@shared_task(priority=9)
def send_emergency_notification(message):
    pass
```

---

## 11. Search & Analytics

### 11.1 Elasticsearch Architecture

**Index Strategy:**
- Per-tenant indices: `oryne_tenant_<id>_courses`, `oryne_tenant_<id>_students`
- Alias for easy rotation: `oryne_tenant_<id>_courses_v1`

**Index Mapping Example:**
```json
{
  "mappings": {
    "properties": {
      "title": { "type": "text", "analyzer": "standard" },
      "description": { "type": "text" },
      "code": { "type": "keyword" },
      "created_at": { "type": "date" },
      "metadata": { "type": "object", "enabled": true }
    }
  }
}
```

### 11.2 Search Synchronization

**Real-time Sync:**
```python
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=Course)
def sync_course_to_elasticsearch(sender, instance, **kwargs):
    # Async task to index
    index_document.delay('courses', instance.id)
```

**Bulk Sync:**
```python
@shared_task
def bulk_index_courses(tenant_id):
    courses = Course.objects.filter(tenant_id=tenant_id)
    bulk_data = []
    for course in courses:
        bulk_data.append({
            '_index': f'oryne_tenant_{tenant_id}_courses',
            '_id': course.id,
            '_source': course.to_dict()
        })
    helpers.bulk(es_client, bulk_data)
```

---

## 12. Storage Architecture

### 12.1 S3 Bucket Structure

```
oryne-media/
├── tenants/
│   ├── tenant_1/
│   │   ├── avatars/
│   │   ├── assignments/
│   │   ├── submissions/
│   │   ├── library/
│   │   └── reports/
│   ├── tenant_2/
│   │   └── ...
│   └── ...
└── public/
    ├── templates/
    └── assets/
```

### 12.2 File Upload Flow

```
Client → Presigned URL Request → Django API
                                     ↓
                            Generate S3 Presigned URL
                                     ↓
Client ← Presigned URL ← Django API
   ↓
Direct Upload to S3
   ↓
Upload Complete → Notify Django API
                        ↓
                   Update Database
```

### 12.3 CDN Integration

- Use CloudFront or CloudFlare for S3 content delivery
- Cache static assets and frequently accessed files
- Implement signed URLs for private content

---

## 13. Deployment Architecture

### 13.1 Kubernetes Deployment

**Namespace Structure:**
```
- oryne-prod
  ├── django-deployment
  ├── fastapi-deployment
  ├── celery-worker-deployment
  ├── celery-beat-deployment
  ├── nginx-ingress
  ├── postgres-statefulset
  ├── redis-statefulset
  └── rabbitmq-statefulset
```

### 13.2 Deployment Diagram

```
┌─────────────────────────────────────────────────┐
│              Kubernetes Cluster                  │
│                                                  │
│  ┌─────────────────────────────────────────┐   │
│  │         Ingress Controller              │   │
│  │         (Nginx/Traefik)                 │   │
│  └──────────────┬──────────────────────────┘   │
│                 │                                │
│  ┌──────────────▼──────────┐ ┌──────────────┐  │
│  │   Django Pods (3x)      │ │ FastAPI (2x) │  │
│  │   - Auto-scaling        │ │ - GPU nodes  │  │
│  │   - HPA enabled         │ │ - ML models  │  │
│  └──────────────┬──────────┘ └──────┬───────┘  │
│                 │                     │          │
│  ┌──────────────▼─────────────────────▼──────┐ │
│  │        Service Mesh (Istio)               │ │
│  └──────────────┬────────────────────────────┘ │
│                 │                                │
│  ┌──────────────▼──────────┐ ┌──────────────┐  │
│  │  Celery Workers (5x)    │ │  Beat (1x)   │  │
│  └──────────────┬──────────┘ └──────┬───────┘  │
│                 │                     │          │
│  ┌──────────────▼─────────────────────▼──────┐ │
│  │           StatefulSets                    │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ │ │
│  │  │PostgreSQL│ │  Redis   │ │ RabbitMQ │ │ │
│  │  │  (3x)    │ │  (3x)    │ │  (3x)    │ │ │
│  │  └──────────┘ └──────────┘ └──────────┘ │ │
│  └───────────────────────────────────────────┘ │
│                                                  │
│  ┌───────────────────────────────────────────┐ │
│  │        Persistent Volumes (EBS/PD)        │ │
│  └───────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

### 13.3 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy Oryne
on:
  push:
    branches: [main, staging]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Tests
        run: |
          python -m pytest tests/
          
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker Images
        run: |
          docker build -t oryne/django:${{ github.sha }} .
          docker push oryne/django:${{ github.sha }}
          
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/django \
            django=oryne/django:${{ github.sha }}
          kubectl rollout status deployment/django
```

---

## 14. Security Architecture

### 14.1 Security Layers

```
┌─────────────────────────────────────┐
│   Layer 1: Network Security         │
│   - WAF (CloudFlare/AWS WAF)        │
│   - DDoS Protection                 │
│   - IP Whitelisting                 │
└─────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────┐
│   Layer 2: Application Security     │
│   - HTTPS/TLS 1.3                   │
│   - CORS Policy                     │
│   - CSP Headers                     │
│   - Rate Limiting                   │
└─────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────┐
│   Layer 3: Authentication           │
│   - JWT Tokens                      │
│   - MFA (Optional)                  │
│   - Session Management              │
└─────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────┐
│   Layer 4: Authorization            │
│   - RBAC                            │
│   - Tenant Isolation                │
│   - Permission Checks               │
└─────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────┐
│   Layer 5: Data Security            │
│   - Encryption at Rest              │
│   - Encryption in Transit           │
│   - Data Masking                    │
│   - Audit Logging                   │
└─────────────────────────────────────┘
```

### 14.2 Data Encryption

**At Rest:**
- PostgreSQL: Transparent Data Encryption (TDE)
- S3: Server-side encryption (SSE-S3 or SSE-KMS)
- Redis: Encrypted snapshots

**In Transit:**
- TLS 1.3 for all API communication
- SSL connections to databases
- VPN for internal service communication

### 14.3 Audit Logging

```python
# Audit log model
class AuditLog(models.Model):
    tenant_id = models.IntegerField()
    user_id = models.IntegerField()
    action = models.CharField(max_length=50)  # CREATE, UPDATE, DELETE
    resource_type = models.CharField(max_length=50)
    resource_id = models.IntegerField()
    changes = models.JSONField()
    ip_address = models.GenericIPAddressField()
    user_agent = models.TextField()
    timestamp = models.DateTimeField(auto_now_add=True)
```

---

## 15. Scalability & Performance

### 15.1 Horizontal Scaling Strategy

**Auto-scaling Rules:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: django-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: django
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 15.2 Caching Strategy

**Multi-level Caching:**
```
┌──────────────────────────────┐
│   Level 1: CDN Cache         │ ← Static assets (24h)
└──────────────────────────────┘
              ↓
┌──────────────────────────────┐
│   Level 2: Redis Cache       │ ← API responses (5-60min)
└──────────────────────────────┘
              ↓
┌──────────────────────────────┐
│   Level 3: Database Query    │ ← Postgres query cache
└──────────────────────────────┘
```

**Cache Invalidation:**
```python
# Invalidate on update
@cache_invalidate('course:detail:{id}')
def update_course(course_id, data):
    pass

# Cache with TTL
@cache_with_ttl(300)  # 5 minutes
def get_dashboard_stats(tenant_id):
    pass
```

### 15.3 Database Optimization

**Read Replicas:**
```
Master (Write) ─┬─→ Replica 1 (Read)
                ├─→ Replica 2 (Read)
                └─→ Replica 3 (Read)
```

**Query Optimization:**
- Use `select_related()` and `prefetch_related()` to avoid N+1 queries
- Implement database indexes on frequently queried fields
- Use materialized views for complex aggregations
- Partition large tables by date/tenant

---

## 16. Monitoring & Observability

### 16.1 Monitoring Stack

```
┌─────────────────────────────────────┐
│         Application Layer           │
│  - Django (metrics endpoint)        │
│  - FastAPI (metrics endpoint)       │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────▼───────────────────┐
│       Prometheus (Metrics)          │
│  - Scrape metrics every 15s         │
│  - PromQL queries                   │
│  - Alerting rules                   │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────▼───────────────────┐
│        Grafana (Visualization)      │
│  - Dashboards                       │
│  - Alerts                           │
│  - Reports                          │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│       ELK Stack (Logs)              │
│  - Elasticsearch (storage)          │
│  - Logstash (processing)            │
│  - Kibana (visualization)           │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│       Sentry (Error Tracking)       │
│  - Exception capture                │
│  - Stack traces                     │
│  - User context                     │
└─────────────────────────────────────┘
```

### 16.2 Key Metrics to Monitor

**Application Metrics:**
- Request rate (requests/second)
- Response time (p50, p95, p99)
- Error rate (4xx, 5xx)
- Active users
- API endpoint performance

**Infrastructure Metrics:**
- CPU usage
- Memory usage
- Disk I/O
- Network traffic
- Pod restarts

**Business Metrics:**
- Active tenants
- User signups
- Feature usage
- Revenue metrics

### 16.3 Alerting Rules

```yaml
# Prometheus alerts
groups:
  - name: oryne_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 2
        for: 5m
        labels:
          severity: warning
```

---

## 17. Disaster Recovery & Backup

### 17.1 Backup Strategy

**PostgreSQL Backups:**
- **Continuous archiving**: WAL archiving to S3
- **Daily snapshots**: Full database backup at 2 AM UTC
- **Retention**: 30 days daily, 12 months monthly
- **Per-tenant restore**: Schema-level backup/restore

**Redis Backups:**
- RDB snapshots every 6 hours
- AOF persistence for durability

**S3 Backups:**
- Versioning enabled
- Cross-region replication
- Lifecycle policies for archival

### 17.2 Disaster Recovery Plan

**RTO (Recovery Time Objective):** 4 hours  
**RPO (Recovery Point Objective):** 15 minutes

**Recovery Procedures:**
1. **Database Failure:**
   - Promote read replica to master
   - Update application config
   - Restore from WAL if needed

2. **Complete Region Failure:**
   - Failover to secondary region
   - DNS update (Route53/CloudFlare)
   - Restore from cross-region backups

3. **Tenant Data Corruption:**
   - Restore specific tenant schema from backup
   - Point-in-time recovery if needed

### 17.3 Business Continuity

**High Availability:**
- Multi-AZ deployment
- Load balancing across zones
- Database replication
- Redis clustering

**Maintenance Windows:**
- Scheduled: Sundays 2-4 AM UTC
- Zero-downtime deployments with rolling updates
- Blue-green deployment for major updates

---

## Appendix A: Technology Decision Matrix

| Requirement | Options Considered | Selected | Rationale |
|-------------|-------------------|----------|-----------|
| Database | PostgreSQL, MySQL, MongoDB | **PostgreSQL** | Best schema-per-tenant support, JSONB, performance |
| API Framework | Django Rest, FastAPI, Flask | **Django REST + FastAPI** | DRF for CRUD, FastAPI for async/ML |
| Cache | Redis, Memcached | **Redis** | Pub/sub, data structures, persistence |
| Message Broker | RabbitMQ, Redis, Kafka | **RabbitMQ** | Reliable, mature, good for Celery |
| Search | Elasticsearch, Solr, Algolia | **Elasticsearch** | Full-text search, analytics, scalable |
| Container Orchestration | Kubernetes, Docker Swarm, ECS | **Kubernetes** | Industry standard, ecosystem, portability |

---

## Appendix B: Glossary

- **GNS**: General Notification Service - existing microservice for notifications
- **DRF**: Django REST Framework
- **RBAC**: Role-Based Access Control
- **WAF**: Web Application Firewall
- **JWT**: JSON Web Token
- **HPA**: Horizontal Pod Autoscaler
- **TLS**: Transport Layer Security
- **RTO**: Recovery Time Objective
- **RPO**: Recovery Point Objective
- **CDN**: Content Delivery Network

---

**Document Status:** ✅ Complete and Ready for Development  
**Last Updated:** 2025-11-11  
**Next Review:** Upon Phase completion

---
