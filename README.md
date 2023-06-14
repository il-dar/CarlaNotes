# CarlaNotes

Install Notes for Carla and Ubuntu 20.04

Currently Carla build documentation for Linux is written for Ubunti 18.04

There can be different reasons for wanting to use 20.04, in my case there was an issue with NVIDIA drivers not working well with the on board AMD card, 
which was sorted out in kernels available on 20.04.

For getting nvidia to work with on a laptop with AMD Ryzen 7 and NVIDIA GeForce GTC 1660 Ti
you might bump into an issue with the NVIDIA X Server Settings window opening, but not displaying any information,
or giving options of settings to look at on the left.

In my case this was an issue with the headers, if you are facing similar issues try

install:
$sudo apt-get install linux-headers-$(uname -r)

check:
$apt list --installed | grep 'linux-headers-'

$dkms status



One of the main issue's with using 20.04 over 18.04 for Carla is the lack of support for clang-8. The initial part of the install doc actually has one section 
regarding a 20.04 install for the requirements

Ubuntu 20.04.

sudo apt-add-repository "deb http://apt.llvm.org/focal/ llvm-toolchain-focal main"
sudo apt-get install build-essential clang-10 lld-10 g++-7 cmake ninja-build libvulkan1 python python-dev python3-dev python3-pip libpng-dev libtiff5-dev libjpeg-dev tzdata sed curl unzip autoconf libtool rsync libxml2-dev git
sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/lib/llvm-10/bin/clang++ 180 &&
sudo update-alternatives --install /usr/bin/clang clang /usr/lib/llvm-10/bin/clang 180

There is nothing 20.04 specific after that, and you will bump into issues as all the build scripts will be looking for clang-8, which is hard coded in.
Luckily there is a change that was done previously where an upgrade was done for the move from clang-7 to clang-8, so we can just manually do a similar
change on the files.

https://github.com/carla-simulator/carla/commit/457b63b85ef9227f36f7a3dee07da34c1c8d9276#diff-6fab226819b7750992e4832abd67a0a5

There was also an error that I bumped into when trying to build the PythonAPI

"carla/LibCarla/source/test/common/test_streaming.cpp"

Solution was found here:

https://github.com/carla-simulator/carla/issues/2416

in LibCarla/source/test/common/test_streaming.cpp

Solved with a few modifications :
Line 58 and 91, replace "Server" by "carla::streaming::low_level::Server"
Line 63 and 94, replace "Client" by "carla::streaming::low_level::Client"

i.e.

58   carla::streaming::low_level::Server<tcp::Server> srv(io.service, TESTING_PORT);
94  carla::streaming::low_level::Server<tcp::Server> srv(io.service, TESTING_PORT);

Even after making all the changed in documents replacing clang-8 references with clang-10 I was getting some issue where
the build script was spitting out an error where it could not find clang-8. "Could not find compiler set in environment variable CC:    /usr/bin/clang-8."


this was solved with creating a linked file for clang 10 that would be accessed when program was looking for the non existent clang-8

$sudo ln -s /usr/bin/clang-10 /usr/bin/clang-8

$sudo ln -s /usr/bin/clang++-10 /usr/bin/clang++-8

I ran the make PythonAPI command with an argument to specifically use python 3.8

$make PythonAPI ARGS="--python-version=3
