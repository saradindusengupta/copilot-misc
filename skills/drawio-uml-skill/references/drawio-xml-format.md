# draw.io XML Format Reference

## Table of Contents
1. [Document Envelope](#1-document-envelope)
2. [mxCell — the fundamental unit](#2-mxcell--the-fundamental-unit)
3. [Vertices (shapes)](#3-vertices-shapes)
4. [Edges (connections)](#4-edges-connections)
5. [UML Shape Styles](#5-uml-shape-styles)
6. [Relationship Arrow Styles](#6-relationship-arrow-styles)
7. [Labels and HTML](#7-labels-and-html)
8. [Geometry](#8-geometry)
9. [Compressed vs Uncompressed](#9-compressed-vs-uncompressed)

---

## 1. Document Envelope

Every draw.io file is an XML document with this outer structure:

```xml
<mxGraphModel dx="1422" dy="762" grid="1" gridSize="10" guides="1"
              tooltips="1" connect="1" arrows="1" fold="1" page="1"
              pageScale="1" pageWidth="1169" pageHeight="827"
              math="0" shadow="0" compressed="false">
  <root>
    <mxCell id="0" />                    <!-- always present, the root -->
    <mxCell id="1" parent="0" />        <!-- always present, default layer -->
    <!-- all user content goes here, with parent="1" (or a group id) -->
  </root>
</mxGraphModel>
```

Key attributes on `mxGraphModel`:
- `compressed="false"` — XML is readable. If `true`, the inner content is base64+deflate encoded
- `pageWidth` / `pageHeight` — canvas size in pixels (A4: 1169×827, Letter: 1100×850)

---

## 2. mxCell — the fundamental unit

Everything visible is an `mxCell`. Two roles:

| Role | Key attribute | Has geometry? |
|---|---|---|
| Vertex (shape) | `vertex="1"` | Yes |
| Edge (connector) | `edge="1"` | Optional (for routing points) |

Common attributes:

```xml
<mxCell
  id="5"              <!-- unique integer string, start from 2 -->
  value="ClassName"   <!-- visible label text; can be HTML -->
  style="..."         <!-- draw.io style string -->
  vertex="1"          <!-- or edge="1" -->
  parent="1"          <!-- parent cell id; "1" = default layer; use group id for children -->
>
  <mxGeometry x="100" y="200" width="160" height="60" as="geometry" />
</mxCell>
```

---

## 3. Vertices (shapes)

### 3.1 Plain rectangle (default)

```xml
<mxCell id="2" value="MyClass" style="rounded=0;whiteSpace=wrap;html=1;"
        vertex="1" parent="1">
  <mxGeometry x="80" y="80" width="120" height="60" as="geometry" />
</mxCell>
```

### 3.2 UML Class (three-compartment)

draw.io models a UML class as a **group** with child cells for each compartment:

```xml
<!-- Group cell (the class container) -->
<mxCell id="10" value="" style="shape=table;startSize=30;container=1;collapsible=1;
        childLayout=tableLayout;fixedRows=1;rowLines=0;fontStyle=1;align=center;
        resizeLast=1;fontSize=14;" vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="200" height="150" as="geometry" />
</mxCell>

<!-- Header row (class name) -->
<mxCell id="11" value="ClassName" style="shape=tableRow;horizontal=0;startSize=0;
        swimlaneHead=0;swimlaneBody=0;fillColor=none;collapsible=0;dropTarget=0;
        points=[[0,0.5],[1,0.5]];portConstraint=eastwest;fontSize=14;fontStyle=1;
        top=0;left=0;right=0;bottom=1;" vertex="1" parent="10">
  <mxGeometry y="0" width="200" height="30" as="geometry" />
</mxCell>
```

**Alternative (simpler) style** — single cell with HTML label:

```xml
<mxCell id="2" value="&lt;b&gt;ClassName&lt;/b&gt;&lt;hr/&gt;- attr: Type&lt;br/&gt;+ method(): void"
        style="text;html=1;strokeColor=#000000;fillColor=#ffffff;align=left;
               verticalAlign=top;spacingLeft=4;spacingRight=4;overflow=hidden;
               points=[[0,0],[0.25,0],[0.5,0],[0.75,0],[1,0],
                       [1,0.25],[1,0.5],[1,0.75],[1,1],
                       [0.75,1],[0.5,1],[0.25,1],[0,1],
                       [0,0.75],[0,0.5],[0,0.25]];
               shape=mxgraph.basic.rect;"
        vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="160" height="90" as="geometry"/>
</mxCell>
```

---

## 4. Edges (connections)

```xml
<mxCell id="20" value="" style="endArrow=open;endFill=0;..." edge="1"
        source="10" target="11" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

Key attributes:
- `source` / `target` — cell IDs of connected vertices
- `value` — label on the edge (e.g., multiplicity, relationship name)
- `style` — controls arrowhead shape, line style, etc.

### Edge with multiplicity labels

Use `entryLabel` / `exitLabel` for multiplicity, or place text cells near the endpoints:

```xml
<mxCell id="20" value="places" style="..." edge="1" source="5" target="6" parent="1">
  <mxGeometry relative="1" as="geometry">
    <Array as="points" />  <!-- routing waypoints, if any -->
  </mxGeometry>
</mxCell>
<!-- Separate label cells for multiplicity -->
<mxCell id="21" value="1" style="resizable=0;..." vertex="1" connectable="0" parent="20">
  <mxGeometry x="-0.9" relative="1" as="geometry"><mxPoint as="offset"/></mxGeometry>
</mxCell>
<mxCell id="22" value="*" style="resizable=0;..." vertex="1" connectable="0" parent="20">
  <mxGeometry x="0.9" relative="1" as="geometry"><mxPoint as="offset"/></mxGeometry>
</mxCell>
```

---

## 5. UML Shape Styles

### Class diagram shapes

| UML Element | draw.io style key |
|---|---|
| Class | `shape=mxgraph.flowchart.start_2` or HTML-label rect |
| Interface | `shape=mxgraph.uml.interface` or `«interface»` label |
| Abstract class | `fontStyle=2` (italic name) on class shape |
| Enumeration | `shape=mxgraph.uml.enumeration` or `«enumeration»` label |
| Package | `shape=mxgraph.uml.package2` |
| Note/Comment | `shape=note;whiteSpace=wrap;` |

### Sequence diagram shapes

| UML Element | draw.io style |
|---|---|
| Actor (stick figure) | `shape=mxgraph.uml.actor;` |
| Lifeline (dashed line) | `shape=mxgraph.uml.lifeline;` |
| Activation bar | `shape=mxgraph.uml.activation;` |
| Destruction (X) | `shape=mxgraph.uml.destroy;` |
| Combined fragment | `shape=mxgraph.uml.frame;` |

### Use case diagram shapes

| UML Element | draw.io style |
|---|---|
| Actor | `shape=mxgraph.uml.actor;` |
| Use case (ellipse) | `ellipse;whiteSpace=wrap;html=1;` |
| System boundary | `swimlane;startSize=20;` |

### Activity diagram shapes

| UML Element | draw.io style |
|---|---|
| Initial node | `ellipse;aspect=fixed;fillColor=#000000;` |
| Final node | `doubleEllipse;aspect=fixed;fillColor=#000000;` |
| Action | `rounded=1;whiteSpace=wrap;html=1;` |
| Decision/merge | `rhombus;` |
| Fork/join | `shape=mxgraph.flowchart.merge;` or thin rectangle |

### State diagram shapes

| UML Element | draw.io style |
|---|---|
| State | `rounded=1;whiteSpace=wrap;` |
| Initial pseudostate | `ellipse;fillColor=#000000;` |
| Final state | `doubleEllipse;fillColor=#000000;` |
| History | `ellipse;` + "H" label |
| Junction | `ellipse;fillColor=#000000;` (small) |

---

## 6. Relationship Arrow Styles

### Class diagram arrows

| UML Relationship | `endArrow` | `endFill` | Line style |
|---|---|---|---|
| Association | `open` | `0` | solid |
| Directed association | `open` | `0` | solid, one direction |
| Dependency | `open` | `0` | `dashed=1` |
| Aggregation | `ERzeroToMany` or `diamond` | `0` | solid |
| Composition | `ERmandOne` or `diamond` | `1` | solid |
| Inheritance (Generalization) | `block` | `0` (hollow) | solid |
| Realization (Interface impl) | `block` | `0` | `dashed=1` |
| `<<include>>` / `<<extend>>` | `open` | `0` | `dashed=1` |

Full style example for inheritance:
```
endArrow=block;endFill=0;edgeStyle=orthogonalEdgeStyle;
```

Full style example for composition:
```
endArrow=ERmandOne;startArrow=ERmanyToOne;endFill=1;startFill=0;
```

### Sequence diagram arrows

| Message type | Style |
|---|---|
| Synchronous call | `endArrow=block;endFill=1;` solid line |
| Asynchronous | `endArrow=open;` solid line |
| Return | `endArrow=open;dashed=1;` |
| Create | `endArrow=open;dashed=1;` |
| Destroy | `endArrow=block;` to destruction X |

---

## 7. Labels and HTML

When `html=1` in the style, the `value` attribute can contain HTML:

```
&lt;b&gt;ClassName&lt;/b&gt;     → bold class name
&lt;hr/&gt;                      → horizontal divider
- attribute: Type&lt;br/&gt;    → attribute line
+ method(): ReturnType           → method line
```

Visibility prefixes (UML convention):
- `+` public
- `-` private
- `#` protected
- `~` package

Static members: underline with `&lt;u&gt;...&lt;/u&gt;`
Abstract members: italics with `&lt;i&gt;...&lt;/i&gt;`

---

## 8. Geometry

```xml
<mxGeometry x="100" y="200" width="160" height="90" as="geometry" />
```

- Coordinates are in pixels from the top-left of the canvas
- `relative="1"` on edge geometry means position is proportional (0–1)
- `<mxPoint>` inside geometry: absolute waypoints for edge routing

---

## 9. Compressed vs Uncompressed

If the user's XML looks like:
```xml
<mxGraphModel>
  <root>
    <mxCell id="0"/>
    <mxCell id="1" parent="0"/>
    <mxCell id="2" value="7ZRRb..." .../>
  </root>
</mxGraphModel>
```
...or if `compressed="true"`, the actual diagram is base64+deflate encoded inside the cell value or the model body.

**Tell the user:** Go to draw.io → File → Properties (or Extras → Edit Diagram) → uncheck "Compress XML" → OK → re-export or re-copy.

You cannot reliably parse compressed XML without decoding it first.
