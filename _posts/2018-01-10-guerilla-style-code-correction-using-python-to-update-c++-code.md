---
layout: post
title:  "Guerilla Style Code Correction - Using Python to Update C++ Code"
date:   2018-01-10
excerpt: "How did this actually work?"
image: "/images/posts/2018-01-10/cpppython.jpeg"
published: false
---

In our recent effort to modernize the VMTK build and distribution system I came across a series of issues which reminded me why I so love programming with the ever adaptable Python language. What I thought would be an inconsequential change in our build/packaging system, updating a single line of CMake code `CMAKE_CXX_STANDARD=98` -> `CMAKE_CXX_STANDARD=11`, turned our error free build system into the stuff of compile time nightmares. 

 In C++11, a new keyword was added to the language: `override`. Any time that your code wishes to override a method from a derived class, you are now supposed to include the `override` keyword in order to make your intentions clear to the compiler. That way, if you misstype the name of a derived class, the compiler can issue an error letting you know there is no such method to override. It seems like its a beneficial language feature, and it will probably be very useful to me in the future; however, it also meant that any time our (outdated) code didn't specify that magic `override` keyword, the compiler would kindely let us know by issuing a warning...

 Actually, more like 600+ of them... 


