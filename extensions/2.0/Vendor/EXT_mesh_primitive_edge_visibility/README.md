<!--
Copyright 2015-2021 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# BENTLEY_edge_visibility

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

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


