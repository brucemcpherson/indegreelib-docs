# Network & Vertex Attributes Guide

This document explains the network attributes added to each vertex (participant) during response processing when `treatmentAnalysis` is enabled in configuration, detailing how each metric is calculated and what it measures in the network.

---

## Overview of Vertex Network Attributes

When processing survey responses, `Network.js` constructs an undirected graph representation of the network using Cytoscape.js. Each vertex in the graph receives a suite of enriched properties categorized into four main domains:

1. **Treatment Classification**
2. **Centrality Metrics**
3. **Bottleneck & Seed Rankings**
4. **Treatment Proximity Tracking**

---

## 1. Treatment Classification Attributes

These attributes decode the raw numeric `treatment_status` code assigned to each participant in the roster.

| Attribute | Type | Description & Value Mapping |
| :--- | :--- | :--- |
| `treatment_group` | String | Broad intervention category:<br>• `"comparison"` (code `0`)<br>• `"influencer-selected"` (codes `1`, `2`, `3`)<br>• `"random-selected"` (codes `4`, `5`, `6`) |
| `treatment_selected` | Boolean | `true` if selected for active intervention (codes `2`, `3`, `5`, `6`); `false` otherwise (codes `0`, `1`, `4`). |
| `treatment_completed` | Boolean | `true` if participant fully completed treatment/intervention (codes `3`, `6`); `false` otherwise. |

---

## 2. Centrality Metrics

Centrality metrics quantify the structural importance, reachability, and influence of individual participants within the social graph.

### `betweenness`
* **Definition**: Measures how frequently a node acts as a bridge along the shortest path between all other pairs of nodes in the graph.
* **Calculation**:
  $$\text{betweenness}(v) = \frac{\text{raw\_betweenness}(v)}{\frac{(n-1)(n-2)}{2}}$$
  Where $n$ is the total number of nodes in the Cytoscape graph.
* **Interpretation**: Values range from `0.0` to `1.0`. A high betweenness score indicates a key "chokepoint" or "broker" who connects otherwise disparate groups or cliques.

### `closeness`
* **Definition**: Measures how close a node is to all other nodes in the network, calculated as the inverse sum of shortest path distances to every other node.
* **Calculation**: Computed using Cytoscape's normalized closeness centrality algorithm:
  $$\text{closeness}(v) = \frac{n - 1}{\sum_{u \neq v} d(v, u)}$$
  Where $d(v, u)$ is the shortest path distance between node $v$ and node $u$.
* **Interpretation**: Values range from `0.0` to `1.0`. High closeness indicates that information or behavioral contagion can spread rapidly from this individual to the rest of the network.

### `eigenvector`
* **Definition**: Measures a node's influence by taking into account the influence of its neighbors. Connectedness to highly-connected nodes yields a higher score than connectedness to isolated nodes.
* **Calculation**: Computed using PageRank as a mathematically sound proxy with a damping factor $d = 0.85$:
  $$\text{eigenvector}(v) = \text{PageRank.rank}(v)$$
* **Interpretation**: Reflects overall network power and prestige within the social structure.

---

## 3. Bottleneck & Seed Rankings

These composite attributes identify key candidates for target interventions, distinguishing between bridge bottlenecks and optimal seed spreaders.

### `bottleneck_rank`
* **Definition**: 1-based integer rank of the node based strictly on its `betweenness` centrality score.
* **Calculation**: Vertices are sorted in descending order of `betweenness`. The top bridge node receives rank `1`.
* **Interpretation**: Identifies the primary chokepoints in the network where information flow can easily be controlled or obstructed.

### `base_seed_score`
* **Definition**: An unpenalized composite score evaluating a node's baseline potential as an initial seed for behavioral contagion.
* **Calculation**: Combines speed (Closeness), bridging (Betweenness), and power (Eigenvector) in equal measure:
  $$\text{base\_seed\_score}(v) = \frac{\text{closeness}(v) + \text{betweenness}(v) + \frac{\text{eigenvector}(v)}{\max(\text{eigenvector})}}{3}$$
* **Interpretation**: Ranges from `0.0` to `1.0`. Higher values represent structurally ideal candidates for launching interventions.

### `seed_rank`
* **Definition**: 1-based integer rank assigned via a **Diversified Greedy Selection Algorithm** to prevent clustering seeds in the same local clique.
* **Calculation**:
  1. Initialize seed candidate pool with all vertices.
  2. Iteratively select the candidate with the highest current score.
  3. Apply distance-based redundancy penalties to all remaining candidates relative to already selected seeds:
     - **Direct Neighbor Penalty ($d = 1$)**: Score reduced by `-0.40` (prevents selecting immediate friends).
     - **2-Hop Shared Neighbor Penalty ($d = 2$)**: Score reduced by `-0.15` (prevents selecting shared friend groups).
  4. Assign `seed_rank = 1` to the first chosen node, `seed_rank = 2` to the second, and so on.
* **Interpretation**: Provides a spatially distributed set of seed participants across distinct network clusters to maximize total contagion reach while minimizing overlap.

---

## 4. Treatment Proximity Tracking

These attributes measure how close each non-treated participant is to the nearest participant who completed treatment.

| Attribute | Type | Calculation & Meaning |
| :--- | :--- | :--- |
| `treated_distance` | Integer | Shortest path hop distance to the nearest vertex where `treatment_completed == true`.<br>• `0`: Self (the node itself completed treatment).<br>• `1`: Direct friend/connection completed treatment.<br>• `2+`: Indirect multi-hop distance to completed treatment.<br>• `-1`: Disconnected / unreachable from any completed treatment node. |
| `treated_id` | String / Number | Vertex ID of the specific target participant who completed treatment providing the shortest path (`null` if unreachable). |

---

## Summary Matrix

| Attribute Name | Category | Primary Use Case |
| :--- | :--- | :--- |
| `treatment_group` | Classification | Segmenting comparison vs active intervention cohorts |
| `treatment_selected` | Classification | Filtering active intervention targets |
| `treatment_completed` | Classification | Identifying active sources of contagion |
| `betweenness` | Centrality | Identifying structural bridges |
| `closeness` | Centrality | Measuring dissemination speed |
| `eigenvector` | Centrality | Identifying prestigious core nodes |
| `bottleneck_rank` | Ranking | Ranking chokepoints for targeted moderation |
| `base_seed_score` | Ranking | Raw composite quality metric for seeds |
| `seed_rank` | Ranking | Optimally picking non-overlapping seed nodes for interventions |
| `treated_distance` | Contagion | Measuring exposure level to intervention across phases |
| `treated_id` | Contagion | Tracing the origin node of intervention exposure |
