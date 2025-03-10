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

3D modeling and computer-aided drafting environments like SketchUp, MicroStation, and Revit provide non-photorealistic visualizations that render 3D objects with their edges outlined. The edges can improve the readability of complex models and convey semantics of the underlying topology. The EXT_mesh_primitive_edge_visibility extension encodes the visibility of each edge in a triangle mesh and optionally includes additional primitives that can be used to render some or all of the edges.

The image below illustrates the typical outlined rendering of a cylinder, with 

<figure>
<img src="./figures/outline.png"/>
<figcaption><em><b>Figure 1</b> Outlined cylinder mesh</em></figcaption>
</figure>

Multiple partial solutions to this problem exist using glTF. One of the simplest approaches uses only the information provided by the triangle mesh itself to produce a wiremesh rendering in which every edge of every triangle is visible, as shown below.

<figure>
<img src="./figures/wiremesh.png"/>
<figcaption><em><b>Figure 2</b> Wiremesh rendering of triangle edges</em></figcaption>
</figure>

[for toon, use iTwin.js emphasis shader with no edges]

[
  Alternatives:
    Wiremesh
    Toon shader
    Line primitives
    CESIUM_primitive_outline
]


[other alternative solutions include fragment shader silhouettes ("toon shaders") which lack accuracy required in many engineering and modeling workflows]

[TODO: Talk about preserving edge information when producing low-level facets from higher-level geometry like solids and surfaces. You can't infer from facet edges.]

Non-photorealistic 3D surfaces, such as untextured buildings, are often more visually compelling when their edges are outlined. Such visualizations are common in computer-aided drafting and 3D modeling environments like Revit, SketchUp, and MicroStation. A simplistic approach to adding outlines to a glTF model would be to include additional primitives of type `LINES` representing the visible edges of the model. However, this approach suffers from z-fighting between the lines and their triangles.

The [CESIUM_primitive_outline](https://github.com/KhronosGroup/glTF/blob/main/extensions/2.0/Vendor/CESIUM_primitive_outline/README.md) extension addresses a subset of the problem by permitting the edges of the surfaces to be specified as indices into the surfaces' own vertex buffers, leaving the details of how to render the edges without z-fighting up to the implementation. However, this approach has a few limitations:
- The surface geometry must be represented as indexed `TRIANGLES` primitives.
- The edges cannot specify a different material from the corresponding surface.
- The representation of the edges as pairs of vertex indices can increase the amount of geometry in the model significantly.
- The edges are always visible - conditionally-visible interior edges ("silhouettes", as [described below](#types-of-edges)) are not supported.

The `BENTLEY_edge_visibility` extension addresses the above limitations by compactly encoding the visibility of each edge of any triangle mesh, including conditionally-visible silhouette edges and, optionally, a set of primitives that can be used to render some or all of the edges. In addition to simply rendering the edges, this information can support analytical use cases that leverage the additional topological semantics encoded into the mesh.

## Types of edges

[
  boundary edge = only one triangle (unshared edge) - special case of "hard" edge
  hard edge = always visible (not a curved surface) - are hard edges and curved edges mutually exclusive?
    A hard boundary edge can come from the perimeter of a curved surface
  silhouette edge = conditionally visible, between two triangles, always from a curved surface?
  Yeah I think silhouette = shared edge between two faces of a curved surface
]

[
  impls MAY render the edge primitives provided
  all edge visibility must be encoded including the edges that have corresponding primitives
  BENTLEY_primitive_restart may be applied to the primitives
    Relevant to naming discussion for that extension?
  Edges MUST draw in front of triangles. No specific implementation prescribed. Some examples.
]
