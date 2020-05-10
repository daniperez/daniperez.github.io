---
layout: post
title: "CMake-enabled libraries (II)"
date: 2016-02-25 08:00:00 +0200
# categories: 
tags: [cmake, external projects, multi-project, multi-library, add_subdirectories]
---

In this post I analyze the drawbacks of using external projects to implement multi-library projects.

**Note:** A [previous article (I)](https://coderwall.com/p/y3zzbq) and a [follow-up to this one (III)](https://coderwall.com/p/qej45g) exist.

<hr/>

In the [previous article](https://coderwall.com/p/y3zzbq) on this subject I showed a way of implementing multi-library projects by means of ```ExternalProject_Add``` and CMake export registry. The solution allowed us to include third-party library sources as if they were installed in the system (i.e. with ```find_package```). The solutions comes no free of drawbacks. After using this implementation for a while, I found some disappointing effects.

Drawbacks
========
The B and C sub-libraries (see the [previous article](https://coderwall.com/p/y3zzbq)) are built as external projects. External projects are **configured** at execution time. Indeed, CMake workflow consists roughly of 2 steps: the configuration time and the execution time. Configuration time is when ```cmake``` processes our ```CMakeLists.txt```. Execution time is the time we run ```make``` or whatever the compilation tool we chose. As a result of that, what is explained in my [previous article](https://coderwall.com/p/y3zzbq) won't work as expected:

* **Cannot reference targets in the sub-libraries**: this is something we already knew and it's a benefit/drawback of using external projects. We are obliged to use [configuration files](http://www.itk.org/Wiki/CMake/Tutorials/How_to_create_a_ProjectConfig.cmake_file), which leads me to my next point.
<br>
* **Configuration files (```*-config.cmake```) of B and C are not written as A is being configured**: indeed, the fact that external projects are configured at execution time, makes that configuration files of B and C won't be written at A's configuration time, resulting in ```find_package(A)``` and ```find_package(B)``` not working.
<br>

Solution
======

Assuming the 1st problem cannot be solved, in order to solve the second, we can do two things (both [commented in this post](http://www.cmake.org/pipermail/cmake/2012-June/050930.html)): one solution is to implement A yet as another external project, sometimes referred as  the "superbuild" solution since our top-most ```CMakeLists.txt``` doesn't do anything but orchestrating several external projects, including our A root project:


      CMakeLists.txt // "superbuild" make list
      B/
        CMakeLists.txt
      C/
        CMakeLists.txt  // depends on B
      A/
        CMakeLists.txt  // depends on B and C

We have only to ensure, in the root ```CMakeLists.txt```, that A is compiled after C, and C in turn after B:

    ExternalProject_Add (  B ... )
    ExternalProject_Add (  C ...  DEPENDS B )
    ExternalProject_Add (  A ...  DEPENDS B C )

Another solution is to keep A as the root project and make use of  [imported libraries](http://www.cmake.org/Wiki/CMake/Tutorials/Exporting_and_Importing_Targets#Importing_Targets), provided we know where B and C are created. [```ExternalProject_Get_Property```](http://cmake.org/cmake/help/v2.8.11/cmake.html#module:ExternalProject) give us some clues but eventually we'll have to hard-code the path to them. This solution is also depicted [in the previous post](http://www.cmake.org/pipermail/cmake/2012-June/050930.html) and [here](http://stackoverflow.com/questions/15175318/cmake-how-to-build-external-projects-and-include-their-targets). I don't like the solution that much since it's a bit *hacky*, but I must admit it's simple.

Yeah... a bit disappointing. How do you deal with multi-library projects? Let me know in the comments! I hope this protip was useful! 
