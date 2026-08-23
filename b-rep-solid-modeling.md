# B-Rep Solid Modeling: Developer Reference and Examples

Boundary representation, or B-Rep, is one of the fundamental data models behind solid modeling in CAD and engineering software. Instead of describing a 3D object as a cloud of points or a collection of rendering triangles, B-Rep represents the object through its boundary: faces, edges and vertices connected into a consistent topological structure.

For developers, this distinction matters because most CAD operations do more than change how an object looks. They modify mathematical geometry and rebuild the relationships between model entities. Understanding that process makes it easier to work with modeling APIs, diagnose failed operations and design application logic around changing geometry.

## Geometry and Topology in a B-Rep Model
A B-Rep model combines two different layers.

Geometry describes mathematical shape. Typical geometric entities include points, lines, circles, spline curves, planes, cylinders, cones and parametric surfaces.

Topology describes how those entities are connected. Common topological entities include:

vertex: a topological point associated with a position
edge: a bounded portion of a curve
loop: an ordered boundary formed by edges
face: a bounded region of a surface
shell: a connected set of faces
body: the complete modeled object
This separation is essential.

A cylindrical surface, for example, can extend indefinitely in its mathematical definition. A cylindrical face on a mechanical part represents only a bounded region of that surface. Topology determines where the face begins and ends and how it connects to surrounding faces.

## Building a Simple Solid from a Profile

Consider a basic extrusion.

The input may be a closed rectangular profile consisting of four line segments. At this stage, the application is working primarily with wireframe geometry.

An extrusion operation creates planar or ruled side surfaces from the profile edges and adds surfaces at both ends. The resulting surfaces are then organized into faces. Their boundaries create edges and vertices, and all faces are connected into a closed shell.

Conceptually, the workflow looks like this:

closed profile → generated surfaces → bounded faces → closed shell → solid body

The application may expose this as a single API operation, but the resulting object contains a full geometric and topological structure.

Developers can later traverse that structure to inspect faces, retrieve edges or identify the underlying surfaces.

## Why Boolean Operations Are More Complicated

Boolean operations demonstrate why solid modeling cannot be reduced to simple coordinate calculations.

Suppose a cylindrical body passes through a rectangular block and must be subtracted to create a hole.

The modeling system first determines where the boundaries of the two bodies intersect. Intersection curves are calculated between relevant surfaces. Existing faces may then need to be split along those curves.

The resulting regions must be classified according to whether they belong inside or outside the final body.

A [geometric kernel](https://c3dlabs.com/products/c3d-toolkit/modeler/) then constructs the topology representing the result. New cylindrical faces may appear, original planar faces may acquire additional inner loops, and some portions of the input bodies disappear entirely.

The visible result is a block with a hole. Internally, the B-Rep may have changed substantially.

## Face, Surface and Trimmed Region Are Not Equivalent

A common source of confusion when working with a modeling SDK is treating a face and its underlying surface as the same object.

They are related, but they represent different concepts.

Suppose a planar face contains a circular hole. The mathematical plane has no hole and may have no finite boundary at all. The topology of the face defines an outer loop and an inner loop. Together, those boundaries specify the portion of the plane that belongs to the solid.

The same principle applies to more complex surface modeling.

This distinction affects API usage. An operation that retrieves a surface provides mathematical geometry. An operation that processes a face must also account for trimming boundaries and topological orientation.

A geometric modeling kernel maintains both levels so that downstream operations can reason about precise shape and model connectivity.

## Example: What Happens During Filleting

Filleting provides another useful example of B-Rep modification.

Assume two planar faces meet along a sharp edge. The user requests a rounded transition.

The modeling engine must identify the adjacent faces, determine the region affected by the requested radius and construct the appropriate blending surface.

Parts of the original faces are trimmed away. New intersection curves define their modified boundaries. The original sharp edge disappears and new edges connect the blend to the remaining faces.

The operation can fail if the requested radius cannot be constructed within the surrounding geometry. This is why production CAD software needs meaningful error handling rather than assuming every geometric operation has a valid result.

## Topology Is Not Stable Across Model Changes

Application developers should not assume that a face or edge will remain structurally identical after a model is rebuilt.

Consider a body containing a pocket. If an upstream dimension changes, one face may become two faces because of a new intersection. Another edge may disappear entirely.

This creates a problem when higher-level application data refers to specific topological entities.

Examples include:

machining operations attached to faces
dimensions associated with edges
annotations attached to geometry
feature dependencies
engineering metadata assigned to model regions
The resulting shape may look almost unchanged while the internal topology differs.

Applications that support parametric or history-based modeling therefore need strategies for tracking geometric entities across modifications rather than relying only on persistent object identifiers.

## Numerical Tolerances Are Part of B-Rep Processing

B-Rep modeling also depends on numerical tolerance.

Floating-point calculations rarely produce mathematically perfect results. Two computed points may be intended to coincide while differing by a very small distance.

A CAD kernel must decide when entities should be considered identical or connected.

This affects surface intersections, edge construction, Boolean operations and imported geometry. If tolerance handling is too strict, valid-looking models may fail to connect. If it is too permissive, unrelated geometric entities may be merged.

External CAD data makes the problem more visible because imported models may contain gaps, short edges or inconsistent boundaries that are not obvious in a rendered view.

## B-Rep as an Application-Level Data Source

B-Rep is useful beyond interactive solid modeling.

CAM software can analyze faces and edges to identify machining regions. CAE preprocessing tools can inspect bodies before meshing. Inspection software can compare measured data with mathematically defined surfaces.

CAD applications can also use topology for feature recognition, geometric queries and model analysis.

A developer might inspect a body by traversing its faces, determine which faces are cylindrical, retrieve their axes and radii, and use adjacency relationships to identify potential holes.

The B-Rep therefore becomes structured engineering data rather than simply the internal representation of a visible 3D object.

## The Practical Mental Model for Developers

When working with B-Rep, it helps to think in two parallel layers.

Geometry answers: What is the mathematical shape?

Topology answers: How is that shape used and connected inside this model?

Most nontrivial CAD operations modify both.

An extrusion creates new geometry and assembles topology. A Boolean operation intersects geometry and reconstructs topology. A fillet generates blending surfaces while replacing edges and trimming neighboring faces.

For CAD developers, understanding this relationship is the foundation for working effectively with solid modeling APIs. B-Rep is not merely a storage format for 3D models. It is the structural model that allows engineering software to create, inspect and modify precise geometry while preserving the relationships that make a collection of surfaces behave like a solid object.
