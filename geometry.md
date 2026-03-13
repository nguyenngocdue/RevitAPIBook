# Geometry Service Documentation

## Overview

The `GeometryService` class provides utility methods for extracting and working with solid geometry from Revit elements. This service is part of the DeepBIM toolkit and is located in the `DeepBIM.Services.Geometry` namespace.

## Purpose

The primary purpose of this service is to simplify the extraction of valid solid geometries from Revit elements. In Revit API, geometry extraction can be complex due to nested geometry instances and various geometry object types. This service handles these complexities and provides a clean interface for obtaining all solids from a collection of elements.

## Key Features

- **Solid Extraction**: Extracts all valid solid objects from a list of Revit elements
- **Recursive Processing**: Handles nested geometry instances automatically
- **Volume Validation**: Filters out invalid solids (solids with zero volume)
- **High-Quality Geometry**: Uses fine detail level for accurate geometry representation

## Class Structure

### Namespace
```csharp
DeepBIM.Services.Geometry
```

### Class Type
`public class GeometryService`

## Methods

### GetSolids

```csharp
public static List<Solid> GetSolids(IList<Element> elements)
```

#### Description
Extracts all valid solid objects from a list of Revit elements. This is the main public method that users should call.

#### Parameters
- `elements` (IList<Element>): A collection of Revit elements from which to extract solids

#### Returns
- `List<Solid>`: A list containing all valid solid geometries found within the provided elements

#### Behavior
1. Validates input (returns empty list if input is null or empty)
2. Configures geometry extraction options:
   - `ComputeReferences = true`: Enables reference computation for downstream operations
   - `DetailLevel = ViewDetailLevel.Fine`: Extracts geometry with maximum detail
   - `IncludeNonVisibleObjects = false`: Excludes hidden or non-visible geometry
3. Iterates through each element and extracts its geometry
4. Delegates to `ExtractSolids` for processing each geometry element
5. Returns accumulated list of all valid solids

#### Usage Example
```csharp
// Get selected elements
IList<Element> selectedElements = // ... your element selection code
var geometryService = new GeometryService();
List<Solid> solids = GeometryService.GetSolids(selectedElements);

// Process the extracted solids
foreach (Solid solid in solids)
{
    double volume = solid.Volume;
    // ... perform operations with the solid
}
```

---

### ExtractSolids

```csharp
private static void ExtractSolids(GeometryElement geometryElement, List<Solid> solids)
```

#### Description
A private helper method that recursively extracts solid objects from a geometry element. This method handles both direct solids and nested geometry instances.

#### Parameters
- `geometryElement` (GeometryElement): The geometry element to process
- `solids` (List<Solid>): The list to which extracted solids are added (passed by reference)

#### Behavior
1. Iterates through all geometry objects in the geometry element
2. For each geometry object:
   - If it's a `Solid` with volume > 0: adds it to the solids list
   - If it's a `GeometryInstance`: recursively processes its instance geometry
3. This recursive approach ensures all solids are found, even in complex nested structures

#### Recursion Logic
The method handles Revit's geometry hierarchy:
- **Direct Solids**: Directly added to the output list if valid
- **Geometry Instances**: Processed recursively to extract nested solids (common in family instances)

## Geometry Extraction Options

The service uses the following configuration for geometry extraction:

| Option | Value | Purpose |
|--------|-------|---------|
| `ComputeReferences` | `true` | Enables face and edge references for advanced operations |
| `DetailLevel` | `ViewDetailLevel.Fine` | Provides highest quality geometry representation |
| `IncludeNonVisibleObjects` | `false` | Excludes hidden elements for performance |

## Common Use Cases

### 1. Volume Calculation
Extract solids from elements to calculate total volume:
```csharp
var solids = GeometryService.GetSolids(elements);
double totalVolume = solids.Sum(s => s.Volume);
```

### 2. Collision Detection
Get solids for interference checking:
```csharp
var solids1 = GeometryService.GetSolids(group1);
var solids2 = GeometryService.GetSolids(group2);
// Perform collision detection
```

### 3. Geometry Analysis
Extract geometry for custom analysis:
```csharp
var solids = GeometryService.GetSolids(elements);
foreach (var solid in solids)
{
    // Analyze faces, edges, vertices
    foreach (Face face in solid.Faces)
    {
        // Process each face
    }
}
```

## Technical Notes

### Performance Considerations
- The service filters null elements to prevent exceptions
- Only solids with volume > 0 are included (avoids invalid geometry)
- Recursive processing may be intensive for elements with deeply nested geometry

### Revit API Dependencies
This service requires the following Revit API types:
- `Autodesk.Revit.DB.Element`
- `Autodesk.Revit.DB.Solid`
- `Autodesk.Revit.DB.GeometryElement`
- `Autodesk.Revit.DB.GeometryInstance`
- `Autodesk.Revit.DB.Options`
- `Autodesk.Revit.DB.ViewDetailLevel`

### Best Practices
1. **Filter elements before calling**: Pre-filter elements by category or type for better performance
2. **Dispose of geometry**: While this service returns solids, be mindful of memory when processing large datasets
3. **Handle empty results**: Always check if the returned list is empty before processing

## Error Handling

The service implements defensive programming:
- Returns empty list if input is `null` or empty
- Skips `null` elements during iteration
- Checks for `null` geometry elements before processing
- Validates solid volume before adding to results

## Integration

This service is designed to work seamlessly with other DeepBIM services and can be used in:
- Custom Revit commands
- External applications
- Batch processing workflows
- Geometry analysis tools

## Version Compatibility

This service uses standard Revit API geometry classes and should be compatible with multiple Revit versions. Ensure your project references the appropriate Revit API version.

---

## Summary

The `GeometryService` class provides a robust, easy-to-use interface for extracting solid geometry from Revit elements. By handling the complexities of geometry extraction, nested instances, and validation, it allows developers to focus on their specific geometry processing needs rather than low-level API details.
