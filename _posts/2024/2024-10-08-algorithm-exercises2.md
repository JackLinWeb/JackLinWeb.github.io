---
title: Algorithm Exercises 2
image: /assets/img/default-banner.jpg
author: jack
date: 2024-10-08 21:55:00 +0800
categories: [notes, algorithms]
tags: [algorithms]
math: true
pin: false
---

Prove by induction that if we remove the root of a $k$-th order binomial tree, it results in $k$ binomial trees of the smaller orders. You can only use the definition of $B_k$ . Per the definition, $B_k$ is formed by joining two $B_{k−1}$ trees.

Solution:

Base Case: If we remove the root node of $B_1$, obviously we will get $1$ $B_0$. Verified.

Induction Hypothesis: Assume that if we remove the root node of $B_k$, we will get $k$ binomial trees of the smaller orders.

Induction Step: Prove the statement above holds with $B_{k+1}$.

A $B_{k+1}$ binomial tree is formed by two $B_k$, we denoted them as $B_{k1}$ and $B_{k2}$. By definition, the root of $B_{k1}$ is the same as the root of $B_{k+1}$ and the root of $B_{k2}$ is the right-most child of the root of $B_{k+1}$. Now we remove the root of $B_{k1}$, we will get $k$ binomial trees of orders smaller than $k$ as mentioned in IH. Then we count in $B_{k2}$, we have $k+1$ binomial trees in total. Each order of them is smaller than $k+1$.

Therefore, we get $k+1$ binomial trees of the smaller orders when we remove the root node of $B_{k+1}$. The given statement is true.

---

Given a sorted array $A$ and a value $x$,  
(a) Design an algorithm using heaps to find the $k$ closest elements to $x$ in $A$.  
(b) Determine and explain the time complexity of your algorithm.

Solution:

(a) We first calculate each distance between $x$ and $A_i$ and store each of them as a tuple $(A_i, \lvert x-A_i \rvert)$. Then we construct a min binary heap of these tuples and heapify by the values of $\lvert x-A_i \rvert$. Last, we then perform deleteMin() for $k$ times and return the list of values.

(b) Assume the array $A$ has $n$ elements. The time complexity of the distances array initialization is $O(n)$. THe time complexity of building a min binary heap is $O(n)$. The time complexity of deleteMin() for $k$ times is $O(k\log n)$. So the time complexity is $O(2n + k\log n)≤O(n \log n)$.

Pseudocode:

```csharp
bool compare(tuple1, tuple2):
    return tupleA[1] < tupleB[1]

List<int> findClosest(A, x, k):
    distances = Array<Tuple>(A.size())
    for i = 0 to A.size() - 1:
        distances[i] = (A[i], Math.Abs(x - A[i]))

    H = BuildHeap(distances, compare)
    result = List<int>()
    for i = 0 to k - 1:
        tuple = H.deleteMin()
        result.Add(tuple[0])

    return result
```

---

There are n ropes of lengths $L_1, L_2, ..., L_n$. Your task is to connect them into one rope. The cost to connect two ropes is equal to sum of their lengths. Design a greedy algorithm such that it minimizes the cost of connecting all the ropes. No proof is required.

Solution:

Use min heap to store the lengths of ropes. In each iteration, we pop two ropes from the heap, connect them and push the new rope back to the heap. Repeat this process until there is only one rope left in the heap. The time complexity is $O(n\log n)$. Pseudocode:

```csharp
int minCostToConnectRopes(ropes):
    H = BuildHeap(ropes)
    cost = 0
    while H.size() > 1:
        rope1 = H.deleteMin()
        rope2 = H.deleteMin()
        newRope = rope1 + rope2
        cost += newRope
        H.insert(newRope)

    return cost
```

---

Given a graph $G=(V,E)$ with non-negative edge weights and the shortest path distances $d(s, u)$ from a source vertex $s$ to all other vertices in $G$, but without the shortest path tree, devise a linear time algorithm, to find a shortest path from $s$ to a given vertex $t$. No proof is required.

Solution:

Because we already know the shortest path distances $d$, we can exclude every path from $s$ to $u$ where its distance does not equal to $d(s,u)$ during the process. Starting from $s$, for each neighbor $v$ of currently visited vertex $u$, we only move to $v$ if $d(s,u)+weight(u,v)=d(s,v)$. This ensures we're always on shortest paths. Continue the process until we reach $t$. The time complexity is $O(\lvert V \rvert + \lvert E \rvert)$.

