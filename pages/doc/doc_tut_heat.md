---
title: Heat equation
permalink: doc_tut_heat.html
sidebar: mydoc_sidebar
tags: [tutorials, pde, objects]
keywords: tutorials, partial differential equation, pde, heat equation, steady
last_updated: August 10, 2016
folder: doc
toc: false
---

Heat equation in the following form is to be solved on a unit domain:

<p> $$\vec{\nabla} \cdot (k \vec{\nabla} T) + \dot q_{o} = 0$$ </p>

exposed to following boundary conditions:

* Top wall:
   $${dT}/{dy}\Big|_{y=1} = {h}/{k}(T - T_{\infty})$$
* Bottom wall:
    $${dT}/{dy}\Big|_{y=0} = 0$$
* Right wall:
    $$T(x=1, y) = 200$$
* Left wall:
    $$T(x=0, y) = 100$$

with the following set of parameters:

$$h = 50  W m^{-2}K^{-1}$$, 
$$k = 2  W m^{-1}K^{-1}$$, 
$$\dot q_o = 5000 W m^{-3}$$, 
$$T_{\infty} = 20 ^{\circ} C$$

{% highlight c++ linenos %}
 // Problem parameters
 auto k = 2.0; auto qdot = 5e3; auto h = 50; auto Tinf = 20;
 // Grid
 Block2* grid = new Block2({0, 0, 0}, {1, 1, 0}, 10, 10); 
 grid->levelHighBound[0] = 2; 
 grid->levelHighBound[1] = 2; 
 grid->addVar("T"); 
 // Variables
 auto T = grid->getVar("T");
 // Linear solver
 T->solver = "BiCGSTAB";
 T->itmax = 1000; 
 T->set(100); 
 // Boundary conditions
 T->setBC("south", "grad", 0); 
 T->setBC("north", "grad", -h/k*Tinf, h/k);
 T->setBC("east", "val", 200); 
 T->setBC("west", "val", 100); 
  
 for (auto i = 0; i< 4; ++i) {
   grid->solBasedAdapt2(grid->getError2(T), 2e-3, 2e-1);
   grid->adapt();  
    
   // Equation 
   grid->lockBC(T); 	
   T->solve( grid->laplace(k) 
 	      + grid->source(0, qdot) ); 
   grid->unlockBC();
    
   grid->writeVTK("heat"); 
 }
    
 delete(grid); 
{% endhighlight %}

Above code produces the following result:

{% include image.html file="tut/tut-heat01.gif" caption="Heat equation solved starting at 5x5 Cartesian grid and refined based on gradient." %}



