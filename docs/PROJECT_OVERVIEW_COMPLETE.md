# Oryne — Complete Project Overview & Getting Started Guide

**Version:** 1.0  
**Date:** 2025-11-11  
**Status:** ✅ Ready for Development

---

## 📋 Table of Contents
1. [Project Summary](#project-summary)
2. [Key Decisions & Tech Stack](#key-decisions--tech-stack)
3. [Architecture Overview](#architecture-overview)
4. [Documentation Index](#documentation-index)
5. [Getting Started](#getting-started)
6. [Development Roadmap](#development-roadmap)
7. [Team & Responsibilities](#team--responsibilities)

---

## 📊 Project Summary

### What is Oryne?
**Oryne** (formerly EduTenant) is an all-in-one AI-powered SaaS platform designed for educational institutions. It combines ERP, LMS, E-Store, and AI capabilities into a single, unified system.

### Mission
To transform educational management by providing institutions with a comprehensive, AI-driven platform that simplifies operations, enhances teaching, and enables data-driven decision-making.

### Target Market
- **Primary:** K-12 schools, colleges, universities
- **Secondary:** Training centers, coaching institutes
- **Geography:** Initially India, expanding globally

### Business Model
- **SaaS Subscription:** Monthly/Annual plans
- **Tiers:** Trial, Basic, Standard, Premium, Enterprise
- **Pricing:** Based on users, features, storage

---

## ✅ Key Decisions & Tech Stack

### Core Technology Decisions

| Component | Technology | Decision Rationale |
|-----------|-----------|-------------------|
| **Database** | **PostgreSQL 15+** | ✅ Best schema-per-tenant support<br>✅ JSONB for flexible data<br>✅ Strong ACID compliance<br>✅ Excellent performance |
| **Backend Framework** | **Django 5.0+ with DRF** | ✅ Rapid development<br>✅ Robust ORM<br>✅ Built-in admin<br>✅ Strong security |
| **API Layer** | **Django REST Framework** | ✅ Powerful serialization<br>✅ Auto-generated OpenAPI docs<br>✅ Authentication built-in<br>✅ Flexible permissions |
| **AI/ML Services** | **FastAPI 0.104+** | ✅ Async support<br>✅ Fast performance<br>✅ Type hints<br>✅ Auto docs |
| **Multi-Tenancy** | **Schema-per-Tenant** | ✅ Strong data isolation<br>✅ Per-tenant backup/restore<br>✅ Easier compliance<br>✅ Performance benefits |
| **Notifications** | **GNS Integration** | ✅ Existing microservice<br>✅ Email, SMS, Push unified<br>✅ Template management<br>✅ Delivery tracking |
| **Task Queue** | **Celery 5.3+ with RabbitMQ** | ✅ Reliable task execution<br>✅ Priority queues<br>✅ Scheduled tasks<br>✅ Retry mechanisms |
| **Cache** | **Redis 7.0+** | ✅ Fast in-memory storage<br>✅ Pub/sub support<br>✅ Session storage<br>✅ Rate limiting |
| **Search** | **Elasticsearch 8.10+** | ✅ Full-text search<br>✅ Analytics capabilities<br>✅ Scalable<br>✅ Rich query DSL |
| **Storage** | **AWS S3 / MinIO** | ✅ Scalable object storage<br>✅ CDN integration<br>✅ Cost-effective<br>✅ Versioning |
| **Frontend** | **Next.js 14+ with React 18** | ✅ SSR/SSG support<br>✅ Great DX<br>✅ Performance<br>✅ SEO friendly |
| **Styling** | **Tailwind CSS 3.4+** | ✅ Utility-first<br>✅ Fast development<br>✅ Consistent design<br>✅ Small bundle size |
| **Container** | **Docker + Kubernetes** | ✅ Consistent environments<br>✅ Easy scaling<br>✅ Cloud agnostic<br>✅ Industry standard |

### Why These Choices?

#### PostgreSQL over MySQL/MongoDB
- Schema-per-tenant is easier in PostgreSQL
- JSONB provides flexibility without sacrificing ACID properties
- Better full-text search capabilities
- PostGIS for potential location-based features

#### Django + FastAPI Hybrid Approach
- Django for CRUD operations (80% of features)
- FastAPI for CPU-intensive AI/ML tasks (20% of features)
- Best of both worlds: rapid development + high performance

#### Schema-per-Tenant over Shared Tables
- **Security:** Complete data isolation
- **Compliance:** GDPR, data residency requirements
- **Performance:** No tenant_id filtering needed
- **Flexibility:** Per-tenant schema customization
- **Backup:** Easy tenant-specific backup/restore

#### GNS for Notifications
- Existing, proven microservice
- Unified interface for all notification types
- Reduces backend complexity
- Template management centralized
- Delivery tracking and analytics

---

## 🏗 Architecture Overview

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     End Users                            │
│  Students | Teachers | Admins | Parents | Platform Admin │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│              Frontend Applications                       │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Web App    │  │  Mobile App  │  │   Admin UI   │ │
│  │  (Next.js)   │  │  (Flutter)   │  │   (React)    │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ HTTPS/TLS
                     │
┌────────────────────▼────────────────────────────────────┐
│                  API Gateway Layer                       │
│             (Nginx / Traefik / Kong)                     │
│  - TLS Termination                                       │
│  - Load Balancing                                        │
│  - Rate Limiting                                         │
│  - Authentication Pre-check                              │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼──────┐         ┌────────▼─────────┐
│   Django     │         │   FastAPI        │
│   Backend    │         │   AI Services    │
│              │         │                  │
│ • REST APIs  │         │ • Recommend.     │
│ • ERP/LMS    │◄────────┤ • Plagiarism     │
│ • E-Store    │ RPC/HTTP│ • Analytics      │
│ • Auth       │         │ • Predictions    │
└───────┬──────┘         └────────┬─────────┘
        │                         │
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │   Message Broker        │
        │                         │
        │  ┌────────┐ ┌────────┐ │
        │  │RabbitMQ│ │ Redis  │ │
        │  │(Tasks) │ │(Cache) │ │
        │  └───┬────┘ └───┬────┘ │
        └──────┼──────────┼───────┘
               │          │
        ┌──────▼──────────▼───────┐
        │   Celery Workers        │
        │  • Notifications        │
        │  • Reports              │
        │  • Data Processing      │
        │  • ML Jobs              │
        └──────┬──────────────────┘
               │
        ┌──────▼──────────────────┐
        │    Data Layer           │
        │                         │
        │  ┌─────────────────┐   │
        │  │  PostgreSQL     │   │
        │  │  Multi-Tenant   │   │
        │  │  Schema-per-    │   │
        │  │  Tenant         │   │
        │  └────────┬────────┘   │
        │           │             │
        │  ┌────────▼────────┐   │
        │  │ Elasticsearch   │   │
        │  │ Search & Analyt.│   │
        │  └────────┬────────┘   │
        │           │             │
        │  ┌────────▼────────┐   │
        │  │  AWS S3/MinIO   │   │
        │  │  Media Storage  │   │
        │  └─────────────────┘   │
        └─────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │  External Services      │
        │                         │
        │  ┌────────┐ ┌────────┐ │
        │  │  GNS   │ │Payment │ │
        │  │Notify  │ │Gateway │ │
        │  └────────┘ └────────┘ │
        └─────────────────────────┘
```

### Multi-Tenancy Architecture

```
┌────────────────────────────────────────────────────┐
│              PostgreSQL Database                    │
│                                                     │
│  ┌───────────────────────────────────────────────┐ │
│  │          PUBLIC SCHEMA                        │ │
│  │                                                │ │
│  │  • tenants (tenant registry)                  │ │
│  │  • platform_users (super admins)              │ │
│  │  • billing_invoices                           │ │
│  │  • subscription_plans                         │ │
│  └───────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ tenant_1     │  │ tenant_2     │  │ tenant_N │ │
│  │ (School A)   │  │ (College B)  │  │ (Inst. C)│ │
│  │              │  │              │  │          │ │
│  │ • users      │  │ • users      │  │ • users  │ │
│  │ • students   │  │ • students   │  │ • ...    │ │
│  │ • teachers   │  │ • teachers   │  │          │ │
│  │ • courses    │  │ • courses    │  │          │ │
│  │ • assignments│  │ • ...        │  │          │ │
│  │ • fees       │  │              │  │          │ │
│  │ • orders     │  │              │  │          │ │
│  │ • ...        │  │              │  │          │ │
│  └──────────────┘  └──────────────┘  └──────────┘ │
└────────────────────────────────────────────────────┘

Request Flow:
1. Request arrives: tenant1.oryne.com/api/students
2. Middleware extracts tenant: "tenant1"
3. Lookup tenant in public.tenants
4. Switch connection to "tenant_1" schema
5. Execute query in tenant schema
6. Return results
7. Reset connection to "public"
```

---

## 📚 Documentation Index

All project documentation is organized in the `/docs` folder:

### Core Architecture Documents

1. **[Oryne_Architecture_Design.md](./Oryne_Architecture_Design.md)**
   - Complete system architecture
   - Technology decisions
   - Multi-tenancy design
   - Security architecture
   - Deployment topology
   - Scalability & performance
   - **Status:** ✅ Complete

2. **[Oryne_Backend_Design_and_API_Spec.md](./Oryne_Backend_Design_and_API_Spec_COMPLETE.md)**
   - Django + FastAPI implementation
   - Database schemas
   - API endpoints (all modules)
   - GNS integration
   - Celery tasks
   - Testing strategy
   - **Status:** ✅ Complete

3. **[Oryne_Frontend_Design_and_Implementation_Plan.md](./Oryne_Frontend_Design_and_Implementation_Plan.md)**
   - Next.js + React architecture
   - Component library
   - Role-based dashboards
   - State management
   - API integration
   - **Status:** 🚧 In Progress

4. **[PROJECT_OVERVIEW_COMPLETE.md](./PROJECT_OVERVIEW_COMPLETE.md)** *(This Document)*
   - Executive summary
   - Tech stack decisions
   - Quick start guide
   - **Status:** ✅ Complete

### Reading Order for New Developers

```
Start Here:
└─> PROJECT_OVERVIEW_COMPLETE.md (This file)
    │
    ├─> Oryne_Architecture_Design.md
    │   └─> Understand system architecture & decisions
    │
    ├─> Oryne_Backend_Design_and_API_Spec.md
    │   └─> Deep dive into backend implementation
    │
    └─> Oryne_Frontend_Design_and_Implementation_Plan.md
        └─> Frontend architecture & components
```

---

## 🚀 Getting Started

### Prerequisites

Before you begin, ensure you have the following installed:

```bash
# Required
- Python 3.11+
- Node.js 18+
- PostgreSQL 15+
- Redis 7+
- Docker & Docker Compose (recommended)

# Optional (for local development)
- RabbitMQ 3.12+
- Elasticsearch 8.10+
- MinIO (S3 compatible storage)
```

### Quick Start (Docker Compose)

**Step 1: Clone the Repository**
```bash
git clone https://github.com/iitdeveloper-git/Oryne.git
cd Oryne
git checkout dev
```

**Step 2: Setup Environment Variables**
```bash
cp .env.example .env
# Edit .env with your configuration
```

**Step 3: Start Services**
```bash
# Start all services
docker-compose up -d

# Services will be available at:
# - Django API: http://localhost:8000
# - FastAPI: http://localhost:8001
# - Frontend: http://localhost:3000
# - PostgreSQL: localhost:5432
# - Redis: localhost:6379
# - RabbitMQ Dashboard: http://localhost:15672
# - Elasticsearch: http://localhost:9200
# - Flower (Celery): http://localhost:5555
```

**Step 4: Initialize Database**
```bash
# Run migrations for public schema
docker-compose exec django python manage.py migrate

# Create superuser
docker-compose exec django python manage.py createsuperuser

# Create test tenant
docker-compose exec django python manage.py create_tenant \
    --name "Test School" \
    --subdomain "testschool" \
    --admin-email "admin@testschool.com"
```

**Step 5: Access the Application**
```bash
# Platform Admin
http://localhost:8000/admin

# Tenant Portal
http://testschool.localhost:3000

# API Documentation
http://localhost:8000/api/docs
http://localhost:8001/docs (FastAPI)
```

### Manual Setup (Without Docker)

<details>
<summary>Click to expand manual setup instructions</summary>

**1. Setup Python Environment**
```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
cd backend
pip install -r requirements/development.txt
```

**2. Setup Database**
```bash
# Create PostgreSQL database
createdb oryne_dev

# Update .env
DATABASE_URL=postgresql://user:password@localhost:5432/oryne_dev

# Run migrations
python manage.py migrate
```

**3. Setup Redis & RabbitMQ**
```bash
# Install Redis (macOS)
brew install redis
brew services start redis

# Install RabbitMQ (macOS)
brew install rabbitmq
brew services start rabbitmq
```

**4. Start Services**
```bash
# Terminal 1: Django
python manage.py runserver

# Terminal 2: Celery Worker
celery -A config worker -l info

# Terminal 3: Celery Beat
celery -A config beat -l info

# Terminal 4: FastAPI
cd ai_services
uvicorn main:app --reload --port 8001

# Terminal 5: Frontend
cd frontend
npm install
npm run dev
```

</details>

### Development Workflow

```bash
# Create a new branch
git checkout -b feature/your-feature-name

# Make changes

# Run tests
pytest

# Run linting
flake8
black .
isort .

# Commit changes
git add .
git commit -m "feat: your feature description"

# Push and create PR
git push origin feature/your-feature-name
```

---

## 🗓 Development Roadmap

### Phase-Wise Implementation (36 weeks total)

#### **Phase 0: Foundation (Weeks 1-2)**
**Goal:** Setup infrastructure and project scaffold

**Week 1:**
- ✅ Repository setup and branching strategy
- ✅ Docker compose for local development
- ✅ CI/CD pipeline (GitHub Actions)
- ✅ Infrastructure as Code (Terraform) for cloud resources

**Week 2:**
- ✅ Base Django project with settings organization
- ✅ PostgreSQL setup with schema-per-tenant
- ✅ Tenant middleware implementation
- ✅ Redis and RabbitMQ integration
- ✅ Basic health check endpoints

**Deliverables:**
- Working local development environment
- CI/CD pipeline
- Infrastructure provisioning scripts
- Basic tenant routing working

---

#### **Phase 1: Authentication & Tenant Management (Weeks 3-5)**
**Goal:** Complete user auth and tenant onboarding

**Week 3: Authentication**
- JWT token implementation
- Login/Logout/Refresh endpoints
- Password reset flow
- Email verification
- RBAC foundation

**Week 4: Tenant Management**
- Tenant CRUD APIs
- Tenant provisioning automation
- Domain management
- Subscription plans model

**Week 5: Admin Portal**
- Platform admin dashboard
- Tenant monitoring
- Billing interface
- Usage analytics

**Deliverables:**
- Complete auth system
- Automated tenant provisioning
- Platform admin portal

---

#### **Phase 2: ERP Core (Weeks 6-9)**
**Goal:** Student, Teacher, and Staff management

**Week 6: Student Management**
- Student CRUD APIs
- Bulk import/export
- Student profiles
- Parent/Guardian management

**Week 7: Teacher & Staff**
- Teacher CRUD APIs
- Staff management
- Department organization
- Employee records

**Week 8: Admissions & Enrollment**
- Admission application flow
- Approval workflow
- Document management
- Enrollment processing

**Week 9: Attendance System**
- Daily attendance marking
- Leave management
- Attendance reports
- Notification integration

**Deliverables:**
- Complete student/teacher management
- Admission system
- Attendance tracking

---

#### **Phase 3: LMS Core (Weeks 10-13)**
**Goal:** Course management and assignments

**Week 10: Course Management**
- Course CRUD APIs
- Course enrollment
- Syllabus management
- Teacher assignment

**Week 11: Assignments**
- Assignment creation
- File uploads (S3)
- Due date management
- Assignment distribution

**Week 12: Submissions & Grading**
- Submission portal
- Grading interface
- Feedback system
- Grade book

**Week 13: Live Classes Integration**
- Zoom/Meet integration
- Class scheduling
- Attendance tracking
- Recording management

**Deliverables:**
- Complete LMS system
- Assignment workflow
- Grading system

---

#### **Phase 4: E-Store, Library, Hostel (Weeks 14-17)**
**Goal:** Supporting modules

**Week 14: E-Store**
- Product catalog
- Shopping cart
- Order management
- Payment gateway integration

**Week 15: Library Management**
- Book catalog
- Loan management
- Reservations
- Digital library

**Week 16: Hostel Management**
- Room allocation
- Day passes
- Mess management
- Hostel fees

**Week 17: Integration & Testing**
- End-to-end testing
- Integration tests
- Bug fixes
- Performance optimization

**Deliverables:**
- E-Store operational
- Library system
- Hostel management

---

#### **Phase 5: Transport, Events, Training (Weeks 18-20)**
**Goal:** Additional institutional modules

**Week 18: Transport & Logistics**
- Route management
- Vehicle tracking
- GPS integration
- Parent notifications

**Week 19: Events & Extracurriculars**
- Event creation
- Registration management
- Attendance tracking
- Certificates generation

**Week 20: Training Portal**
- Training modules
- Professional development
- Certification tracking
- Evaluation system

**Deliverables:**
- Transport system
- Events management
- Training portal

---

#### **Phase 6: AI & Analytics (Weeks 21-24)**
**Goal:** FastAPI microservices for AI/ML

**Week 21: FastAPI Foundation**
- FastAPI project setup
- Database connection
- Authentication with Django
- Base ML pipeline

**Week 22: Recommendation Engine**
- Course recommendations
- Personalized learning paths
- Content suggestions
- Collaborative filtering

**Week 23: Plagiarism Detection**
- Text similarity detection
- Document comparison
- Report generation
- Integration with submissions

**Week 24: Predictive Analytics**
- Student performance prediction
- Dropout risk analysis
- Resource optimization
- Dashboard integration

**Deliverables:**
- AI microservices operational
- Recommendation engine
- Plagiarism detection
- Predictive analytics

---

#### **Phase 7: Frontend Development (Weeks 25-31)**
**Goal:** Complete frontend for all roles

**Week 25-26: Component Library**
- Design system
- Reusable components
- Storybook setup
- Theme configuration

**Week 27-28: Admin & Teacher Portals**
- Dashboard layouts
- Course management UI
- Grading interface
- Reports view

**Week 29-30: Student & Parent Portals**
- Student dashboard
- Course enrollment UI
- Assignment submission
- Parent view

**Week 31: Mobile App Foundation**
- Flutter setup
- API integration
- Core screens
- Push notifications

**Deliverables:**
- Complete web application
- Mobile app foundation
- Responsive design

---

#### **Phase 8: Testing & Hardening (Weeks 32-34)**
**Goal:** Production readiness

**Week 32: Security Audit**
- Penetration testing
- Vulnerability scanning
- OWASP compliance
- Security fixes

**Week 33: Performance Optimization**
- Database optimization
- Query profiling
- Caching strategy
- Load testing

**Week 34: Monitoring & Observability**
- Prometheus metrics
- Grafana dashboards
- ELK stack setup
- Alerting rules

**Deliverables:**
- Security audit report
- Performance benchmarks
- Monitoring dashboards

---

#### **Phase 9: Beta Testing & Launch (Weeks 35-36)**
**Goal:** Production deployment

**Week 35: Beta Testing**
- Pilot with 3-5 institutions
- Feedback collection
- Bug fixes
- Documentation updates

**Week 36: Production Launch**
- Production deployment
- Marketing launch
- Customer onboarding
- Support system activation

**Deliverables:**
- Production system live
- Beta feedback incorporated
- Launch completed

---

### Timeline Summary

```
┌─────────────┬──────────────────────────────────────────┐
│   Phase     │           Duration         │   Status    │
├─────────────┼──────────────────────────────────────────┤
│ Phase 0     │ Weeks 1-2    (2 weeks)    │ 🚧 Current  │
│ Phase 1     │ Weeks 3-5    (3 weeks)    │ ⏳ Planned  │
│ Phase 2     │ Weeks 6-9    (4 weeks)    │ ⏳ Planned  │
│ Phase 3     │ Weeks 10-13  (4 weeks)    │ ⏳ Planned  │
│ Phase 4     │ Weeks 14-17  (4 weeks)    │ ⏳ Planned  │
│ Phase 5     │ Weeks 18-20  (3 weeks)    │ ⏳ Planned  │
│ Phase 6     │ Weeks 21-24  (4 weeks)    │ ⏳ Planned  │
│ Phase 7     │ Weeks 25-31  (7 weeks)    │ ⏳ Planned  │
│ Phase 8     │ Weeks 32-34  (3 weeks)    │ ⏳ Planned  │
│ Phase 9     │ Weeks 35-36  (2 weeks)    │ ⏳ Planned  │
├─────────────┼──────────────────────────────────────────┤
│   Total     │        36 weeks (~9 months)              │
└─────────────┴──────────────────────────────────────────┘
```

---

## 👥 Team & Responsibilities

### Current Team Structure

**Backend Team:**
- Lead Backend Developer
- Django Developer (ERP/LMS)
- FastAPI Developer (AI/ML)
- Database Administrator

**Frontend Team:**
- Lead Frontend Developer
- React Developers (2)
- Mobile Developer (Flutter)

**DevOps Team:**
- DevOps Engineer
- Cloud Architect

**QA Team:**
- QA Lead
- Test Engineers (2)

**Product Team:**
- Product Manager
- UI/UX Designer
- Technical Writer

---

## 📞 Support & Contact

**Development Team:**
- Email: dev@oryne.com
- Slack: #oryne-dev
- GitHub: https://github.com/iitdeveloper-git/Oryne

**Documentation:**
- Technical Docs: `/docs`
- API Docs: `http://localhost:8000/api/docs`
- Wiki: GitHub Wiki

---

## 📄 License

Proprietary - IIT DEVELOPER © 2025

---

**Document Version:** 1.0  
**Last Updated:** 2025-11-11  
**Next Review:** Phase 0 completion

---

## Quick Reference Commands

```bash
# Start development environment
docker-compose up -d

# Run migrations
docker-compose exec django python manage.py migrate

# Create tenant
docker-compose exec django python manage.py create_tenant \
    --name "School Name" --subdomain "schoolname"

# Run tests
docker-compose exec django pytest

# View logs
docker-compose logs -f django

# Stop all services
docker-compose down

# Rebuild after code changes
docker-compose up -d --build
```

---

✅ **Status:** All core documentation complete and ready for development!
