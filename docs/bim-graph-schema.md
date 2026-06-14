# BIM Graph Schema

## Overview

The BIM Graph is the central data structure and single source of truth for all platform operations. Every architectural element, constraint, and relationship is modeled as nodes and edges in a directed acyclic graph (DAG).

## Core Node Types

### 1. Spatial Elements

#### Room
```json
{
  "id": "room_uuid",
  "type": "Room",
  "name": "Living Room",
  "level": "Ground Floor",
  "dimensions": {
    "width": 20.0,
    "depth": 15.0,
    "height": 3.0,
    "unit": "feet"
  },
  "area": 300.0,
  "perimeter": 70.0,
  "floor_finish": "Marble",
  "wall_finish": "Paint - Matte",
  "ceiling_finish": "Gypsum",
  "door_locations": ["door_001", "door_002"],
  "window_locations": ["window_001"],
  "electrical_fixtures": ["fixture_001"],
  "hvac_elements": ["hvac_001"],
  "occupancy_type": "Living",
  "constraints": {
    "min_width": 12.0,
    "natural_light": true,
    "accessibility": true
  }
}
```

#### Wall
```json
{
  "id": "wall_uuid",
  "type": "Wall",
  "name": "West Wall - Living Room",
  "parent_room": "room_001",
  "start_point": [0.0, 0.0, 0.0],
  "end_point": [20.0, 0.0, 0.0],
  "height": 3.0,
  "thickness": 0.33,
  "material": "Brick - 210mm",
  "fire_rating": "2HR",
  "sound_rating": "STC 50",
  "openings": ["door_001", "window_001"],
  "structural": false,
  "load_bearing": true
}
```

#### Door
```json
{
  "id": "door_uuid",
  "type": "Door",
  "name": "Entry Door",
  "room_from": "room_001",
  "room_to": "room_002",
  "width": 0.9,
  "height": 2.1,
  "type_category": "Swing",
  "swing_direction": "Left",
  "material": "Wood - Oak",
  "finish": "Stained",
  "hardware": "Brass",
  "fire_rating": "1HR",
  "accessibility_compliant": true,
  "location_on_wall": "wall_001"
}
```

#### Window
```json
{
  "id": "window_uuid",
  "type": "Window",
  "name": "Living Room Window",
  "room": "room_001",
  "width": 1.5,
  "height": 1.2,
  "sill_height": 0.9,
  "frame_material": "Aluminum",
  "glass_type": "Double Glazed Low-E",
  "u_value": 0.28,
  "solar_heat_gain": 0.23,
  "shading_type": "Motorized Blinds",
  "location_on_wall": "wall_001"
}
```

### 2. Structural Elements

#### Column
```json
{
  "id": "column_uuid",
  "type": "Column",
  "name": "C1",
  "level": "Ground Floor",
  "position": [10.0, 5.0, 0.0],
  "height": 3.5,
  "cross_section": {
    "shape": "Rectangular",
    "width": 0.4,
    "depth": 0.4
  },
  "material": "Concrete - C30",
  "reinforcement": "12 mm bars",
  "fire_rating": "2HR",
  "base_type": "Fixed"
}
```

#### Beam
```json
{
  "id": "beam_uuid",
  "type": "Beam",
  "name": "B1",
  "level": "Ground Floor",
  "start_point": [0.0, 5.0, 3.0],
  "end_point": [20.0, 5.0, 3.0],
  "cross_section": {
    "shape": "I-Section",
    "height": 0.5,
    "flange_width": 0.25
  },
  "material": "Steel - Grade 250",
  "supports": ["column_001", "column_002"],
  "loads": ["slab_001"]
}
```

#### Slab
```json
{
  "id": "slab_uuid",
  "type": "Slab",
  "name": "Ground Floor Slab",
  "level": "Ground Floor",
  "thickness": 0.15,
  "material": "Reinforced Concrete",
  "perimeter_points": [[0,0], [20,0], [20,15], [0,15]],
  "area": 300.0,
  "floor_finish": "Tiles",
  "supported_by": ["beam_001", "beam_002"]
}
```

#### Stair
```json
{
  "id": "stair_uuid",
  "type": "Stair",
  "name": "Main Staircase",
  "from_level": "Ground Floor",
  "to_level": "First Floor",
  "tread": 0.28,
  "riser": 0.18,
  "runs": 12,
  "width": 1.2,
  "material": "Hardwood",
  "handrail_type": "Wood",
  "landing_width": 1.5,
  "code_compliant": true
}
```

### 3. Roof & Foundation

#### Roof
```json
{
  "id": "roof_uuid",
  "type": "Roof",
  "name": "Main Roof",
  "roof_type": "Pitched",
  "slope": 30,
  "slope_unit": "degrees",
  "material": "Ceramic Tiles",
  "underlayment": "Bituminous Membrane",
  "insulation": "Polyurethane 100mm",
  "u_value": 0.15,
  "perimeter_points": [[0,0], [20,0], [20,15], [0,15]],
  "eaves_overhang": 0.6
}
```