```csharp
List<int> shortestPath(V, E, d, s, t):
    previousVertex = List<int>(V.size())
    visited = List<bool>(V.size(), false)
    visited[s] = true
    Q = Queue<int>()
    Q.push(s)
    while Q.size() > 0:
        u = Q.pop()
        if u == t:
            previousVertex[t] = u
            break
        for v in neighbors(u):
            if d(s, u) + weight(u, v) == d(s, v) and !visited[v]:
                previousVertex[v] = u
                visited[v] = true
                Q.push(v)

    // Construct the path reversely
    path = List<int>()
    u = t
    while u != s:
        path.Add(u)
        u = previousVertex[u]
    return path.Reverse()
```

---

Given a graph $G=(V,E)$ where a set of cities $V$ is connected by a network of roads $E$. Each road/edge has a positive weight, $w(u, v)$ between cities $u$ and $v$. There is a proposal to add a new road to the network. The proposal suggests a list $C$ of candidate pairs of cities between which the new road may be built. Your task is to choose the road that would result in the maximum decrease in the driving distance between given city $s$ and city $t$. Design an efficient algorithm for solving this problem (8 points). No proof is required. Give the runtime complexity of your algorithm (2 points).

Solution:

Use Dijkstra's algorithm to find the shortest path distances from $s$ to all other cities $x$, denoting them by $d_s(x)$. Use Dijkstra's algorithm to find the shortest path distances from $t$ to all other cities $x$, denoting them by $d_t(x)$.

For each candidate road $(u, v)$ in $C$: obtain the shortest path distances of $s⇝u→v⇝t$ and $s⇝v→u⇝t$, which can be calculated by $d_s(u)+w(u,v)+d_t(v)$ and $d_s(v)+w(v,u)+d_t(u)$ respectively. We update the shortest path distance if we find a larger decrease. After all iterations, return the candidate pair that results in the maximum decrease in the driving distance.

Time complexity: Dijkstra's algorithm with binary heap takes $O((\lvert V \rvert+ \lvert E \rvert)\log V)$. Iterating through candidate roads takes $O(2 * \lvert C \rvert)$. So the total time complexity is $O(2(\lvert V \rvert+\lvert E \rvert)\log V + 2 * \lvert C \rvert)=O((\lvert V \rvert+\lvert E \rvert)\log V)$.

Pseudocode:

```csharp
List<int> bestNewRoad(V, E, C, s, t):
    distS = Dijkstra(V, E, s)  // Dijkstra's from s to all vertices
    distT = Dijkstra(V, E, t)  // Dijkstra's from t to all vertices

    origDist = dist_s[t]
    maxDecrease = 0
    bestRoad = null
    foreach (u, v) in C:
        newDist = min(distS[u] + w(u, v) + distT[v], 
                       distS[v] + w(v, u) + distT[u])
        decrease = origDist - newDist
        
        if decrease > maxDecrease:
            maxDecrease = decrease
            bestRoad = (u, v)

    return bestRoad
```

---

Let us say that a graph $G=(V,E)$ is a near tree if it is connected and has most $n + k$ edges, where $n = |V|$ and $k$ is a **constant**. Give an algorithm with running time $O(n)$ that takes a near tree $G$ with costs on its edges, and returns a minimum spanning tree of $G$. Explain the time complexity of your algorithm. You may assume that all edge costs are distinct.  
(Note: $k$ is a **constant**. No need to prove or justify the correctness of the algorithm.)

Solution:

Generate an arbitrary spanning tree $T$ using BFS. We then have $k+1$ remaining edges that are not in $T$. For each remaining edge $(u, v)$: we run DFS to find the path $u⇝v$ from $T$ and find the edge with the largest weight from that path. If the weight of the edge is large than the weight of $(u, v)$, we replace the edge with $(u, v)$.

The initial BFS requires $O(\lvert V \rvert + \lvert E \rvert)$ time. For $(k+1)$ remaining edges, we run DFS which also requires $O(\lvert V \rvert + \lvert E \rvert)$ on each. Therefore the total time complixity is $O((k+2)*(\lvert V \rvert + \lvert E \rvert))$. Because $k+2$ is a constant and $\lvert E \rvert ≤ \lvert V \rvert+k$, the time complexity can be simplified as $O(\lvert V \rvert+\lvert E \rvert)$=$O(\lvert V \rvert+(\lvert V \rvert+k))=O(\lvert V \rvert)$.Pseudocode:

