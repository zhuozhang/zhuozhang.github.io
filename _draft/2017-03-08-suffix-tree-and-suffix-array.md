---
layout: post
title: Suffix Tree and Suffix Array
tag: [string matching, algorithm]
---

Suffix trees are tree structures built to facilitate repeated string
matching on the same string $S$ of length $m$.  The McCreight
algorithm is one of the algorithms that construct a suffix tree in
linear space and time.

There are several tricks used by this algorithm to achieve such
complexity.

- Naive tree construction of the explicit expression of the string $S$
   takes $O(m^2)$ storage.  This can be reduced by using the original
   string as reference and using only two indicies to represent a
   substring for the edge/node of the tree.  Since the number of nodes is
   $O(m)$, the space of the tree is $O(m)$.

- Traversal the tree is slow in naive method.  Suffix links are used
  to accelerate this action.  For node with path label string
  $x\alpha$, where $x$ is a single character, the suffix link $s(v)$
  corresponds to the node with path label $\alpha$. This reduced the
  number of comparison by $|\alpha|$, for the addition of a suffix.

The full algorithm goes as follows

    Input: string S of length m

    Output: suffix tree T of S

    1. Increment S with symbol $ to be S$. This end symbol is used
    to make all suffix identifiable in the tree.

    2. Initialize the root node r of T, i.e., parent(r) = r, and s(r)
    = r.  

    3. Initialize first node v0. label(v0) = S, parent(v0) = r.

    4. Iterating i from 1 to m. In iteration i,

        5. b = label(v{i-1}).  

        6. If v{i-1} != r && parent(v{i-1}) == r then 

        7.

        8.


    9.