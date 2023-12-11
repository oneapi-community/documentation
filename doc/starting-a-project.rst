====================================================
Starting a project a oneAPI project
====================================================

oneAPI is a vast ecosystem where you can build so many different kind of projects. If you look at  `Awesome oneAPI <https://github.com/oneapi-community/awesome-oneapi/README.md>`_ you find many great examples with code on what you can do with oneAPI and the various toolkits. oneAPI runs the gambit from physics simulations to ray tracing to video editing. oneAPI helps you enable to use all your hardware not just your CPU.

Awesome oneAPI will provided a curated set of projects that are impactful and useful and follow proper software engineering practices. We also have a web showcase that culls the latest from the internet. You can check out `oneAPI Web Showcase <https://oneapi-community.github.io>`_. It has a more modern experience. 

Do you already bave a project that you're working on? You can easily port your project to oneAPI.

-----------------
Porting from CUDA
-----------------

Already using CUDA? You can port to oneAPI and still continue to use NVIDIA but also other accelerators. Letting those who use your applications use whatever available GPU or CPUs they at their convenience.

There are some great tools that will help you with porting from CUDA to SYCL. You can use the great `SYCLomatic <https://github.com/oneapi-src/SYCLomatic>`_ tool to automatically port from CUDA to SYCL. There are of course some caveats as there not always direct 1:1 mappings especially if you use the tool. You can find many guides on web search to help you figure out how to port your codebase to SYCL.

---------------
Porting to SYCL
---------------

`SYCL <https://www.khronos.org/sycl/>`_ is based on C++ - if your project already is in C++ then your toolchain won't change too much and should be able to make the pre-requisites to do the porting. If you have other languges that might be a harder lift and requires some method of encapsulating and embedding SYCL code that you can invoke. If your code base is written in Rust for instance then you should be able to build a library that encapsulates the functions you won't to offload to an accelerator and then call it.

---------------------
Starting from Scratch
---------------------

There isn't a guide to start a project from scratch yet. This is something that community can help by providing the kind of documentation that will help you. Like everything, how you start depends on what you want to do. There are paths for high performance computing, deep and machine learning, and various others.

To use SYCL, you need to figure out which version to use SYCL. One of the wonderful things about an open ecosystem is that there are many choices. Today, the best choices when it comes to using SYCL are:

* `AdaptiveCPP <https://github.com/AdaptiveCpp/AdaptiveCpp>`_ - this is the community version of SYCL.
  `DPC++ <https://github.com/intel/llvm>`_ Intel's open source implementation of SYCL.

If you're interested in AI for instance, the easiest way to do oneAPI is to use an AI framework that already uses oneAPI. `pyTorch <https://pytorch.org>`_ in combination with the `Intel extension for PyTorch <https://www.intel.com/content/www/us/en/developer/tools/oneapi/optimization-for-pytorch.html>`_ will let you use pyTorch with oneAPI underneath. Your porting is simply to use the Intel's pyTorch extension to realize the benefits of oneAPI.

