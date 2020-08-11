# RELEVANCY #
*Works as of: 2020-Aug-11*

## INTRODUCTION ##
This guide is to help you *get AMDGPU-PRO's OpenCL implementation to work on Debian (and probably Ubuntu, Mint, etc. -- but I don't use those, so I make no guarantee on them), without switching off Mesa (because you play games and stuff, of course, right?)*

# REASONING #
This guide exists to help you get at least a base OpenCL implementation on Debian, if you find that ``mesa-opencl-icd`` returns the below BS, when you try to use GPU compute on like Blender, or if it screws up like GIMP and LibreOffice, as I have come across from other users, while perusing something like 36,000 threads during my own trip through this gauntlet...

# THE ERROR THAT DESTROYED MY WEEK AND YOURS #
```
: CommandLine Error: Option 'polly' registered more than once!
LLVM ERROR: inconsistency in registered CommandLine options
```
*"Option 'polly'" is sometimes a different error, according to other threads, but this one seemed the most common, and it's the one I had.*

# HISTORICAL CONTEXT #
I spent literally like 30 freaking hours over the course of a week trying to fix this, and I must've installed and removed that package like 100 times (NOT exaggerating), trying to test different configs. This bug has apparently been on the ``bugs.debian.org`` site, on and off, since 2017 or so.

I'm writing it to save you some freaking time, in case you, too, are experiencing this issue... because I don't wanna be *That Guy*™, who solves their problem and just closes the thread without helping others.

## GET IT GOING ##
# DOWNLOAD THE DRIVERS #
https://www.amd.com/en/support

# FIND THE LIBS #
First, unpack the driver ``.so`` files from the following (64-bit versions -- if you have 32-bit, use the i386 packages, and extract those [the "64" in the filenames won't be there]):
1) ``libamdocl64.so`` from ``opencl-amdgpu-pro-icd_[some number]``
1) ``libamdocl-orca64.so`` and ``libamdocl12cl64.so`` from ``opencl-orca-amdgpu-pro-icd_[some number]``
1) ``libdrm.so.2``, ``libdrm.so.2.4.0``, ``libkms.so.1``, and ``libkms.so.1.0.0`` from ``libdrm2-amdgpu_[some number]``
1) ``libOpenCL.so.1`` and ``libOpenCL.so.1.2`` from ``ocl-icd-libopencl1-amdgpu-pro_[some number]``

*You can extract the libraries using like Ark or something -- that's what I used*
*They will be under the* ``opt`` *sub-directory -- follow it from there, until you reach the* ``.so`` *files.*

# ORGANIZE IT #
Once these have been extracted, place them in a directory of choice (doesn't matter, because you will be building an ICD driver, which is dynamically-linked against them on load, so long as the path is correct).

# DEPENDENCY CHECK #
You will need the ``ocl-icd-libopencl1`` package to load the ICD files you are about to create. Make sure you install it.

# BUILD THE ICD FILES #
Create the following files in the directory ``/etc/OpenCL/vendors/``:
1) ``amdpro.icd`` with the contents: [path/to/drivers]/libamdocl64.so
1) ``amdpro12.icd`` with the contents: [path/to/drivers]/libamdocl12cl64.so
1) ``amdpro-orca.icd`` with the contents: [path/to/drivers]/libamdocl-orca64.so

*(Again, assuming you are using the 64-bit drivers)*

*Please note that the name of the ICD files do not actually matter -- this is just what I named them. As long as they point to the correct* ``.so`` *files, they will still work.*

*DO NOT KEEP THE* ``mesa-opencl-icd`` *PACKAGE INSTALLED -- REMOVE IT!*

*(If the package is causing the issues... might be fixed, by the time you are reading this)*

# TEST IT #
To test it, you will want to install the package ``clinfo`` from the repos. Should detect your GPU, so long as your card supports OpenCL.

## WHAT ELSE ##
# KNOWN ISSUES #
1) As of the moment, since I went through this cockamamie BS to render using the Cycles engine on Blender with GPU compute, it should be noted that *FINAL* rendering works with Cycles, but you will *VERY LIKELY* need to switch to CPU compute for viewport rendering, or Blender will freeze and/or crash, if you are using materials, textures, loaded images, or nodes.

# OTHER INFORMATION #
1) Blender's RadeonProRender will probably crash Blender. I have no idea if this is just caused by the add-on, or if it is due to how I hacked together this config.

# HOW YOU CAN HELP #
If you happen to notice any further ways to improve this hack, please let me know, and I might test it out. This is certainly not the end-all be-all of bodging this mess together, and one of you persistent mofos could very well come up with a way to get better results.

## TO-DO ##
1) I'll try to write up a script to automate this, at some point.