```csharp
List<int> minSpanningTree(V, E):
    T = findSpanningTreeBFS(V, E)
    remainingEdges = E - T
    for (u, v) in remainingEdges:
        edges = findPathDFS(T, u, v)
        maxWeightEdge = edges[0]
        for i = 0 to edges.size() - 1:
            if weight(edges[i]) > weight[maxWeightEdge]:
                maxWeightEdge = edges[i]
        if weight[maxWeightEdge] > weight(u, v):
            T.removeEdge(maxWeightEdge)
            T.addEdge((u, v))
    return T
```

---

In a lawn, there are n lawn sprinklers $S_1, ..., S_n$ installed on the center line of the lawn. Every sprinkler sprays $D$ meters long in the same fixed direction from east to west. The sprinkler $S_1$ is at the east borderline of the lawn. Each sprinkler $S_i (2 ≤ i ≤ n)$ is $d_i ≤ D$ meters away from the previous sprinkler $S_i−1$. The sprinkler $S_n$ is at the west borderline of the lawn. What is the minimum number of sprinklers you must open to cover the lawn from its east side to its west side? Design an algorithm that solves the above optimization problem. (4 points). What is the asymptotic running time of your algorithm in terms of n? (1 points). Prove the correctness of your algorithm. (5 points)

Solution:

Use greedy approach. Denote the distance from $S_1$ to a sprinkler $S_x$ as $f(S_x) = \displaystyle{\sum_{i=1}^x} d_i$. We first open $S_1$. In each iteration, we choose the sprinkler covering farthest distance from the range from $f(S_i)$ to $f(S_i) + D$. Repeat until $S_i$ covers the west borderline. The time complexity is $O(n)$.

Prove the correctness:

Denote the distance from $S_1$ to a sprinkler $S_x$ as $f(S_x) = \displaystyle{\sum_{i=1}^x} d_i$. Let $\{i_1,i_2,...,i_k\}$ be the sprinklers chosen by our greedy approach and $\{j_1,j_2,...,j_m\}$ be the optimal solution. We will prove by induction that $f(i_r) ≥ f(j_r)$ and thus our greedy approach cannot be worse than the optimal solution.

Base Case: $r=1$. The first sprinkler is opened by both our greedy approach and optimal solution is $S_1$. Verified.

Induction Hypothesis: Assume $f(i_r) ≥ f(j_r)$.

Induction Step: Prove $f(i_{r+1}) ≥ f(j_{r+1})$ holds.

From the statement of IH, we know $f(i_r) + D ≥ f(j_r) + D$. Our greedy approach can choose farther sprinklers between $f(j_r) + D$ and $f(i_r) + D$ as $i_{r+1}$, but the optimal solution can't. Therefore, we have $f(i_{r+1}) ≥ f(j_{r+1})$.

Next we prove by contradiction that our greedy approach has the same size as the optimal solution i.e., $k=m$. Assume $k>m$. From our inductive proof, we know that for all $r ≤ m$, $f(i_r) ≥ f(j_r)$. This means when our greedy approach opens sprinkler $i_m$, it already covers at least as much distance as the optimal solution. Contradiction, $k=m$. Thus, the greedy approach is optimal.

Psuedocode:

```csharp
int minSprinklers(distances, D, lawnLength):
    n = len(distances)
    openSprinkler = 0
    currentCoverage = 0
    i = 0
    while currentCoverage < lawnLength:
        maxReach = currentCoverage
        while i < n and distances[i] <= currentCoverage + D:
            maxReach = max(maxReach, distances[i] + D)
            i += 1
        
        openSprinkler += 1
        currentCoverage = maxReach

    return openSprinkler
```

---

Suppose you are to drive from USC to Santa Monica along I-10. Your gas tank, when full, holds enough gas to go $p$ miles, and you have a map that contains the information on the distances between gas stations along the route. Let $d_1 < d_2 < ... < d_n$ be the locations of all the gas stations along the route where $d_i$ is the distance from USC to the $i$-th gas station. We assume that the distance between neighboring gas stations is at most $p$ miles. Your goal is to make as few gas stops as possible along the way. Give the most efficient algorithm to determine at which gas stations you should stop (3 points) and prove that your strategy yields an optimal solution (6 points). Give the time complexity of your algorithm as a function of $n$ (1 points). Suppose the gas stations have already been sorted by their distances to USC.

