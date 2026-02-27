# draw.io XML Generation Templates

Use these templates as starting points when generating draw.io XML from descriptions.
Replace placeholders in `ALL_CAPS` with actual values. Increment IDs sequentially.

## Table of Contents
1. [Document envelope](#1-document-envelope)
2. [Class diagram — single class](#2-class-diagram--single-class)
3. [Class diagram — inheritance](#3-class-diagram--inheritance)
4. [Class diagram — association with multiplicity](#4-class-diagram--association-with-multiplicity)
5. [Sequence diagram — basic exchange](#5-sequence-diagram--basic-exchange)
6. [Use case diagram — basic](#6-use-case-diagram--basic)
7. [Activity diagram — decision flow](#7-activity-diagram--decision-flow)
8. [State diagram — basic](#8-state-diagram--basic)
9. [Layout guidelines](#9-layout-guidelines)

---

## 1. Document envelope

Always wrap generated diagrams in this envelope:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mxGraphModel compressed="false" dx="1422" dy="762" grid="1" gridSize="10"
              guides="1" tooltips="1" connect="1" arrows="1" fold="1"
              page="1" pageScale="1" pageWidth="1169" pageHeight="827"
              math="0" shadow="0">
  <root>
    <mxCell id="0" />
    <mxCell id="1" parent="0" />
    <!-- INSERT CELLS HERE (id starts at 2) -->
  </root>
</mxGraphModel>
```

---

## 2. Class diagram — single class

```xml
<!-- CLASS: CLASSNAME at position (X, Y) -->
<mxCell id="2" value="&lt;b&gt;CLASSNAME&lt;/b&gt;"
        style="text;html=1;strokeColor=#000000;fillColor=#dae8fc;
               align=center;verticalAlign=top;spacingLeft=4;spacingRight=4;
               overflow=hidden;rotatable=0;points=[[0,0.5],[1,0.5]];
               portConstraint=eastwest;"
        vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="180" height="30" as="geometry" />
</mxCell>

<!-- ATTRIBUTES compartment -->
<mxCell id="3" value="- ATTR1: TYPE1&lt;br&gt;- ATTR2: TYPE2"
        style="text;html=1;strokeColor=#000000;fillColor=#ffffff;
               align=left;verticalAlign=top;spacingLeft=4;spacingRight=4;
               overflow=hidden;rotatable=0;"
        vertex="1" parent="1">
  <mxGeometry x="100" y="130" width="180" height="50" as="geometry" />
</mxCell>

<!-- OPERATIONS compartment -->
<mxCell id="4" value="+ METHOD1(): RETURN_TYPE&lt;br&gt;+ METHOD2(PARAM: TYPE): void"
        style="text;html=1;strokeColor=#000000;fillColor=#ffffff;
               align=left;verticalAlign=top;spacingLeft=4;spacingRight=4;
               overflow=hidden;rotatable=0;"
        vertex="1" parent="1">
  <mxGeometry x="100" y="180" width="180" height="50" as="geometry" />
</mxCell>
```

**Notes:**
- Use `fillColor=#dae8fc` (blue) for regular classes
- Use `fillColor=#d5e8d4` (green) for interfaces
- Use `fillColor=#fff2cc` (yellow) for enumerations
- Use `fillColor=#f8cecc` (red/pink) for abstract classes
- Use `fillColor=#ffffff` (white) for attribute/method compartments
- Stack compartments vertically: name at Y, attrs at Y+30, methods at Y+30+attrs_height

---

## 3. Class diagram — inheritance

```xml
<!-- PARENT class (id=2,3,4 as above at x=300, y=100) -->
<!-- CHILD class (id=5,6,7 as above at x=300, y=280) -->

<!-- GENERALIZATION arrow (child → parent, hollow triangle) -->
<mxCell id="8" value="" style="endArrow=block;endFill=0;
        edgeStyle=orthogonalEdgeStyle;orthogonalLoop=1;jettySize=auto;
        exitX=0.5;exitY=0;exitDx=0;exitDy=0;
        entryX=0.5;entryY=1;entryDx=0;entryDy=0;"
        edge="1" source="5" target="2" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

**For interface realization** (dashed + hollow triangle):
```xml
<mxCell id="8" value="" style="endArrow=block;endFill=0;dashed=1;
        edgeStyle=orthogonalEdgeStyle;" edge="1" source="CHILD_ID" target="INTERFACE_ID" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

---

## 4. Class diagram — association with multiplicity

```xml
<!-- ASSOCIATION line -->
<mxCell id="10" value="RELATIONSHIP_NAME"
        style="endArrow=open;endFill=0;edgeStyle=orthogonalEdgeStyle;"
        edge="1" source="SOURCE_ID" target="TARGET_ID" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- Multiplicity at SOURCE end (e.g., "1") -->
<mxCell id="11" value="1" style="resizable=0;html=1;align=left;
        verticalAlign=bottom;labelBackgroundColor=none;fontSize=12;"
        vertex="1" connectable="0" parent="10">
  <mxGeometry x="-0.9" relative="1" as="geometry">
    <mxPoint as="offset" />
  </mxGeometry>
</mxCell>

<!-- Multiplicity at TARGET end (e.g., "0..*") -->
<mxCell id="12" value="0..*" style="resizable=0;html=1;align=right;
        verticalAlign=bottom;labelBackgroundColor=none;fontSize=12;"
        vertex="1" connectable="0" parent="10">
  <mxGeometry x="0.9" relative="1" as="geometry">
    <mxPoint as="offset" />
  </mxGeometry>
</mxCell>
```

**Composition** (filled diamond at whole end):
```
style="endArrow=ERmandOne;startArrow=ERmanyToOne;endFill=1;startFill=1;..."
```

**Aggregation** (hollow diamond at whole end):
```
style="endArrow=ERzeroToMany;startArrow=diamond;endFill=0;startFill=0;..."
```

**Dependency** (dashed open arrow):
```
style="endArrow=open;endFill=0;dashed=1;..."
```

---

## 5. Sequence diagram — basic exchange

```xml
<!-- ACTOR (optional — for human actors) -->
<mxCell id="2" value="ACTOR_NAME"
        style="shape=mxgraph.uml.actor;whiteSpace=wrap;html=1;"
        vertex="1" parent="1">
  <mxGeometry x="60" y="40" width="30" height="60" as="geometry" />
</mxCell>

<!-- PARTICIPANT box -->
<mxCell id="3" value="OBJECT : CLASS"
        style="text;html=1;strokeColor=#000000;fillColor=#dae8fc;
               align=center;verticalAlign=middle;whiteSpace=wrap;"
        vertex="1" parent="1">
  <mxGeometry x="200" y="40" width="160" height="40" as="geometry" />
</mxCell>

<!-- LIFELINE (dashed vertical line) -->
<mxCell id="4" value=""
        style="endArrow=none;dashed=1;edgeStyle=orthogonalEdgeStyle;"
        edge="1" source="3" target="5" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- LIFELINE endpoint (invisible anchor at bottom) -->
<mxCell id="5" value="" style="point;x=0;y=0;perimeter=0;" vertex="1" parent="1">
  <mxGeometry x="280" y="600" as="geometry" />
</mxCell>

<!-- ACTIVATION BAR -->
<mxCell id="6" value=""
        style="strokeColor=#000000;fillColor=#ffffff;"
        vertex="1" parent="1">
  <mxGeometry x="271" y="120" width="18" height="80" as="geometry" />
</mxCell>

<!-- SYNCHRONOUS MESSAGE (filled arrowhead) -->
<mxCell id="7" value="methodName(args)"
        style="endArrow=block;endFill=1;edgeStyle=orthogonalEdgeStyle;html=1;"
        edge="1" source="SENDER_ACTIVATION_ID" target="RECEIVER_ACTIVATION_ID" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- RETURN MESSAGE (dashed) -->
<mxCell id="8" value=":ReturnValue"
        style="endArrow=open;endFill=0;dashed=1;edgeStyle=orthogonalEdgeStyle;html=1;"
        edge="1" source="RECEIVER_ACTIVATION_ID" target="SENDER_ACTIVATION_ID" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

**Layout tip:** Place participants at y=40–60. Space them 200px apart horizontally. Lifelines extend to y=600. Messages are horizontal arrows connecting activation bars.

---

## 6. Use case diagram — basic

```xml
<!-- SYSTEM BOUNDARY -->
<mxCell id="2" value="SYSTEM_NAME"
        style="swimlane;startSize=20;fillColor=#f5f5f5;strokeColor=#666666;
               fontColor=#333333;fontSize=14;fontStyle=1;"
        vertex="1" parent="1">
  <mxGeometry x="160" y="60" width="500" height="400" as="geometry" />
</mxCell>

<!-- ACTOR (outside boundary) -->
<mxCell id="3" value="ACTOR_NAME"
        style="shape=mxgraph.uml.actor;whiteSpace=wrap;html=1;"
        vertex="1" parent="1">
  <mxGeometry x="60" y="200" width="40" height="80" as="geometry" />
</mxCell>

<!-- USE CASE (inside boundary, parent="2") -->
<mxCell id="4" value="USE_CASE_NAME"
        style="ellipse;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;"
        vertex="1" parent="2">
  <mxGeometry x="120" y="100" width="160" height="60" as="geometry" />
</mxCell>

<!-- ASSOCIATION (actor to use case) -->
<mxCell id="5" value=""
        style="endArrow=none;edgeStyle=orthogonalEdgeStyle;"
        edge="1" source="3" target="4" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- «include» relationship -->
<mxCell id="6" value="«include»"
        style="endArrow=open;endFill=0;dashed=1;edgeStyle=orthogonalEdgeStyle;"
        edge="1" source="BASE_UC_ID" target="INCLUDED_UC_ID" parent="2">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- «extend» relationship -->
<mxCell id="7" value="«extend»"
        style="endArrow=open;endFill=0;dashed=1;edgeStyle=orthogonalEdgeStyle;"
        edge="1" source="EXTENDING_UC_ID" target="BASE_UC_ID" parent="2">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

---

## 7. Activity diagram — decision flow

```xml
<!-- INITIAL NODE -->
<mxCell id="2" value=""
        style="ellipse;aspect=fixed;fillColor=#000000;strokeColor=#000000;"
        vertex="1" parent="1">
  <mxGeometry x="240" y="40" width="20" height="20" as="geometry" />
</mxCell>

<!-- ACTION -->
<mxCell id="3" value="ACTION_LABEL"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;"
        vertex="1" parent="1">
  <mxGeometry x="160" y="100" width="180" height="50" as="geometry" />
</mxCell>

<!-- DECISION -->
<mxCell id="4" value="CONDITION?"
        style="rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;"
        vertex="1" parent="1">
  <mxGeometry x="185" y="200" width="130" height="70" as="geometry" />
</mxCell>

<!-- FLOW edge (with guard label) -->
<mxCell id="5" value="[Yes]"
        style="endArrow=block;endFill=1;edgeStyle=orthogonalEdgeStyle;"
        edge="1" source="4" target="ACTION2_ID" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- ACTIVITY FINAL NODE -->
<mxCell id="8" value=""
        style="ellipse;aspect=fixed;fillColor=#000000;strokeColor=#000000;
               double=1;"
        vertex="1" parent="1">
  <mxGeometry x="237" y="500" width="26" height="26" as="geometry" />
</mxCell>
```

---

## 8. State diagram — basic

```xml
<!-- INITIAL PSEUDOSTATE -->
<mxCell id="2" value=""
        style="ellipse;aspect=fixed;fillColor=#000000;strokeColor=#000000;"
        vertex="1" parent="1">
  <mxGeometry x="245" y="40" width="20" height="20" as="geometry" />
</mxCell>

<!-- STATE -->
<mxCell id="3" value="STATE_NAME"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;
               arcSize=30;"
        vertex="1" parent="1">
  <mxGeometry x="160" y="100" width="180" height="60" as="geometry" />
</mxCell>

<!-- TRANSITION -->
<mxCell id="5" value="event [guard] / action"
        style="endArrow=block;endFill=1;edgeStyle=orthogonalEdgeStyle;"
        edge="1" source="SOURCE_STATE_ID" target="TARGET_STATE_ID" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- FINAL STATE -->
<mxCell id="6" value=""
        style="ellipse;aspect=fixed;fillColor=#000000;strokeColor=#000000;
               double=1;"
        vertex="1" parent="1">
  <mxGeometry x="237" y="400" width="26" height="26" as="geometry" />
</mxCell>
```

---

## 9. Layout guidelines

### Spacing
- Class boxes: 200px horizontal gap, 150px vertical gap between boxes
- Sequence lifelines: 200–250px apart horizontally; messages at 60px vertical intervals
- Use case ellipses: 180px wide × 60px tall; 40px gap between them
- Activity nodes: 180px wide × 50px tall; 80px vertical gap between them

### Positioning strategy
- **Class diagrams**: Parent classes above children; interfaces top-left
- **Sequence diagrams**: Left to right by call order; initiating actor/object at far left
- **Use case diagrams**: Actors on left margin; primary use cases in center; secondary on right
- **Activity diagrams**: Top to bottom; decision branches go left/right, merge back to center
- **State diagrams**: Initial pseudostate top-center; states arranged to minimize crossing transitions

### ID numbering
- IDs 0 and 1: reserved for root and default layer — never reuse
- Start user content at id="2"; increment by 1 for each cell
- For multiplicity label cells (child of edge), give them the next available ID
- Keep a mental counter as you generate XML

### Color palette (use consistently)
- Regular class: `fillColor=#dae8fc` (light blue)
- Interface: `fillColor=#d5e8d4` (light green)
- Enumeration: `fillColor=#fff2cc` (light yellow)
- Abstract class: `fillColor=#f8cecc` (light red)
- Note/comment: `fillColor=#ffffc0` (light yellow, no stroke)
- Decision/choice: `fillColor=#fff2cc`
- Attribute/method compartments: `fillColor=#ffffff`
- System boundary: `fillColor=#f5f5f5`
