<!--
Copyright 2015-2021 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# EXT_mesh_primitive_edge_visibility

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

3D modeling and computer-aided drafting environments like SketchUp, MicroStation, and Revit provide non-photorealistic visualizations that render 3D objects with their edges visible. The edges can improve the readability of complex models and convey semantics of the underlying topology. The `EXT_mesh_primitive_edge_visibility` extension augments a triangle mesh primitive with sufficient information to enable engines to produce such visualizations.

The image below illustrates a typical rendering of a cylinder with its edges. The width of the edges has been exaggerated for emphasis.

<figure>
<img src="./figures/visible-edges.png"/>
<figcaption><em><b>Figure 1</b> Cylinder mesh with visible edges</em></figcaption>
</figure>

This image shows both of the two types of edges described by `EXT_mesh_primitive_edge_visibility`.
- A [silhouette edge](https://en.wikipedia.org/wiki/Silhouette_edge) is any edge separating a front-facing triangle from a back-facing triangle. In the Figure 1, one silhouette is visible along each of the curved left and right sides of the cylinder. Silhouette edges are *conditionally visible* - their visibility is determined at display time based on the camera direction.
- A hard edge is any edge attached to only a single triangle, or any edge between two logical faces of the 3D object. In the Figure 1, hard edges are visible around the perimeters of the cylinder's circular end caps. Hard edges are *always visible* regardless of the camera direction.

## Shortcomings of Existing Techniques

Various techniques can be applied to a glTF asset to approximate the rendering in Figure 1. One of the simplest approaches uses only the information encoded in the triangle mesh itself to produce a wiremesh rendering in which every edge of every triangle is visible, as shown in Figure 2.

<figure>
<img src="./figures/wiremesh.png"/>
<figcaption><em><b>Figure 2</b> Triangle edges drawn using a wiremesh technique</em></figcaption>
</figure>

The interior edges displayed in Figure 2 can be useful for visualizing the structure of the triangle mesh, but they obscure the semantics of the cylinder topology. Note that each of the vertical edges encircling the cylinder in Figure 2 are potential silhouette edges.

Screen-space techniques (e.g., a "toon shader") can be applied to add edges during image post-processing. Such techniques can only approximate the actual edges. The technique used in Figure 3, for example, fails to reconstruct the edges along the top end cap of the cylinder where they fall within the cylinder's volume projected onto the image plane.

<figure>
<img src="./figures/outline.png"/>
<figcaption><em><b>Figure 3</b> Edges drawn using a screen-space technique</em></figcaption>
</figure>

The hard edges could be encoded explicitly into the glTF asset as additional primitives to be drawn along with the triangle mesh. However, this will generally produce a "stippling" effect where the edges and triangles collide in the depth buffer, as shown in Figure 4.

<figure>
<img src="./figures/depth-fighting.png"/>
<figcaption><em><b>Figure 4</b> Edges drawn as separate primitives</em></figcaption>
</figure>

The [CESIUM_primitive_outline](../CESIUM_primitive_outline/README.md) extension attempts to address the depth fighting artifacts shown in Figure 4 by providing an additional index buffer describing each visible edge as a line segment (a pair of indices into the triangle mesh's list of vertices). The extension leaves the details of how to render the edges without depth-fighting up to the engine. This approach satisfies the use case for which it was intended - displaying the edges of boxy, low-resolution buildings - but suffers some limitations:
- The surface geometry must be represented as indexed triangles.
- Silhouette edges are not supported.
- The representation of the edges are pairs of vertex indices can significantly increase the size of the glTF asset.
- The edges cannot specify their own materials.

<figure>
<img src="./figures/hard-edges.png"/>
<figcaption><em><b>Figure 5</b> Hard edges encoded using CESIUM_primitive_outline</em></figcaption>
</figure>

## glTF Schema Updates

The `EXT_mesh_primitive_edge_visibility` extension is applied to a mesh primitive of topology type 4 (triangles), 5 (triangle strip), or 6 (triangle fan) that uses indexed or non-indexed geometry.

### Edge Visibility

The visibility of a single edge is specified using 2 bits as one of the following visibility values:
- 0: Hidden edge - the edge should never be drawn.
- 1: Silhouette edge - the edge should be drawn only when one of its adjacent faces is facing toward the camera and the other is facing away from the camera.
- 2: Hard edge - the edge should always be drawn.
- 3: Hard edge encoded in `primitives` - a representation of the edge is included in the extension's `primitives` property. The edge should always be drawn, either by drawing the `primitives` array or by drawing it like an edge with visibility value `2`.

The extension's `visibility` property specifies the index of an accessor of `SCALAR` type and `UNSIGNED BYTE` component type that encodes as a bitfield the visibility of every edge of every triangle in the mesh. The ordering of triangles and vertices is as described by section 3.7.2.1 of the glTF 2.0 specification. For each triangle `(v0, v1, v2)`, the bitfield encodes three visibility values for the edges `(v0:v1, v1:v2, v2:v0)` in that order. Therefore, the number of bytes in the bitfield **MUST** be `6 * N / 8` rounded up to the nearest whole number, where `N` is the number of triangles in the primitive.
[Mention ordering of triangles when primitive restart extension is present?]

Engines **MUST** render all edges according to their specified visibility values. Engines may choose a material with which to draw the edges - e.g., they could apply the primitive's material to the edges, or apply some uniform material to the edges of all primitives in the scene, or some other consistent option. If engines choose to render the edges specified by the `primitives` property, then they **MUST NOT** also produce their own rendering of the edges with visibility value `3`.

### Edge Primitives

The extension's `primitives` property optionally provides an array of primitives of topology type 1 (lines), 2 (line loop), and/or 3 (line strip) representing all or a subset of the hard edges encoded in the `visibility` property. The primary use case is for edges that should be drawn using a different material than the corresponding surface - for example, a red outline drawn around a green shape. Engines should render these primitives directly, in which case they **MUST** do so without depth-fighting and **MUST NOT** also produce their own rendering of these edges.

All edges included in `primitives` **MUST** be encoded with visibility value `3` in the `visibility` bitfield.

### Constraints

If `visibility` contains any visibility values of `3`, the extension's `primitives` property **MUST** be defined.

## Example


## JSON Schema


## Implementation Notes

This extension does not dictate any specific rendering technique. It only requires that the edges be drawn and that they avoid depth-fighting wit the corresponding surfaces. [TODO example implementations].


[
  impls MAY render the edge primitives provided
  all edge visibility must be encoded including the edges that have corresponding primitives
  BENTLEY_primitive_restart may be applied to the primitives
    Relevant to naming discussion for that extension?
    Need to establish how dependencies between extensions is expected to work...
      KHR_mesh_quantization depends upon KHR_texture_transform
  Edges MUST draw in front of triangles. No specific implementation prescribed. Some examples.
  Visibility of a shared edge is encoded only once; subsequent occurences are encoded as 0 (not visible)
  required: visibility
  required if any silhouettes in `visibility`: normals
  required if any entry in `visibility` is `2` indicating hard edge represented in `primitives`: primitives
  `normals` may use KHR_mesh_quantization -- need to establish how dependencies between extensions is expected to work...
]

[
  probably don't REQUIRE edges to be drawn - the edge visibility can be used for other purposes
  or at least, apps may want option to turn edge display on and off
]

