====================================================
Building oneAPI and the SYCL compiler from Source.
====================================================

oneAPI is a completley open platform and you can build everything from source. Here you will learn how to build each of the components of oneAPI. One of the promises of oneAPI is being able to use all open source components. 

#############################
Build the SYCL DPC++ compiler
#############################

There are many flavors of SYCL compilers that are available. The most popular is the DPC++ compiler that is built from llvm. We'll show you how to build the SYC compiler. 

Currently, this tutorial only supports a Linux distribution. However, it
should work on WSL2. If there are build issues, please file an issue.

*************
Prerequisites
*************

* A Linux distribution that supports distrobox, systemd, podman 
* `Distrobox <https://github.com/89luca89/distrobox/blob/main/docs/README.md>`_

Alternatively, Ubuntu LTS 20.04 is your base OS or already in a container
or running bare metal.


*********
Distrobox
*********

Most distribution should have packaged distrobox - if not, feel free to
install it from https://github.com/89luca89/distrobox on your distribution.

Whether distrobox supports your distro - you can check out distrobox `Host
Distro <https://github.com/89luca89/distrobox/blob/main/docs/compatibility.md#host-distros>`_

These steps were tested on Fedora SilverBlue.

You can complete skip the distrobox method if you are running Ubuntu LTS 20.04.

************************
Setting up the container
************************

Assuming that you have distrobox installed. Let's get started. Pull up a
Linux shell.

Create the container by:

.. code-block:: shell

  $ distrobox create oneapi -i docker.io/library/ubuntu:20.04
  $ distrobox enter oneapi

You will be in a container that is running Ubuntu 20.04.

************************
Build the DPC++ Compiler
************************

You will need to create a workspace. This workspace is where you are going
to have all the SYCL libraries and compiler binaries. You'll be setting up
an LD_LIBRARY_PATH to point at this space.

.. code-block:: shell

  $ mkdir sycl_workspace
  $ cd sycl_workspace
  export DPCPP_HOME=\`pwd\`
  $ git clone https://github.com/intel/llvm -b sycl

Now to configure the build:
.. code-block:: shell

  `$ python $DPCPP_HOME/llvm/buildbot/configure.py`
  `$ python $DPCPP_HOME/llvm/buildbot/compile.py`

Wait till the build is done.

We should have a compiler, but we don't have the basic oneAPI libraries.

We first need to install our low level runtimes: the things that recognizes
accelerators. For now, we'll use the ones that recognize the x86 Intel
processors as that is what is on my laptop right now.

We will first need to identify the latest versions of the
runtimes we need to download. You need to look this up in the
`dependency.conf <https://github.com/intel/llvm/blob/sycl/buildbot/dependency.conf>`_ .

Now to install the libraries:

.. code-block:: shell

    $ sudo mkdir -p /opt/intel
    $ sudo mkdir -p /etc/OpenCL/vendors/intel_fpgaemu.icd
    $ cd /tmp
    $ wget https://github.com/intel/llvm/releases/download/2022-WW50/oclcpuexp-2022.15.12.0.01_rel.tar.gz
    $ wget https://github.com/intel/llvm/releases/download/2022-WW50/fpgaemu-2022.15.12.0.01_rel.tar.gz
    $ sudo bash
    # cd /opt/intel
    # mkdir oclfpgaemu-<fpga_version>
    # cd oclfpgaemu-<fpga_version>
    # tar xvfpz /tmp/fpgaemu-2022.15.12.0.01_rel.tar.gz
    # cd ..
    # mkdir oclcpuexp_<cpu_version>
    # cd oclcpuexp-<cpu_version>
    # tar xvfpz /tmp/oclcpuexp-<cpu_version>
    # cd ..


Now we need to create some configuration files:

.. code-block:: shell

  # pwd
  /opt/intel
  # echo  /opt/intel/oclfpgaemu_<fpga_version>/x64/libintelocl_emu.so > /etc/OpenCL/vendors/intel_fpgaemu.icd
  # echo /opt/intel/oclcpuexp_<cpu_version>/x64/libintelocl.so > /etc/OpenCL/vendors/intel_expcpu.icd

Grab the latest release of oneTBB from github.

.. code-block:: shell

  $ cd /tmp
  $ wget https://github.com/oneapi-src/oneTBB/releases/download/v2021.7.0/oneapi-tbb-2021.7.0-lin.tgz

Extract the tar file into /opt/intel:

.. code-block:: shell

  $ cd /opt/intel
  $ sudo bash
  # tar xvfpz /tmp/oneapi-tbb-2021.7.0-lin.tgz


We'll need to reference some of the libraries in the oneTBB directory in
our build. There will be new versions all the time. In order to keep this
documentation relevant and not have to constantly update - there will be a
'version' placeholder for the actual version.

