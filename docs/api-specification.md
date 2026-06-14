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

## BIM Operations API

### Get BIM Graph
```
GET /projects/{projectId}/bim
?include=rooms,walls,doors,windows
&level=Ground Floor

Response: 200 OK
{
  "nodes": [
    {
      "id": "room_001",
      "type": "Room",
      "properties": {
        "name": "Living Room",
        "area": 300,
        "dimensions": { "width": 20, "depth": 15, "height": 3 }
      }
    }
  ],
  "edges": [
    {
      "from": "room_001",
      "to": "door_001",
      "relationship": "contains"
    }
  ]
}
```

### Get BIM Element
```
GET /projects/{projectId}/bim/elements/{elementId}

Response: 200 OK
{
  "id": "room_001",
  "type": "Room",
  "name": "Living Room",
  "level": "Ground Floor",
  "dimensions": { "width": 20, "depth": 15, "height": 3 },
  "area": 300,
  "perimeter": 70,
  "properties": {
    "floor_finish": "Marble",
    "wall_finish": "Paint - Matte",
    "ceiling_finish": "Gypsum"
  },
  "constraints": {
    "min_width": 12,
    "natural_light": true
  },
  "related_elements": {
    "doors": ["door_001", "door_002"],
    "windows": ["window_001"],
    "adjacent_rooms": ["room_002"]
  }
}
```

### Query BIM Graph (Cypher)
```
POST /projects/{projectId}/bim/query
Content-Type: application/json

{
  "query": "MATCH (r:Room)-[:contains]->(d:Door) WHERE r.name = 'Living Room' RETURN d",
  "parameters": {}
}

Response: 200 OK
{
  "results": [
    { "d": { "id": "door_001", "type": "Door", "width": 0.9 } }
  ]
}
```

### Update BIM Element
```
PATCH /projects/{projectId}/bim/elements/{elementId}
Content-Type: application/json

{
  "properties": {
    "area": 350,
    "floor_finish": "Tiles"
  }
}

Response: 200 OK
{
  "id": "room_001",
  "updated_properties": ["area", "floor_finish"],
  "affected_elements": ["door_001", "wall_001"],
  "compliance_status": "Valid"
}
```

---

## AI Editing API

### Conversational Edit
```
POST /projects/{projectId}/ai/edit
Content-Type: application/json

{
  "command": "Increase living room by 25%",
  "context": {
    "selected_elements": ["room_001"],
    "conversation_history": [...]
  }
}

Response: 202 Accepted
{
  "edit_id": "edit_uuid",
  "status": "processing",
  "interpretation": {
    "operation": "scale_room",
    "target_room": "room_001",
    "scale_factor": 1.25
  }
}
```

### Get Edit Result
```
GET /projects/{projectId}/ai/edit/{editId}

Response: 200 OK
{
  "edit_id": "edit_uuid",
  "status": "completed",
  "modifications": [
    {
      "element_id": "room_001",
      "changes": { "area": { "old": 300, "new": 375 } }
    }
  ],
  "validation": {
    "compliant": true,
    "quality_score": 0.91
  }
}
```

### Design Refinement
```
POST /projects/{projectId}/ai/refine
Content-Type: application/json

{
  "aspect": "facade",
  "style": "modern",
  "intensity": 0.8
}

Response: 202 Accepted
{
  "refine_id": "refine_uuid",
  "status": "processing",
  "modifications_count": 15
}
```

### Get Suggestions
```
GET /projects/{projectId}/ai/suggestions
?type=improvement&limit=5

Response: 200 OK
{
  "suggestions": [
    {
      "id": "sug_001",
      "type": "compliance",
      "severity": "warning",
      "message": "Bedroom size below minimum (65 sqft vs 70 sqft required)",
      "recommended_fix": "Increase width by 1 foot",
      "estimated_effort": "low"
    }
  ]
}
```

---

## Analysis API

### Compliance Check
```
POST /projects/{projectId}/analysis/compliance
Content-Type: application/json

{
  "codes": ["IBC 2021", "IECC 2021", "ADA"],
  "location": "Austin, TX"
}

Response: 202 Accepted
{
  "analysis_id": "analysis_uuid",
  "status": "processing"
}

GET /projects/{projectId}/analysis/compliance/{analysisId}

Response: 200 OK
{
  "compliance_report": {
    "location": "Austin, TX",
    "codes_checked": ["IBC 2021", "IECC 2021", "ADA"],
    "compliance_score": 0.94,
    "violations": [
      {
        "severity": "Critical",
        "element": "bedroom_002",
        "rule": "Minimum bedroom area 70 sqft",
        "current": 65,
        "required": 70,
        "solution": "Increase width by 1 foot"
      }
    ],
    "passed_checks": 87,
    "failed_checks": 5
  }
}
```

