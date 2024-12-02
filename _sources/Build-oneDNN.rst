===========================
Building oneDNN from source
===========================

This page will help you build the oneDNN repository from source.

#############
Prerequisites
#############

You will need to set up your build environment. Please read [Build with
DPCPlusPlus](Build-with-DPCPlusPlus.md) to set up the clang compiler. It
will be required for you to build each of the libraries. The instructions
assumes that you are either building into a ubuntu 20.04 container,
bare metal or in a VM.

The instructions assume a build for CPUs only and not GPUs.


###############
Building oneDNN
###############

Building oneDNN is fairly straight forward. Start with cloning the repository:

.. code-block:: shell

  git clone https://github.com/oneapi-src/oneDNN

once the clone completes, you will need to create a build directory.

.. code-block:: shell

  $ mkdir build
  $ cd build

Next we will need to make sure that we have all the pre-requisites installed to do the
build. At this point, we should have already built the pre-requisites. The only one is
making sure that CMake and make is installed. These should already be installed if you
followed the clang++ build setup.

.. code-block:: shell

  $ sudo apt-get install cmake

Now you can build oneDNN. You should still be inside the 'build' sub-directory. Now
execute:

.. code-block:: shell

  $ cmake ..

Once that command is complete. Type:

.. code-block:: shell

  $ make

The build should complete without any issues. Please file an issue if this is not the
case.

#####################
Testing that it works
#####################

run 

.. code-block:: shell

  $ make test 

To test that everything passes. 

You've successfully completed a build of oneDNN.


############
Using oneDNN
############

Look at the example code to see how to use it. You'll need to set the appropriate library paths so that the compiler can find the libraries. You can find many code examples `here <https://github.com/oneapi-src/oneDNN/tree/master/examples>`_. 
