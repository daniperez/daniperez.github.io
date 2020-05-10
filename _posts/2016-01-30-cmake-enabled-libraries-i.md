---
layout: post
title: "CMake-enabled libraries (I)"
date: 2016-01-30 08:00:00 +0200
# categories: 
tags: [cmake, external projects, multi-project, multi-library, add_subdirectories]
---

In this post I explain how to re-use CMake-enabled libraries within another CMake-enabled project when the former aren't installed in your system but in Github or in a subdirectory.

**Note**: A couple of follow-ups to this solution exist: [II](https://coderwall.com/p/qk2eog) and [III](https://coderwall.com/p/qej45g).

<hr/>

The problem
=========
Very often I make use of CMake-enabled libraries of mine in new projects. Very often as well, those libraries are not installed in my system but somewhere in Github or Bitbucket.
    
Even if the libraries are not installed, CMake makes it easy to use them: you copy the library into a folder in the project source tree (or use Git Submodules or whatever) and then use `add_subdirectories` to include the project. Consider for example this sample project A with libraries B and C, where C depends on B. The resulting source tree will be something like this:
    
        A/
          CMakeLists.txt
          B/
            CMakeLists.txt
          C/
            CMakeLists.txt  // depends on B
    
Where A's CMakeLists.txt would have `add_subdirectories(B)` and `add_subdirectories(C)`. Nevertheless this approach has a few drawbacks. Notably, C would probably try to find B by means of `find_package(B)` and this won't work out of the box for a couple of reasons:    

* `find_package(B)` in C's **CMakeLists.txt won't find B unless you provide some hints about where** to look up (e.g. pointing `CMAKE_MODULE_PATH` to B's configuration file). This implies pointing C to A somehow (ugly and in the worst case not possible if we don't own the module) or setting `CMAKE_MODULE_PATH` in A, which could be an acceptable solution.
    
* If eventually we get C to find B, **CMake will complain about a "duplicated target"** coming from B. The problem arises because B will define targets that later on are "imported" again by C. This happens because A, B and C are sharing the same "namespace" with regards to variables and targets. This problem can be circumvented but it requires changing C which, as aforementioned, is not desirable.
    
The solution
=========
    
After trying several solutions, the one that works for me is the following:
    
* **Step 1:** Use `ExternalProject_Add` instead of `add_subdirectories`.
* **Step 2:** Use CMake's `install(EXPORT)` feature to export B's configuration.
* **Step 3:** Use CMake's `export(PACKAGE)` to add B's configuration to the "package registry" where C can later on find it.
    
This procedure is clean in the sense that we don't have to modify C in order to find B. The "package registry" is one of the places where CMake looks for package configuration files. Nevertheless, we still have to modify B or any project we depend upon in order to get its configuration file exported, which is, generally speaking, a good practice.
    
Step 1: Add external projects
----------------------------
    
In A's CMakeLists.txt we replace `add_subdirectories` calls by:

```cmake
 include(ExternalProject)

 ExternalProject_Add (
        B
       GIT_REPOSITORY "<git_repo_url>"
       GIT_TAG "<git_tag_commit_or_branch>"
       UPDATE_COMMAND ""
       INSTALL_COMMAND "" )

 ExternalProject_Add (
        C
        GIT_REPOSITORY "<git_repo_url>"
        GIT_TAG "<git_tag_commit_or_branch>"
        UPDATE_COMMAND ""
        INSTALL_COMMAND "" )
```

You can also use SVN or CVS, or include the sources right in your source tree and then point `SOURCE_DIR` parameter to them (have a look at [ExternalProject_Add](http://cmake.org/cmake/help/v2.8.9/cmake.html#module:ExternalProject)).

`UPDATE_COMMAND` and `INSTALL_COMMAND` lines will avoid using the default update and install procedures, which are usually not needed.

Step 2: Export a configuration
-------------------------------
    
If we want C be able to see B by means of `find_package`, we must export B's configuration. To achieve that, we do the following in B's CMakeLists.txt:
    
*  **Add targets to a export group**:

 ``` CMake
 install ( TARGETS <targets> ... EXPORT <export_name> )
 ```

* Export the export group (sic). CMake will  **write a configuration file** of the current build tree:

``` CMake
export ( TARGETS <targets> FILE <file> )
```
Use `${CMAKE_BINARY_DIR}/<package name>-config.cmake` or `${CMAKE_BINARY_DIR}/<package name>Config.cmake`, otherwise the configuration file won't be found.


Step 3: Export build tree to the package registry
------------------------------------------------

Add the build tree to CMake's package registry found in ` .cmake/packages/<package>/` in unix systems by default:

``` CMake
export ( PACKAGE <package_name> )
```

Once all these steps are done, in C's CMakeLists.txt, you can do just as if B were a regular package:

``` CMake
find_package ( <package_name> )
```
The previous call implies `find_package`'s "no module" or "config" signature. The `find_package( <package_name> MODULE )` signature doesn't look up the package registry.


Please, don't hesitate to share with me if you found a better way of achieving the same goal!

References
=======

* [Exporting and Importing Targets](http://www.cmake.org/Wiki/CMake/Tutorials/Exporting_and_Importing_Targets)
* [How to create a ProjectConfig.cmake file](http://www.itk.org/Wiki/CMake/Tutorials/How_to_create_a_ProjectConfig.cmake_file)
* [CMake Manual](http://cmake.org/cmake/help/v2.8.9/cmake.html)
