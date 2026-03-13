# Surface Geometry Service Documentation

## Overview

The `SurfaceGeometryService` class provides utility methods for extracting face geometry from Revit elements and solids. This service specializes in working with surface-level geometry, including both general faces and planar faces. It is part of the DeepBIM toolkit and is located in the `DeepBIM.Services.Geometry` namespace.

## Purpose

The primary purpose of this service is to simplify the extraction of face geometry from Revit elements and solids. Faces are fundamental building blocks in Revit geometry and are essential for operations such as:
- Area calculations
- Surface analysis
- Face-based family placement
- Material application
- UV mapping and texturing
- Normal vector analysis

This service provides convenient methods to extract all faces or only planar faces from elements or solids, saving developers from writing repetitive geometry traversal code.

## Key Features

- **Face Extraction**: Extract all faces from solids or elements
- **Planar Face Filtering**: Extract only planar (flat) faces from geometry
- **Dual Input Types**: Work with either Revit elements or pre-extracted solids
- **Type Safety**: Properly filters and casts faces to specific types (e.g., PlanarFace)
- **Null Safety**: Handles null inputs and invalid geometry gracefully

## Class Structure

### Namespace
```csharp
DeepBIM.Services.Geometry
```

### Class Type
`public static class SurfaceGeometryService`

**Note**: This is a static class, so all methods are static and no instantiation is required.

## Methods

### GetFaceBySolids

```csharp
public static IList<Face> GetFaceBySolids(IList<Solid> solids)
```

#### Description
Extracts all faces from a collection of solid objects. This method iterates through each solid and collects all faces into a single list.

#### Parameters
- `solids` (IList<Solid>): A collection of Revit solid objects from which to extract faces

#### Returns
- `IList<Face>`: A list containing all faces from all provided solids

#### Behavior
1. Creates an empty list to store faces
2. Iterates through each solid in the input collection
3. For each solid, iterates through all faces and adds them to the result list
4. Returns the complete list of all faces

#### Usage Example
```csharp
// Assuming you have a list of solids
List<Solid> solids = GeometryService.GetSolids(elements);
IList<Face> faces = SurfaceGeometryService.GetFaceBySolids(solids);

// Process the faces
foreach (Face face in faces)
{
    double area = face.Area;
    // ... perform operations with the face
}
```

---

### GetFaceByElements

```csharp
public static IList<Face> GetFaceByElements(IList<Element> elements)
```

#### Description
Extracts all faces from a collection of Revit elements. This is a convenience method that combines solid extraction and face extraction in one call.

#### Parameters
- `elements` (IList<Element>): A collection of Revit elements from which to extract faces

#### Returns
- `IList<Face>`: A list containing all faces from all solids found in the provided elements

#### Behavior
1. Uses `GeometryService.GetSolids()` to extract solids from the elements
2. Iterates through each solid and collects all faces
3. Returns the complete list of all faces

#### Usage Example
```csharp
// Get selected elements
IList<Element> selectedElements = // ... your element selection code
IList<Face> faces = SurfaceGeometryService.GetFaceByElements(selectedElements);

// Calculate total surface area
double totalArea = 0;
foreach (Face face in faces)
{
    totalArea += face.Area;
}
```

---

### GetPlanarFaceBySolids

```csharp
public static IList<PlanarFace> GetPlanarFaceBySolids(IList<Solid> solids)
```

#### Description
Extracts only planar (flat) faces from a collection of solid objects. This method filters out curved faces and returns only faces that lie in a single plane.

#### Parameters
- `solids` (IList<Solid>): A collection of Revit solid objects from which to extract planar faces

#### Returns
- `IList<PlanarFace>`: A list containing only the planar faces from all provided solids

#### Behavior
1. Validates input (returns empty list if solids is null or empty)
2. Iterates through each solid, skipping null solids
3. For each face in the solid, checks if it's a `PlanarFace`
4. Only adds faces that are successfully cast to `PlanarFace`
5. Returns the filtered list of planar faces

#### Usage Example
```csharp
List<Solid> solids = GeometryService.GetSolids(elements);
IList<PlanarFace> planarFaces = SurfaceGeometryService.GetPlanarFaceBySolids(solids);

// Get normal vectors of all planar faces
foreach (PlanarFace face in planarFaces)
{
    XYZ normal = face.FaceNormal;
    // Check if face is horizontal (normal pointing up/down)
    if (Math.Abs(normal.Z) > 0.99)
    {
        // This is a horizontal surface
    }
}
```

