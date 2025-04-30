# Understanding Hub Definition in the HubGraph

The HubGraph is an optimization layer that reduces the complexity of pathfinding by focusing only on decision points while preserving the detailed position information through spokes.

## Hub Definition

A vertex in the PositionGraph becomes a hub in the HubGraph if it meets any of these criteria:

1. **Junction points** - Positions with more than two connections
2. **Corner/turning points** - Positions with exactly two connections that aren't opposite each other
3. **Lift positions** - Vertical transport points

## Visual Examples

### Example 1: Straight Path
```
PositionGraph: A---B---C---D---E
HubGraph:      A-----------E
```
- **PositionGraph**: 5 positions connected in a straight line
- **HubGraph**: Only the endpoints A and E are hubs (each has only one connection)
- B, C, and D are not hubs because they each have exactly 2 connections in opposite directions
- The single spoke A→E contains the full position list [A,B,C,D,E]

### Example 2: Corner/Turn
```
    C
    |
A---B---D---E
```
- **PositionGraph**: 5 positions with a branch at B
- **HubGraph**: A, B, C, and E are hubs
- B is a hub because it has 3 connections
- The spokes are A→B, B→C, and B→E
- The spoke B→E contains positions [B,D,E]

### Example 3: Multiple Junctions
```
A---B---C---D
    |       |
    E---F---G---H
        |
        I---J
```
- **PositionGraph**: 10 positions with complex connections
- **HubGraph**: A, B, D, E, F, G, H, I, and J are hubs (B, D, E, F, and G are junctions)
- The spokes include A→B, B→C→D, B→E, E→F, F→G, G→D, G→H, F→I, and I→J

### Example 4: Ring Road
```
A---B---C
|       |
F---E---D
```
- **PositionGraph**: 6 positions forming a ring
- **HubGraph**: All positions are hubs because each has exactly 2 connections that are not opposite
- The spokes are A→B, B→C, C→D, D→E, E→F, and F→A

## Benefits in Real Scenarios

In a warehouse with long straight aisles and occasional intersections:

```
A---B---C---D---E---F---G
                |
H---I---J---K---L---M---N
                |
O---P---Q---R---S---T---U
```

- **PositionGraph**: 21 positions
- **HubGraph**: Only 9 positions (A, G, F, H, L, N, O, S, U) are hubs
- **Pathfinding from A to U**:
  - PositionGraph: Must explore up to 21 vertices
  - HubGraph: Explores just 5 vertices (A→F→L→S→U), then expands to full path

This dramatically reduces pathfinding complexity while maintaining the ability to generate detailed paths when needed.