#### Foundation
```json
{
  "id": "foundation_uuid",
  "type": "Foundation",
  "name": "Strip Foundation",
  "foundation_type": "Continuous Strip",
  "depth": 1.2,
  "width": 0.5,
  "material": "Reinforced Concrete",
  "bearing_capacity": 200,
  "perimeter": 70.0,
  "supported_walls": ["wall_001", "wall_002"]
}
```

### 4. MEP Elements

#### HVAC
```json
{
  "id": "hvac_uuid",
  "type": "HVAC",
  "name": "AC Unit - Living Room",
  "equipment_type": "Cassette Unit",
  "room": "room_001",
  "capacity_btu": 12000,
  "eer": 11.5,
  "refrigerant": "R410A",
  "ductwork": "duct_001",
  "filter_type": "HEPA"
}
```

#### Plumbing
```json
{
  "id": "plumbing_uuid",
  "type": "Plumbing",
  "name": "Main Water Supply",
  "element_type": "Pipe",
  "diameter": 50,
  "material": "Copper",
  "connected_fixtures": ["sink_001", "toilet_001"],
  "water_pressure": 2.5
}
```

#### Electrical
```json
{
  "id": "electrical_uuid",
  "type": "Electrical",
  "name": "Main Panel",
  "element_type": "Panel",
  "voltage": 230,
  "amperage": 200,
  "circuits": ["circuit_001", "circuit_002"],
  "protection": "MCB"
}
```

### 5. Annotations & Documentation

#### Schedule
```json
{
  "id": "schedule_uuid",
  "type": "Schedule",
  "name": "Door Schedule",
  "content": [
    {
      "door_id": "door_001",
      "width": 0.9,
      "height": 2.1,
      "type": "Swing",
      "material": "Wood"
    }
  ]
}
```

#### Material
```json
{
  "id": "material_uuid",
  "type": "Material",
  "name": "Brick - Red",
  "category": "Wall Finish",
  "color": "#A0523D",
  "texture": "Textured",
  "cost_per_unit": 5.50,
  "unit": "sq.ft",
  "supplier": "Local Brick Works"
}
```

## Relationship Types

### Spatial Relationships
- **Contains**: Room contains doors, windows, fixtures
- **Connects**: Door connects two rooms
- **Located_On**: Window located on wall
- **Adjacent_To**: Room adjacent to another room
- **Above**: Upper floor above lower floor

### Structural Relationships
- **Supports**: Beam supports slab, column supports beam
- **Load_Path**: Slab → Beam → Column → Foundation
- **Constrained_By**: Element constrained by building grid
- **Intersects**: Ductwork intersects beam (coordination)

### MEP Relationships
- **Branches_To**: Main pipe branches to fixture
- **Runs_Through**: Ductwork runs through wall cavity
- **Connected_To**: Electrical outlet connected to circuit
- **Served_By**: Room served by HVAC zone

### Design Relationships
- **References**: Schedule references multiple doors
- **Uses_Material**: Wall uses brick material
- **Complies_With**: Design complies with code
- **Derived_From**: Elevation derived from BIM

## Graph Queries

### Query 1: Room Information
```
MATCH (room:Room {id: "room_001"})
RETURN room.name, room.dimensions, room.area
```

### Query 2: Load Path
```
MATCH (slab:Slab)-[:supports*]->(column:Column)-[:supports]->(foundation:Foundation)
RETURN slab, column, foundation
```

### Query 3: Room Connectivity
```
MATCH (room1:Room)-[:connects]-(door:Door)-[:connects]-(room2:Room)
WHERE room1.id = "room_001"
RETURN room2.name
```

### Query 4: Material Usage
```
MATCH (element)-[:uses_material]->(material:Material)
WHERE material.category = "Wall Finish"
RETURN element, COUNT(*) as usage_count
```

## Constraints

### Dimensional Constraints
- Room minimum width: 8 feet
- Door minimum width: 2.6 feet
- Window sill height: 0.9-1.0 meters
- Stair rise: 0.15-0.20 meters
- Stair tread: 0.25-0.30 meters

### Compliance Constraints
- Fire rating requirements per occupancy type
- Accessibility (ADA) compliance
- Energy code compliance (U-values, SHGC)
- Structural load ratings
- Sound transmission class (STC)

### Spatial Constraints
- No overlapping elements
- Proper egress paths
- Minimum aisle widths
- Headroom clearances

## Serialization Format

### Native Format (Graph)
```json
{
  "nodes": [
    { "id": "room_001", "type": "Room", "properties": {...} },
    { "id": "wall_001", "type": "Wall", "properties": {...} }
  ],
  "edges": [
    { "from": "room_001", "to": "wall_001", "relationship": "contains" }
  ]
}
```

### IFC Export
BIM Graph serialized to IFC 4.3 standard

### DWG Export
Flattened to CAD entities for AutoCAD compatibility

## Performance Considerations

- Node indexing by ID, type, level
- Spatial indexing for geometric queries
- Edge caching for frequent relationships
- Lazy loading for large projects
- Incremental updates to dependent elements
