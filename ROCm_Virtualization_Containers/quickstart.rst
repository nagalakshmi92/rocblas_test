
.. _quickstart:

====================
quick start guide
====================



The following instructions assume a fresh/blank machine to be prepared for the ROCm + Docker environment; no additional software has been installed other than the typical stock package updating.

It is my recommendation to install the rocm kernel first. Depending on how distribution release cycles lines up, the rocm kernel is often newer than the stock kernel shipping in most linux distributions. The newer kernel often supports newer AMD hardware better, and stock video resolutions and hardware acceleration performance are typically improved. As of the time of this writing, ROCm officially supports Ubuntu and Fedora Linux distributions. The following asciicast demonstrates updating the kernel on Ubuntu 14.04. More detailed instructions can be found on the Radeon Open Compute website:

* `Installing ROCK kernel <http://rocm-documentation.readthedocs.io/en/latest/Installation_Guide/Installation-Guide.html#ubuntu-install>`_ on Ubuntu

Step 1: Install rocm-kernel
****************************

:: 

  wget -qO - http://packages.amd.com/rocm/apt/debian/rocm.gpg.key | sudo apt-key add -
  sudo sh -c 'echo deb [arch=amd64] http://packages.amd.com/rocm/apt/debian/ trusty main  \
    > /etc/apt/sources.list.d/rocm.list'
  sudo apt-get update && sudo apt-get install rocm-kernel

Make sure to reboot the machine after installing the ROCm kernel package to force the new kernel to load on reboot. You can verify the ROCm kernel is loaded by typing the following command at a prompt:

::

   uname -r

Printed on the screen should be a string that's obviously the new ROCm kernel, such as : 4.6.0-kfd-compute-rocm-rel-1.4-16. In this case, it is plain to see that the rocm compute kernel is based off of the linux 4.6.0 kernel.

Step 2: Install docker
************************

After verifying the new kernel is running, next install the docker engine. Manual instructions to install docker on various distro's can be found on the `docker website <https://docs.docker.com/engine/installation/>`_ , but perhaps the simplest method is to use a bash script available from docker itself. If it's OK in your organization to run a bash script on your machine downloaded from the internet, open a bash prompt and execute the following line:

::

  curl -sSL https://get.docker.com/ | sh


The above script looks at the linux distribution and the installed kernel, and installs docker appropriately. The script will output a warning message on a ROCm platform saying that it does not recognize the rocm kernel; this is normal and can be safely ignored. The script does proper docker installation without recognizing the kernel.

Step 3: Verify/Change the docker device storage driver
*******************************************************
The docker device storage driver manages how docker accesses images and containers. There are many available, and `documentation and thorough descriptions <https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/>`_ on storage driver architecture can be found on the official docker website. It is possible to check which storage driver docker is using by issuing a

::

   sudo docker info

command at the command prompt and looking for the 'Storage Driver: ' output. It is hard to predict what storage driver Docker will choose as default on install, and defaults change over time, but in our experience we have run into a problems with the 'devicemapper' storage driver with large image sizes. The 'devicemapper' storage driver imposes limitations on the maximum size images and containers can be. If you work in a field of 'big data', such as in DNN applications, the 10 GB default limit of 'devicemapper' is limiting. There are two options available if you run into this limit:

1.Switch to a different storage driver
   * AMD recommends using 'overlay2', whose dependencies are met by the ROCm kernel and should be available
       * overlay2 provides for unlimited image size
   * If 'overlay2' is not an option, storage drivers can be chosen at service startup time with the --storage-driver=<name> option
2. If you must stick with 'devicemapper', pass the 'devicemapper' configuration variable --dm.basesize on service startup to increase 	 the potential image maximum

The downside to switching to the 'overlay2' storage driver after creating and working with 'devicemapper' images is that existing images need to be recreated. As such, we recommend verifying that docker be set up using the 'overlay2' storage driver before engaging in significant work.

Step 4a: Build ROCm container using docker CLI
************************************************

 * Clone and build the container

::

   git clone https://github.com/RadeonOpenCompute/ROCm-docker
   cd ROCm-docker
   sudo docker build -t rocm/rocm-terminal rocm-terminal
   sudo docker run -it --rm --device="/dev/kfd" rocm/rocm-terminal
   (optional) Step 4b: Build ROCm container using docker-compose


 * Clone and build the container using `docker-compose <https://docs.docker.com/compose/install/>`_

::

   git clone https://github.com/RadeonOpenCompute/ROCm-docker
   cd ROCm-docker
   sudo docker-compose run --rm rocm

Step 5: Verify successful build of ROCm-docker container
**********************************************************

* Verify a working container-based ROCm software stack
* After step #2 or #3, a bash login prompt to a running docker container should be available
   * hcc --version should display version information of the AMD heterogeneous compiler
* Execute sample application
   * cd /opt/rocm/hsa/sample
   * sudo make
   * ./vector-copy
* Text displaying successful creation of a GPU device, successful kernel compilation and successful shutdown should be printed to    	stdout

