# AI BIM Architect - REST API Specification

## Base URL
```
https://api.bim-architect.com/v1
```

## Authentication

All API requests require a valid JWT token in the Authorization header:
```
Authorization: Bearer <jwt_token>
```

### Token Refresh
```
POST /auth/refresh
Body: { refresh_token: "..." }
Response: { access_token: "...", refresh_token: "..." }
```

---

## Projects API

### Create Project
```
POST /projects
Content-Type: application/json

{
  "name": "Downtown Office Complex",
  "description": "Modern office building design",
  "project_type": "Commercial",
  "location": "Austin, TX",
  "is_public": false
}

Response: 201 Created
{
  "id": "proj_uuid",
  "name": "Downtown Office Complex",
  "owner_id": "user_uuid",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### Get Project
```
GET /projects/{projectId}

Response: 200 OK
{
  "id": "proj_uuid",
  "name": "Downtown Office Complex",
  "owner_id": "user_uuid",
  "project_type": "Commercial",
  "location": "Austin, TX",
  "bim_graph_id": "graph_uuid",
  "collaborators": []
}
```

### List Projects
```
GET /projects?page=1&limit=20

Response: 200 OK
{
  "projects": [...],
  "total": 50,
  "page": 1,
  "limit": 20
}
```

### Update Project
```
PUT /projects/{projectId}
Content-Type: application/json

{
  "name": "Updated Name",
  "description": "Updated description"
}

Response: 200 OK
```

### Delete Project
```
DELETE /projects/{projectId}

Response: 204 No Content
```

---

## Design Generation API

### Generate from Natural Language
```
POST /projects/{projectId}/generate/from-text
Content-Type: application/json

{
  "prompt": "Design a modern 4-bedroom duplex on a 50ft x 100ft plot with a 2-car garage, open kitchen, balcony, home office, and swimming pool",
  "style": "Modern",
  "region": "Austin, TX"
}

Response: 202 Accepted
{
  "generation_id": "gen_uuid",
  "status": "processing",
  "estimated_time": 120,
  "webhook_url": "https://your-domain.com/webhook"
}
```

### Generate from Image
```
POST /projects/{projectId}/generate/from-image
Content-Type: multipart/form-data

File: floor_plan.jpg (binary)
Fields:
  - prompt: "Use this floor plan but increase living room by 25%"
  - confidence_threshold: 0.8

Response: 202 Accepted
{
  "generation_id": "gen_uuid",
  "status": "processing",
  "extraction_confidence": 0.87,
  "extracted_elements": { "rooms": 5, "doors": 8, "windows": 12 }
}
```

### Generate from DWG/DXF
```
POST /projects/{projectId}/generate/from-dwg
Content-Type: multipart/form-data

File: plan.dwg (binary)
Fields:
  - extract_layers: true
  - scale_unit: "feet"

Response: 202 Accepted
{
  "generation_id": "gen_uuid",
  "status": "processing",
  "detected_scale": 1/100,
  "layer_count": 32
}
```

### Get Generation Status
```
GET /projects/{projectId}/generate/{generationId}

Response: 200 OK
{
  "generation_id": "gen_uuid",
  "status": "completed",
  "progress": 100,
  "results": {
    "bim_graph_id": "graph_uuid",
    "quality_score": 0.92,
    "compliance_score": 0.88,
    "total_area": 2500,
    "room_count": 10
  }
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Input validation failed",
    "details": {
      "field": "prompt",
      "issue": "Prompt too short (min 10 characters)"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_uuid"
  }
}
```

Common error codes:
- `AUTHENTICATION_ERROR` (401)
- `AUTHORIZATION_ERROR` (403)
- `NOT_FOUND` (404)
- `VALIDATION_ERROR` (400)
- `CONFLICT_ERROR` (409)
- `RATE_LIMIT_EXCEEDED` (429)
- `SERVER_ERROR` (500)

---

## Rate Limiting

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1705315800
```

Limits:
- Design generation: 10/hour per user
- API calls: 1000/hour per user
- File uploads: 100/day per user
- Collaborators per project: 50
