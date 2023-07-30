---
layout: post
title:  "HackerRank “Roads and Libraries” Solution Using Disjoint Set Union Operations"
date:   2023-07-21
categories: competitive programming
---

![Library](/assets/library.jpg)


Most graph theory problems rest heavily on various implementations of depth-first search (DFS). HackerRank’s problem “Roads and Libraries” is an example of a puzzle where a DFS solution can be complicated and error prone. In this blog post, I will demonstrate how disjoint set union (DSU) operations can be used to answer the problem more simply.

# The Problem
Determine the minimum cost to provide library access to all citizens of HackerLand. There are n cities numbered from 1 to n. Currently there are no libraries and the cities are not connected. Bidirectional roads may be built between any city pair listed in cities. A citizen has access to a library if:

* Their city contains a library.
* They can travel by road from their city to a city containing a library.

# Example 

![HackerRank Graph](/assets/roads_libraries_example.png)

In this problem, we are provided with the following input: the number of cities, the number of possible roads, the cost to build a library, the cost to repair a road, and an adjacency list describing which cities can be connected by roads. Let’s say that the cost to build a road is 2, while the cost to build a library is 3, and the adjacency list of potential connecting roads from each city is [[1, 7], [1, 3], [1, 2], [2, 3 ], [5, 6], [6, 8]]. In this example, our total cost to provide libraries for all the citizens of Hackerland would be 5 * 2 = 10 for the cost of roads, and 3 * 2 = 6 for the cost of libraries, for a total of 16. Notice from the above illustration that one of the roads connecting 1, 2, and 3 forms a cycle and would break our logic. Let’s see how DSU operations can ensure that we find these cycles and prevent them before they factor into our calculations later.

The disjoint-union set data structure, also known as a union-find data structure, is used to efficiently manage disjoint sets and perform operations on them. The primary goal of this data structure is to determine if two elements belong to the same set or not and to efficiently merge sets together. It accomplishes this by representing each set as a tree, where each element points to its parent. Initially, each element is considered to be its own set, forming a forest of singleton trees. Through the “union” operation, two disjoint sets can be merged by attaching one tree’s root to the other. The “find” operation allows us to determine the representative or root of a given element’s set, enabling us to check for set membership.

# Solution

In this solution, I use a Python list (data[]) to represent a tree-like structure, where each node has a parent. Initially, all nodes are set to -1, meaning they will be considered as their own parent. When a city joins a set, we update the parent value for all nodes in that set so that they point to the same root node. This root node represents the set. To check if two cities are in the same set (connected), we find their root nodes using a recursive function. If both cities have the same root node, they are in the same set, and we return 0 to avoid creating a cycle in the graph. This approach efficiently handles connected components and helps us build the correct number of necessary roads. Below is a graphic depiction of how the components for our graph will be constructed.

![Find Union Graph](/assets/find_union_op.gif)

In this implementation I have broken the solution up into three functions, find(), unite(), and solve().

```
def find(x: int):
    global data
    return x if data[x] < 0 else find(data[x])
```

This function is used to find the root (representative) of the set to which a given element x belongs. If an element’s parent is itself (i.e., data[x] < 0), then it is the root of the set. If not, it recursively calls find on the parent until it reaches the root.

```
def unite(x: int, y: int):
    global data
    x = find(x)
    y = find(y)
    if x == y:
        return 0
    if data[x] > data[y]:
        x, y = y, x
    data[x] += data[y]
    data[y] = x
    return 1
```

This function is used to unite (merge) two sets to which elements x and y belong. It first finds the roots of the sets containing x and y using the find function. If the roots are the same, it means x and y are already in the same set, and no further action is needed. Otherwise, it merges the smaller set into the larger set. It does this by updating the size (stored as negative value) of the root and making the root of the smaller set point to the root of the larger set. This ensures that the height of the tree is kept low, optimizing future find operations.

```
def solve():
    n, m, cl, cr = map(int, input().split())
    sz = n
    
    global data
    data = [-1] * 125252
    
    ans = cl * n
    for _ in range(m):
        x, y = map(int, input().split())
        if unite(x, y):
            sz -= 1
            ans = min(ans, cl * sz + cr * (n - sz))
    print(ans)
```
This function is responsible for implementing the main logic to solve our given problem. It reads the input containing the number of queries q, where each query contains information about cities, roads, and their costs. The solve function reads each query and applies the disjoint-set operations to find the minimum cost required to connect all cities.

For each query, the time complexity of this solution is O(m * log(n)), where m is the number of roads, and n is the number of elements (cities). Since each query is independent of the others, the total time complexity for processing all queries is O(q * m * log(n)), where q is the number of queries.

Below I’ve listed some resources that I found helpful in understanding DSU operations. I’m always happy to talk code, so reach out if you have any comments, questions, suggestions, or catch any errors. Happy Hacking!

# Resources
[A gist containing my code.](https://gist.github.com/justgnnr/57adf527f0efa44e8e8311285076a6a3)

[Disjoint-set data structure. Wikipedia.](https://en.wikipedia.org/wiki/Disjoint-set_data_structure#Proof_of_O(m_log*_n)_time_complexity_of_Union-Find)

[Abdul Bari lecture.](https://youtu.be/wU6udHRIkcc)

[Introduction to Disjoint Set Data Structure or Union-Find Algorithm. GeeksforGeeks.](https://www.geeksforgeeks.org/introduction-to-disjoint-set-data-structure-or-union-find-algorithm/)

[Disjoint Set Union (Randomized Algorithm). GeeksforGeeks.](https://www.geeksforgeeks.org/disjoint-set-union-randomized-algorithm/)