---

### GetPlanarFaceByElements

```csharp
public static IList<PlanarFace> GetPlanarFaceByElements(IList<Element> elements)
```

#### Description
Extracts only planar (flat) faces from a collection of Revit elements. This is the highest-level convenience method that handles the complete workflow from elements to planar faces.

#### Parameters
- `elements` (IList<Element>): A collection of Revit elements from which to extract planar faces

#### Returns
- `IList<PlanarFace>`: A list containing only the planar faces from all elements

#### Behavior
1. Validates input (returns empty list if elements is null or empty)
2. Uses `GeometryService.GetSolids()` to extract solids from elements
3. Validates extracted solids (returns empty list if no solids found)
4. Delegates to `GetPlanarFaceBySolids()` for final planar face extraction
5. Returns the filtered list of planar faces

#### Usage Example
```csharp
IList<Element> walls = // ... get wall elements
IList<PlanarFace> planarFaces = SurfaceGeometryService.GetPlanarFaceByElements(walls);

// Find all vertical faces (wall surfaces)
var verticalFaces = planarFaces.Where(f => 
    Math.Abs(f.FaceNormal.Z) < 0.1).ToList();
```

## Face Types in Revit API

Understanding different face types helps in using this service effectively:

| Face Type | Description | Use Cases |
|-----------|-------------|-----------|
| `Face` | Base class for all faces | General face operations, area calculations |
| `PlanarFace` | Flat faces in a single plane | Wall surfaces, floor slabs, normal vector analysis |
| `CylindricalFace` | Cylindrical surfaces | Pipes, columns (not extracted by planar methods) |
| `RuledFace` | Ruled surfaces | Complex geometry (not extracted by planar methods) |
| `HermiteFace` | Free-form surfaces | Complex geometry (not extracted by planar methods) |

## Common Use Cases

### 1. Surface Area Calculation
Calculate total surface area of elements:
```csharp
var faces = SurfaceGeometryService.GetFaceByElements(elements);
double totalArea = faces.Sum(f => f.Area);
```

### 2. Finding Horizontal Surfaces
Identify floors, roofs, or horizontal surfaces:
```csharp
var planarFaces = SurfaceGeometryService.GetPlanarFaceByElements(elements);
var horizontal = planarFaces.Where(f => 
{
    XYZ normal = f.FaceNormal;
    return Math.Abs(normal.Z) > 0.99; // Nearly vertical normal = horizontal surface
}).ToList();
```

### 3. Finding Exterior Walls
Identify outward-facing wall surfaces:
```csharp
var planarFaces = SurfaceGeometryService.GetPlanarFaceByElements(walls);
foreach (PlanarFace face in planarFaces)
{
    XYZ normal = face.FaceNormal;
    XYZ origin = face.Origin;
    // Check if normal points outward based on building geometry
}
```

### 4. Face-Based Family Placement
Get planar faces for placing face-based families:
```csharp
var planarFaces = SurfaceGeometryService.GetPlanarFaceByElements(hostElements);
// Use planarFaces as hosts for FamilyInstance.ByFace()
```

### 5. Material Analysis
Analyze materials applied to surfaces:
```csharp
var faces = SurfaceGeometryService.GetFaceByElements(elements);
foreach (Face face in faces)
{
    ElementId materialId = face.MaterialElementId;
    // Process material information
}
```

## Method Selection Guide

Choose the appropriate method based on your input and requirements:

```
┌─────────────────────────────────────────────────┐
│         Do you already have Solids?             │
└────────────┬──────────────────┬─────────────────┘
             │                  │
            YES                NO
             │                  │
             │         ┌────────▼────────┐
             │         │  Start from     │
             │         │   Elements      │
             │         └────────┬────────┘
             │                  │
    ┌────────▼────────┐         │
    │  Need only      │         │
    │  Planar Faces?  │         │
    └────┬───────┬────┘         │
         │       │              │
        YES     NO              │
         │       │              │
         │       │    ┌─────────▼─────────┐
         │       │    │  Need only        │
         │       │    │  Planar Faces?    │
         │       │    └─────┬───────┬─────┘
         │       │          │       │
         │       │         YES     NO
         │       │          │       │
┌────────▼───────┴──┐  ┌────▼───────▼────────┐
│GetPlanarFaceBySolids│  │GetFaceByElements     │
│                     │  │                      │
└─────────────────────┘  └──────────────────────┘
         │                        │
         │               GetPlanarFaceByElements
         │                        
    GetFaceBySolids
```