Solution:

Use gready approach. Always stop at the farthest reachable gas station before we run out of fuel. Fully refill the tank and continue driving, applying the same strategy until we reach the destination. The time complexity is $O(n)$.

Prove the correctness:

Let $\{i_1,i_2,...,i_k\}$ be the gas stations chosen by our greedy approach and $\{j_1,j_2,...,j_m\}$ be the optimal solution. We will prove by induction that $d_{i_r} ≥ d_{j_r}$ and thus our greedy approach cannot be worse than the optimal solution.

Base Case: $r=1$. $d_{i_1} ≥ d_{j_1}$ is true because our greedy approach chooses the farther gas station from $0$ to $p$ miles.

Induction Hypothesis: Assume $d_{i_r} ≥ d_{j_r}$.

Induction Step: Prove $d_{i_{r+1}} ≥ d_{j_{r+1}}$ holds.

From the statement of IH, we know $d_{i_r} + p ≥ d_{j_r} + p$. Our greedy approach can choose farther gas stations between $d_{j_r} + p$ and $d_{i_r} + p$ as $i_{r+1}$, but the optimal solution can't. Therefore, we have $d_{i_{r+1}} ≥ d_{j_{r+1}}$.

Next we prove by contradiction that our greedy approach has the same size as the optimal solution i.e., ${k=m}$. Assume $k>m$. From our inductive proof, we know that for all $r ≤ m$, $d_{i_r} ≥ d_{j_r}$. This means when our greedy approach stops at gas station $i_m$, it already covers at least as much distance as the optimal solution. Contradiction, $k=m$. Thus, the greedy approach is optimal.

Pseudocode:

```csharp
List<int> minGasStops(d, p):
    n = len(d)
    currentPosition = 0
    prevStop = 0
    stops = List<int>()

    for i = 0 to n - 1:
        if d[i] - currentPosition > p:
            stops.Add(prevStop)
            currentPosition = d[prevStop]
        prevStop = i

    return stops
```

---

In a greenhouse, several plants are planted in a row. You can imagine this row as a long line segment. Your task is to water the plants. Each plant in the greenhouse has some minimum amount of water (in $L$) per day to stay alive. Suppose there are $n$ plants in the greenhouse and let the minimum amount of water (in $L$) they need per day be $l_1, l_2, l_3, ..., l_n$. You generally order $n$ water bottles ($1$ bottle for each plant) of $max(l_1, l_2, l_3, ..., l_n)$ capacity per day to water the plants, but due to some logistics issue on a bad day, you received $n$ bottles of different capacities (in $L$). Suppose $c_1, c_2, c_3, ..., c_n$ be the capacities of the water bottles, and you are required to use one bottle completely to water one plant. In other words, you will allocate one bottle per plant, and use the entire water present (even if it is more than the minimum amount of water required for that plant) in that bottle to water a particular plant. You cannot use more than one bottle (or partial amount of water) to water a single plant (You need to use exactly one bottle per plant). Suggest an algorithm to determine whether it is possible to come up with an arrangement such that every plant receives more than or equal to its minimum water requirement. Prove the correctness of your algorithm.

Solution:

Sort both the minimum water requirements $L$ and the water bottle capacities $\{c_1, c_2, ..., c_n\}$ in non-descending orders. Use greedy approach. We iterate through sorted minimum water requirements and match them with the smallest available bottle that can satisfy the requirement. If we can find a bottle for each plant, then it is possible to come up with an arrangement such that every plant receives more than or equal to its minimum water requirement. The time complexity is $O(n\log n)$.

Prove the correctness:

Since we sort both the minimum water requirements and the bottle capacities in non-descending orders, we can always match the smallest available bottle with the plant requiring smallest minimum water. If the smallest available bottle can't satisfy the matched requirement, then the bottle can't satisfy any subsequent non-descending requirements. Otherwise, we can always find a bottle satisfy a plant. Therefore, the greedy approach is correct.

Pseudocode:

```csharp
bool enoughWater(L, C):
    n = len(L)
    L.sort()
    C.sort()
    for i = 0 to n - 1:
        if L[i] > C[i]:
            return false
    return true
```