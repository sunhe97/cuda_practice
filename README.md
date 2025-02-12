# cuda_practice

## 1. vector_add.cu matrix multiplication 
single thread CPU compute took 2.334249 seconds for DSIZE = 1024 in matrix multiplication
GPU compute took 0.686000 seconds for DSIZE = 4096 in matrix multiplication
amazing!

## 2. shared_memory is faster than global memory, global memory is not in GPU, but shared memory is in GPU, so it is faster than global memory five times, but it is shared with threads in the same block, size is lower than 48k for each block， __syncthreads（） is used to synchronize threads in the same block, each thread must be waiting for other threads to finish.
in the first code, each calcualte will read data from global memory, in second code, split the matrix to several blocks, each block will read data from global memory first, the calculate the block's result (temp), then load next block's data to shared memory,  calculate and plus the next block's result (temp), finally, the last block will write the result to global memory.
each calculate will refresh the data of the corresponding position in global memory.


for example:
4 * 4 block size is 2 * 2

block 1 calulate 0，0, 0,1, 1,0, 1,1
shared memory load A0,0, A0,1, A1,0, A1,1   B0,0, B0,1, B1,0, B1,1

so, block 1 process:
load A0,0, A0,1, A1,0, A1,1   B0,0, B0,1, B1,0, B1,1

wait for other threads in the same block to finish

calculate tmp += A0,0 * B0,0 + A0,1 * B1,0 thread 0, 0 
calculate tmp += A1,0 * B0,1 + A1,1 * B1,1 thread 0, 1
calculate tmp += A1,0 * B0,0 + A1,1 * B1,0 thread 1, 0
calculate tmp += A1,0 * B0,1 + A1,1 * B1,1 thread 1, 1

wait for other threads in the same block to finish

four thread calculate unorderly, but the result is correct.

then block 2 process:
update in the same shared memory, load A0,2 A0,3 A1,2 A1,3 B0,2 B0,3 B1,2 B1,3(so the second update is needed)

wait for other threads in the same block to finish

calculate tmp += A0,2 * B0,2 + A0,3 * B1,2 thread 0, 0
calculate tmp += A1,2 * B0,3 + A1,3 * B1,3 thread 0, 1
calculate tmp += A1,2 * B0,2 + A1,3 * B1,2 thread 1, 0
calculate tmp += A1,2 * B0,3 + A1,3 * B1,3 thread 1, 1

wait for other threads in the same block to finish

......