## Integration with GeometryService

This service works seamlessly with `GeometryService`:

```csharp
// Step 1: Extract solids using GeometryService
List<Solid> solids = GeometryService.GetSolids(elements);

// Step 2: Extract faces using SurfaceGeometryService
IList<Face> allFaces = SurfaceGeometryService.GetFaceBySolids(solids);
IList<PlanarFace> planarFaces = SurfaceGeometryService.GetPlanarFaceBySolids(solids);

// Or combine both steps
IList<Face> faces = SurfaceGeometryService.GetFaceByElements(elements);
```

## Performance Considerations

### Optimization Tips
1. **Pre-filter elements**: Filter elements by category before extraction to reduce processing time
2. **Reuse solids**: If you need both faces and solids, extract solids once and reuse them
3. **Use planar methods when appropriate**: If you only need planar faces, use the planar methods to avoid processing curved faces

### Performance Comparison
```csharp
// Less efficient: Extract solids twice
var allFaces = SurfaceGeometryService.GetFaceByElements(elements);
var solids = GeometryService.GetSolids(elements); // Duplicate work

// More efficient: Extract solids once
var solids = GeometryService.GetSolids(elements);
var allFaces = SurfaceGeometryService.GetFaceBySolids(solids);
var planarFaces = SurfaceGeometryService.GetPlanarFaceBySolids(solids);
```

## Error Handling

The service implements robust null-checking:
- Returns empty lists for null or empty input
- Skips null solids in `GetPlanarFaceBySolids`
- Handles empty solid collections gracefully
- Type-safe casting for PlanarFace filtering

## Revit API Dependencies

This service requires the following Revit API types:
- `Autodesk.Revit.DB.Element`
- `Autodesk.Revit.DB.Solid`
- `Autodesk.Revit.DB.Face`
- `Autodesk.Revit.DB.PlanarFace`
- `DeepBIM.Services.Geometry.GeometryService` (internal dependency)

## Best Practices

1. **Choose the right method**: Use element-based methods for simplicity, solid-based methods for reusability
2. **Filter by type**: If you only need planar faces, use the planar-specific methods
3. **Handle empty results**: Always check if the returned list has items before processing
4. **Understand face types**: Not all faces are planar; curved surfaces will be excluded by planar methods
5. **Memory considerations**: Large models can produce many faces; consider processing in batches if needed

## Advanced Usage

### Finding Faces by Orientation
```csharp
var planarFaces = SurfaceGeometryService.GetPlanarFaceByElements(elements);

// Group faces by orientation
var horizontal = new List<PlanarFace>();
var vertical = new List<PlanarFace>();
var inclined = new List<PlanarFace>();

foreach (PlanarFace face in planarFaces)
{
    double zComponent = Math.Abs(face.FaceNormal.Z);
    if (zComponent > 0.99)
        horizontal.Add(face);
    else if (zComponent < 0.1)
        vertical.Add(face);
    else
        inclined.Add(face);
}
```

### Face Area Analysis by Element
```csharp
var elements = // ... get elements
var facesByElement = new Dictionary<ElementId, List<Face>>();

List<Solid> solids = GeometryService.GetSolids(elements);
for (int i = 0; i < elements.Count; i++)
{
    var elementId = elements[i].Id;
    var faces = SurfaceGeometryService.GetFaceBySolids(new[] { solids[i] });
    facesByElement[elementId] = faces.ToList();
}
```

## Version Compatibility

This service uses standard Revit API geometry classes and should be compatible with multiple Revit versions. The static class design ensures no instantiation overhead and clean API usage.

---

## Summary

The `SurfaceGeometryService` class provides a comprehensive, easy-to-use interface for extracting face geometry from Revit elements and solids. By offering multiple methods for different input types and output requirements, it gives developers flexibility while maintaining simplicity. The service integrates seamlessly with `GeometryService` and follows defensive programming practices to ensure reliability in production environments.