.. code-block:: shell

    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbb.so /opt/intel/oclfpgaemu_<fpga_version>/x64
    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbbmalloc.so /opt/intel/oclfpgaemu_<fpga_version>/x64
    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbb.so.12 /opt/intel/oclfpgaemu_<fpga_version>/x64
    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbbmalloc.so.2 /opt/intel/oclfpgaemu_<fpga_version>/x64

    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbb.so /opt/intel/oclcpuexp_<cpu_version>/x64
    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbbmalloc.so /opt/intel/oclcpuexp_<cpu_version>/x64
    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbb.so.12 /opt/intel/oclcpuexp_<cpu_version>/x64
    # ln -s /opt/intel/oneapi-tbb-<tbb_version>/lib/intel64/gcc4.8/libtbbmalloc.so.2 /opt/intel/oclcpuexp_<cpu_version>/x64

Now we will configure the library paths:

.. code-block:: shell

  # echo /opt/intel/oclfpgaemu_<fpga_version>/x64 > /etc/ld.so.conf.d/libintelopenclexp.conf
  # echo /opt/intel/oclcpuexp_<cpu_version>/x64 >> /etc/ld.so.conf.d/libintelopenclexp.conf
  # ldconfig -f /etc/ld.so.conf.d/libintelopenclexp.conf

We'll need to test the environment an make sure it passes all tests.

Run this test as yourself not root.

.. code-block:: shell

  $ python $DPCPP_HOME/llvm/buildbot/check.py

If you've come back with no failures then the build environment is correctly
setup.


***************************
Using the build environment
***************************

To use this build environment - here is a test setup that you can use.

.. code-block:: shell

    $ mkdir -p ~/src/simple-oneapi/
    $ cd ~/src/simple-oneapi
    $ export PATH=$DPCPP_HOME/llvm/build/bin:$PATH
    $ export LD_LIBRARY_PATH=$DPCPP_HOME/llvm/build/lib:$LD_LIBRARY_PATH
    $ cat > simple-oneapi.cpp

    #include <sycl/sycl.hpp>

    int main() {
      // Creating buffer of 4 ints to be used inside the kernel code
      sycl::buffer<sycl::cl_int, 1> Buffer(4);

      // Creating SYCL queue
      sycl::queue Queue;

      // Size of index space for kernel
      sycl::range<1> NumOfWorkItems{Buffer.size()};

      // Submitting command group(work) to queue
      Queue.submit([&](sycl::handler &cgh) {
        // Getting write only access to the buffer on a device
        auto Accessor = Buffer.get_access<sycl::access::mode::write>(cgh);
        // Executing kernel
        cgh.parallel_for<class FillBuffer>(
            NumOfWorkItems, [=](sycl::id<1> WIid) {
              // Fill buffer with indexes
              Accessor[WIid] = (sycl::cl_int)WIid.get(0);
            });
      });

      // Getting read only access to the buffer on the host.
      // Implicit barrier waiting for queue to complete the work.
      const auto HostAccessor = Buffer.get_access<sycl::access::mode::read>();

      // Check the results
      bool MismatchFound = false;
      for (size_t I = 0; I < Buffer.size(); ++I) {
        if (HostAccessor[I] != I) {
          std::cout << "The result is incorrect for element: " << I
                    << " , expected: " << I << " , got: " << HostAccessor[I]
                    << std::endl;
          MismatchFound = true;
        }
      }

      if (!MismatchFound) {
        std::cout << "The results are correct!" << std::endl;
      }

      return MismatchFound;
    }

To build this source file:

.. code-block:: shell

  $ clang++ -fsycl simple-sycl-app.cpp -o simple-sycl-app

*********************************************************************
Automatically setting up the container to set up the SYCL environment
*********************************************************************

You will recall that we had to set up some environment variables to make
everything work. Specifically, we needed setup some paths so that you an
find the DPC++ compiler.

If you set up this environment using distrobox instead of a VM/bare metal
install of Ubuntu 20.04.

Use a fresh terminal or exit out of the container.

.. code-block:: shell

  $ uname -a

On a Fedora system you should get:

.. code-block:: shell

  Linux fedora 6.0.13-300.fc37.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 14 16:15:19 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux 

Use the window where you are in the Ubuntu container or enter the container:

.. code-block:: shell

  $ distrobox enter oneapi
  $ uname -a
  Linux oneapi.fedora 6.0.13-300.fc37.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 14 16:15:19 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

You will notice that after "Linux" when you are in the container, there is a "oneapi" prefix to fedora.

We can take advantage of that. Let's make sure that when we enter the
container that we can set things up from the shell perspective to be ready
to write SYCL code.

Exit out of your container or use a fresh terminal and add this to your
.bashrc:

.. code-block:: shell

  if [ $oneapi -gt 0 ]; then
     echo "Initializing oneAPI"
     export DPCPP_HOME="/var/home/your_username/src/dpcplusplus"
     export PATH="$PATH:/var/home/your_username/.local/bin:$DPCPP_HOME/llvm/build/bin"
     export LD_LIBRARY_PATH="$DPCPP_HOME/llvm/build/lib"
  fi

Replace all paths with correct paths set in DPCPP_HOME from the beginning.


**********
Conclusion
**********

You've successfully completed building your own oneAPI environment. Things
didn't work out for you? Please file an issue with details of the OS and
what problem you've encountered.