### Quality Assessment
```
POST /projects/{projectId}/analysis/quality

Response: 200 OK
{
  "quality_assessment": {
    "overall_score": 82,
    "spatial_efficiency": 85,
    "compliance": 92,
    "aesthetics": 75,
    "functionality": 88,
    "sustainability": 72,
    "recommendations": [
      "Improve window-to-wall ratio (currently 25%, ideal 30%)",
      "Consider cross-ventilation paths"
    ]
  }
}
```

### Bill of Quantities
```
POST /projects/{projectId}/survey/boq

Response: 202 Accepted
{
  "survey_id": "survey_uuid",
  "status": "processing"
}

GET /projects/{projectId}/survey/boq/{surveyId}

Response: 200 OK
{
  "boq": {
    "structure": [
      {
        "item": "Foundations",
        "description": "Continuous strip foundation, 1200mm deep",
        "unit": "meter",
        "quantity": 140,
        "rate": 150,
        "amount": 21000
      }
    ],
    "total_cost": 150000,
    "contingency_10%": 15000,
    "grand_total": 165000
  }
}
```

---

## Export API

### Export to IFC
```
GET /projects/{projectId}/export/ifc
?version=4.3&include=geometry,properties,relationships

Response: 200 OK
Content-Type: application/octet-stream
Content-Disposition: attachment; filename="project.ifc"

[Binary IFC file]
```

### Export to DWG
```
GET /projects/{projectId}/export/dwg
?include_layers=true&include_annotations=true

Response: 200 OK
Content-Type: application/octet-stream
Content-Disposition: attachment; filename="project.dwg"

[Binary DWG file]
```

### Export to PDF
```
POST /projects/{projectId}/export/pdf
Content-Type: application/json

{
  "include": ["plans", "elevations", "sections", "schedules"],
  "scale": "1:100",
  "page_size": "A1"
}

Response: 202 Accepted
{
  "export_id": "export_uuid",
  "status": "processing",
  "estimated_pages": 15
}

GET /projects/{projectId}/export/pdf/{exportId}

Response: 200 OK (when ready)
Content-Type: application/pdf
[PDF file]
```

### Export to GLTF
```
GET /projects/{projectId}/export/gltf
?lod=detailed&include_materials=true

Response: 200 OK
Content-Type: model/gltf-binary

[Binary GLTF file]
```

---

## Collaboration API

### Add Collaborator
```
POST /projects/{projectId}/collaborators
Content-Type: application/json

{
  "email": "colleague@company.com",
  "permission_level": "edit"
}

Response: 201 Created
{
  "collaborator_id": "collab_uuid",
  "user_email": "colleague@company.com",
  "permission_level": "edit",
  "added_at": "2024-01-15T10:30:00Z"
}
```

### Get Collaborators
```
GET /projects/{projectId}/collaborators

Response: 200 OK
{
  "collaborators": [
    {
      "user_id": "user_uuid",
      "email": "colleague@company.com",
      "name": "John Doe",
      "permission_level": "edit",
      "status": "active"
    }
  ]
}
```

### Add Comment
```
POST /projects/{projectId}/comments
Content-Type: application/json

{
  "element_id": "room_001",
  "text": "The living room feels too narrow",
  "type": "general"
}

Response: 201 Created
{
  "comment_id": "comment_uuid",
  "user_id": "user_uuid",
  "element_id": "room_001",
  "text": "The living room feels too narrow",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### Get Comments
```
GET /projects/{projectId}/comments
?element_id=room_001&resolved=false

Response: 200 OK
{
  "comments": [...],
  "total": 5
}
```

---

## Version Control API

### Get Versions
```
GET /projects/{projectId}/versions
?page=1&limit=20

Response: 200 OK
{
  "versions": [
    {
      "version_number": 3,
      "created_by": "user_uuid",
      "created_at": "2024-01-15T10:30:00Z",
      "message": "Increased living room size",
      "changes_summary": { "modified_elements": 5, "deleted_elements": 0 }
    }
  ],
  "total": 10,
  "current_version": 3
}
```

### Restore Version
```
POST /projects/{projectId}/versions/{versionNumber}/restore

Response: 200 OK
{
  "current_version": 2,
  "restored_from": 3,
  "restored_at": "2024-01-15T10:35:00Z"
}
```

### Compare Versions
```
GET /projects/{projectId}/versions/{versionA}/compare/{versionB}

Response: 200 OK
{
  "differences": {
    "modified_elements": [
      {
        "id": "room_001",
        "changes": {
          "area": { "from": 300, "to": 375 },
          "floor_finish": { "from": "Marble", "to": "Tiles" }
        }
      }
    ],
    "added_elements": ["door_003"],
    "removed_elements": []
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

---

## Webhooks

Register webhook:
```
POST /users/{userId}/webhooks

{
  "url": "https://your-domain.com/webhook",
  "events": ["design.completed", "design.failed", "collaboration.joined"],
  "secret": "webhook_secret"
}
```

Webhook events:
```json
{
  "event": "design.completed",
  "project_id": "proj_uuid",
  "generation_id": "gen_uuid",
  "data": {
    "quality_score": 0.92,
    "compliance_score": 0.88
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```
