# Basics
CUDA (Compute Unified Device Architecture) is a software layer that gives direct access to the GPU's virtual instruction set (PTX - Parallel Thread eXecution). So, rather than a programming model, CUDA is best defined as an ecosystem; it is compatible with other programming models, there are specific IDEs, compilers, libraries (cuFFT, cuRAND, cuBLAS, cuDNN, nvGraph, NPP and more). The reason there are so many optimized libraries for so many applications is the fact that most of these are the main examples of GPU-friendly operations.




I missed the other starting points until slide 10.

### CUDA porting
1. The first thing to do is identifying the portions of code that need parallelizing and isolate them
2. What is also needed is to identify the data that need to be transferred on the device


In traditional multi-thread programming there are a lot of problems about synchronization, locks etc. this does not apply for GPU programming!

