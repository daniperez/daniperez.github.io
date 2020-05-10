---
layout: post
title: "CMake-enabled libraries (III)"
date: 2016-04-25 08:00:00 +0200
# categories: 
tags: [cmake, external projects, multi-project, multi-library, add_subdirectories]
---

In this postI explain how our library can be included in other projects without having to install it. We will generate a config file that will work regardless the library is installed or not.

**Note:** read the previous articles for context: [I](https://coderwall.com/p/y3zzbq) and [II](https://coderwall.com/p/qk2eog).
<hr/>

If you read my [previous article](https://coderwall.com/p/y3zzbq) and [its follow-up](https://coderwall.com/p/qk2eog), it should be pretty easy to have a multi-library project up and running,
despite all the boilerplate code. But how we get it to be easy to integrate in other multi-library projects? The [mechanics](http://www.cmake.org/Wiki/CMake/Tutorials/How_to_create_a_ProjectConfig.cmake_file) to allow others to find your library are easy: furnish a file named ```<name>-config.cmake```, ```<name>Config.cmake``` or ```Find<name.cmake``` in your package, set variables in it telling where to find the libraries, headers, etc, and you're done.  But  how to write a config file that is relocatable and works both when you install your library or when you use only its sources?

This is a description of what worked for me. I ommit the CMake code not related to the configuration file generation.

Implementation
==============

I'll assume that the package I'm configuring is called "foo". The paths between angle brackets have to be
replaced by custom paths to the corresponding files. ```foo_LIBRARIES``` has to be defined as well with ```add_library/add_executable```.

First of all we'll make use of [CMakePackageConfigHelpers](http://cmake.org/cmake/help/v2.8.11/cmake.html#module:CMakePackageConfigHelpers):

    include ( CMakePackageConfigHelpers )

This package allows us to write config files that can be relocated, that is to say, where paths are not hard-coded.
Next we can write the config file that will be used if the **package is not installed**:

```cmake
# In my case the folder of includes can be any source folder,
# a practice very extended (in opposition to a single folder
# containing all the headers).
set ( foo_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}" )

# Destination of the installed config files (relative path):
set ( CMAKE_CONFIG_DEST "share/cmake/Modules" )

# We configure our template. The template is described later.
configure_package_config_file (                         
        "<PATH TO THE CONFIG TEMPLATE>/foo-config.cmake.in"
        # Important to write in CMAKE_BINARY_DIR if you want the registry
        # mechanism to work:
        "${CMAKE_BINARY_DIR}/foo-config.cmake"  
        INSTALL_DESTINATION "${CMAKE_CONFIG_DEST}"
        PATH_VARS foo_INCLUDE_DIRS )

# This file is included in our template:
export ( TARGETS foo_LIBRARIES FILE "${CMAKE_BINARY_DIR}/fooTargets.cmake" )

export ( PACKAGE foo ) 
```

To note that I use ```configure_package_config_file``` but [the plain-old configure_file](http://cmake.org/cmake/help/v2.8.11/cmake.html#command:configure_file) would have worked as well here.

So far we have only written the part corresponding to the in-source use of our package, but we still have to write the configuration file that will be distributed and **installed** with our package. The process is similar to the previous one but a bit simpler:

```cmake
# We redefine this variable, using this time a relative path:
set ( foo_INCLUDE_DIRS "include" )

# We write in the 'export' folder in order not to collide with
# the previous config file:
configure_package_config_file ( 
       "<PATH TO THE CONFIG TEMPLATE>/foo-config.cmake.in" 
       "${CMAKE_BINARY_DIR}/export/foo-config.cmake.cmake"
       INSTALL_DESTINATION "${CMAKE_CONFIG_DEST}"
       PATH_VARS foo_INCLUDE_DIRS )

install (
      EXPORT export 
      DESTINATION ${CMAKE_CONFIG_DEST} FILE "fooTargets.cmake" )
```
As you can appreciate, **in-source** and **installed** configurations are symmetric, almost the same, but still we have to use 2 different configuration files.

The only bit missing is the template ```foo-config.cmake.in```:

```cmake
@PACKAGE_INIT@ 
 
set_and_check ( foo_INCLUDE_DIRS "@PACKAGE_foo_INCLUDE_DIRS@")

include ( "${CMAKE_CURRENT_LIST_DIR}/fooTargets.cmake" )
```

```@PACKAGE_INIT@``` will be initialized by ```configure_package_config_file``` and we just need to
set the variables of our project (using ```set_and_check```, defined in ```@PACKAGE_INIT@```) and include our targets.

Conclusion
========

I showed how we can generate two config files, one for in-source usage and another for "installed" usage. Nevertheless the two files use the same template and the creation of both files is almost identical. I hope in the future we can use the same config file for both cases. Also in the future I think we won't need to define our ```*_INCLUDE_DIRS``` variable, making it even simpler.

I hope the protip was useful! How do you generate your config files? Can one do better?

