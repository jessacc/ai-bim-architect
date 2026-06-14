# AI BIM Architect Platform - Architecture & System Design

## System Overview

The AI BIM Architect Platform is a cloud-based Building Information Modeling system that generates complete, editable BIM models from multiple input types (natural language, images, PDFs, DWG, IFC) through an orchestrated pipeline of 13 specialized AI models.

### Core Design Principle

**BIM First** → **Everything Else Derived**

- Central BIM Graph = Single Source of Truth
- All outputs (plans, sections, elevations, 3D) automatically derived from BIM
- No disconnected drawings or static images
- Full editability maintained at all times

---

## System Architecture

### High-Level Pipeline

User Input (Natural Language, Images, PDFs, DWG, IFC) 
→ Model 1-2 (Requirements & Reasoning)
→ Model 3-4 (Layout & BIM Generation)
→ Models 5-7 (Elevations, Sections, 3D)
→ Model 8 (Plan Understanding from images)
→ BIM Graph (Central DAG)
→ Models 10-13 (Validation, Quality, BOQ)
→ Professional Outputs (Plans, Schedules, Exports)

---

## Backend Architecture

### Technology Stack

**API Layer:**
- FastAPI (async Python)
- GraphQL for BIM queries
- REST for traditional operations
- WebSocket for real-time collaboration

**Core Services:**
- BIM Engine (Open CASCADE)
- IFC Handler (ifcopenshell)
- AI Model Orchestrator
- Collaboration Manager

**Data Layer:**
- PostgreSQL (project metadata, user data)
- Neo4j (BIM Graph storage)
- Redis (caching, session management)
- MinIO (file storage for uploads)

**Compute:**
- PyTorch + CUDA
- Hugging Face Transformers
- TensorRT for inference optimization
- Kubernetes for scaling

---

## Frontend Architecture

### Workspace Layout

```
Top Menu Bar: File | Edit | View | Tools | Collaborate | Help

Left Panel: Project Explorer
├─ Ground Floor
├─ First Floor
├─ Roof

Main Viewport: [3D View / Plan / Elevation / Section / Schedule]

Right Panel: Properties / AI Assistant
├─ Properties (when element selected)
├─ Materials
├─ Constraints
└─ AI Assistant Chat

Bottom: Status Bar, Timeline, History
```

### Key UI Components

1. **3D BIM Viewer** (Babylon.js)
   - Real-time visualization
   - Interactive selection
   - Exploded views
   - Material preview

2. **2D Floor Plan Editor**
   - Vector-based editing
   - Snap-to-grid
   - Dimension annotations

3. **Properties Panel**
   - Element properties (read/write)
   - Material selector
   - Dimension editor

4. **AI Assistant Chat**
   - Conversational editing
   - Design suggestions
   - Compliance alerts

5. **Collaboration Panel**
   - Active users
   - Comments thread
   - Edit notifications

---

## AI Models Overview

### 13-Model Pipeline

1. **Requirement Understanding** (Qwen 14B) - NL to constraints
2. **Architectural Reasoning** (GNN + Transformer) - Spatial relationships
3. **Layout Generation** (Diffusion) - Room layouts
4. **BIM Object Generation** (Graph Transformer) - Complete BIM
5. **Elevation Generation** (Conditional Diffusion) - Building elevations
6. **Section Generation** (Rule-based + Neural) - Building sections
7. **3D Massing** (Geometric Deep Learning) - 3D geometry
8. **Plan Understanding** (ViT + OCR) - Extract BIM from images
9. **Edit Understanding** (Instruction-tuned LLM) - Interpret edits
10. **Compliance** (RAG + Rule Engine) - Code checking
11. **Quantity Survey** (Transformer Regression) - BOQ generation
12. **Preference Learning** (RL) - Personalization
13. **Design Quality** (Multi-modal Transformer) - Quality scoring

---

## Database Schema

### PostgreSQL (Metadata)
- Users, Projects, Versions
- Collaborators, Comments
- Design Memory (for Learning)
- User Preferences

### Neo4j (BIM Graph)
- Node Labels: Room, Wall, Door, Window, Column, Beam, Slab, Stair, Roof, Foundation, HVAC, Plumbing, Electrical, Material, Schedule
- Relationships: Contains, Connects, Adjacent_To, Supports, Load_Path, etc.

---

## Performance Targets

### System SLAs

**Design Generation:**
- Simple project (2-3 rooms): < 2 minutes
- Medium project (5-10 rooms): < 5 minutes
- Large project (10+ rooms): < 10 minutes

**Editing Operations:**
- Edit interpretation: < 1 second
- BIM update: < 2 seconds
- Visualization refresh: < 500ms

**API Endpoints:**
- 99.9% availability
- P50 latency: < 200ms
- P95 latency: < 1 second

### Scalability

- Concurrent Users: 1,000+
- Projects per system: 100,000+
- Designs per hour: 50-100
- Horizontal scaling via Kubernetes

---

## Security Architecture

### Authentication & Authorization
- OAuth 2.0 (Google, GitHub, Enterprise SSO)
- JWT tokens (1-hour expiry)
- MFA support (TOTP, email)
- Role-based access control

### Data Encryption
- TLS 1.3 for all transport
- AES-256 encryption at rest
- Field-level encryption for sensitive data

---

## Deployment

### Cloud Infrastructure (AWS)
- ECS Fargate for API services
- EC2/EKS for AI models (A100 GPUs)
- RDS Aurora PostgreSQL
- Neptune/Neo4j for BIM Graph
- ElastiCache Redis
- S3 for file storage
- CloudFront CDN

### Kubernetes
- Helm charts for deployment
- Auto-scaling policies
- StatefulSets for databases
- Persistent volumes for models

---

## See Also

- `bim-graph-schema.md` - BIM data model details
- `ai-models.md` - Detailed AI model specifications
- `api-specification.md` - REST API documentation
- `implementation-roadmap.md` - Development timeline
