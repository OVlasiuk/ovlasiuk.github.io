---
title: "Code snippets"
permalink: /_pages/code/
layout: splashmath
date: 2017-11-29T17:02:20-05:00
---
# BRieszk
/ˈbrisk/ is a selection of routines helpful in dealing with discrete measures
converging to a continunous density. They will compute near-minimizers of
varying quality for the [Riesz energy][1]-styled functionals. Several elementary
underlying sets are built-in; general (well-behaved) implicit surfaces are
supported. Proceed with care. 

Below, for example, are 40,000 points on the *tangle* 

$$x^2 (x^2 - 5) + y^2 (y^2 - 5) + z^2 (z^2 - 5) + 11 = 0,  $$

distributed according to the absolute value of its Gaussian curvature.

Left: (approximate) Delaunay triangulation with faces color-coded by the
value (not the absolute value) of curvature, blue-orange/low-high; right:
(approximate) Voronoi diagram with faces colored by the number of edges,
blue-yellow/few-many (hexagons are the majority). Here the approximation is
understood in the sense that both the Delaunay and Voronoi are computed locally
in the tangent plane, then transplanted back to the surface. Notice also that
the goal is never to reach an exact local minimizer of the respective energy,
rather to emulate certain features of such minimizers. Accordingly, no Hessians
are computed and no careful line searches performed. In fact, it is rather
remarkable that these points aren't terrible on the energy scale; proving a
theorem about this is left as an exercise to the reader for now.

The Github repo for BRieszk is [here][2].


![Tangle 1]({{ "/assets/images/tangle40k_1.png" | absolute_url}})|![Tangle 2]({{ "/assets/images/tangle40k_2.png" | absolute_url}})

# 3dRBFnodes
Along the same lines, one can produce reasonable nodes for RBF-functions and use
the latter for, say, geomodelling. A reasonably scalable implementation can be
found below; it handles up to 3M nodes in 3d on an i5 CPU laptop. There is nonempty
code overlap with BRieszk. A detailed discussion is contained in the
[paper][3].

The Github repo for 3dRBFnodes is [here][4]. Below are two pictures showing
Andes, the one on the right is color-coded by elevation.

![Andes 1]({{ "https://raw.githubusercontent.com/OVlasiuk/3dRBFnodes/master/img/andes.png" }})|![Andes 2]({{ "https://raw.githubusercontent.com/OVlasiuk/3dRBFnodes/master/img/andes_scatter.png" }})

[1]: https://en.wikipedia.org/wiki/Poppy-seed_bagel_theorem
[2]: https://github.com/OVlasiuk/BRieszk
[3]: https://arxiv.org/abs/1710.05011
[4]: https://github.com/OVlasiuk/3dRBFnodes
