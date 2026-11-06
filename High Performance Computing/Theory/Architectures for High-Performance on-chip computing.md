# GPUs
Multiple parts contained in a GPU do not have a counterpart inside CPUs.

IMMAGINE QUI

A lot of these additional elements are extremely specialized in certain types of operations (e.g. shader cores).
Typically, inside a CPU, one would expect to find a big cache block (MM latency is one of the main performance killers) and other parts. Instead, for GPUs, the priority is keeping only the simpler parts and replicate those to the extreme. (IMMAGINI!!!)

From these considerations the main problem would be the ab


The standard idea for CPU cache management was to have smaller private memories (L1 and L2) in the hope that the L2 cache could resolve most of the cache misses that happened in L1. The L3 memory is shared across all cores and is much bigger (so i requires much less bandwidth to work efficiently). 
In GPUs there are no cache blocks inside the single core, so, to make all the cores efficiently comunicate with the MM, a lot of bandwidth is required (even considered the big dimensions of the MM).
The problem is that the amount of computational power a GPU has (20 times more than a CPU, for an high-end GPU), is too much compared to the bandwidth gain w.r.t. CPUs (only 6x more).

On CPUs, speculative techniques are used to try and optimize the cache allocation (entire rows written on caches), in case the prediction proves to be wrong, there would be a cache miss which causes stalls. On GPUs, considered all the previous considerations, much preciser cache management methods are required to make up for the bandwidth problems.

- One way is limiting the number of data requests -> use the same data to do more math operations
- SOMETHING ELSE



The general idea of GPUs is that they are"throughput processors", but in reality their main characteristic is the fact that they are built specifically to optimize the way the data retrieved is used.


