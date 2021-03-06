This repository contains multi-thread, multi-node and hybrid (multi-core + multi-node) implementations of SVD and PCA using QR decomposition for Tall and Skinny matrices (TSQR).
This work has been inspired by work of Benson, Austin R., David F. Gleich, and James Demmel in "Direct QR factorizations for tall-and-skinny matrices in MapReduce architectures." and extends their work to introduce task parallelism.

This method of calculating PCA / SVD works only if cols >> rows, and this should be true even at lowest granularity i.e. even for individual core / worker thread, cols >> rows, otherwise this algorithm produces indeterminate results. 

This code uses Eigen C++ template library for performing linear algebra operations.

The makefile in each individual folder works only if Eigen library has been added on path. Otherwise the files can be compiled in the following way by mentioning the path to Eigen folder explicitly.

g++ -I /path/to/eigen/library serial.cc -o serial
g++ -I /path/to/eigen/library -fopenmp omp.cc -o omp
mpicxx -I /path/to/eigen/library mpi.cc -o mpi
mpicxx -I /path/to/eigen/library -fopenmp hybrid.cc -o hybrid

Command-line arguments
While executing the number of rows & cols of input matrix need to be mentioned. Besides this, number of threads needs to be mentioned if applicable. Also path to the input data file needs to be given as command line argument. Individual numbers should be seperated by white space. Currently does not support csv as input.

IMP :
Currently, the input matrix is being read in a naive way and might be very slow for large matrices owing to large number of disk writes. If the IO time is very high, I will write another method for file IO using buffers and supply it soon. 

For testing a simple bash script can be written similar to the one given below - 

#!/bin/bash

make

# For executing OpenMP code, write ./<name of executable> <number of rows> <number of columns> <number of threads>

echo "omp output"
./omp 4000000000 4 8
./omp 2500000000 10 8
./omp 500000000 50 8
./omp 150000000 100 8

# For executing MPI code, write mpirun -np <number of processors> <name of executable> <number of rows> <number of columns>

echo "mpi output"
mpirun -np 8 mpi 4000000000 4
mpirun -np 8 mpi 2500000000 10
mpirun -np 8 mpi 500000000 50 
mpirun -np 8 mpi 150000000 100

# For executing Hybrid code, write mpirun -np <number of processors> <name of executable> <number of rows> <number of columns> <number of threads>

echo "hybrid output"
mpirun -np 8 hybrid 4000000000 4 4
mpirun -np 8 hybrid 2500000000 10 4
mpirun -np 8 hybrid 500000000 50 4
mpirun -np 8 hybrid 150000000 100 4

The outputs (time for executing PCA / SVD ) would be directed to files in the cwd. 

The time for execution does not account for disk IOs involved (in the beginning while reading input). It only accounts time for computing SVD/PCA.

Also, as a part of sanity tests, the orthogonality tests should be done. i.e. calculating L2-norm of orthogonal matrices U & V after subtracting them from identity matrix.
    
	|U^T * U - I|_2 

This value will determine how orthogonal the matrices produced by this algorithm are and will validate the correctness of the algorithm.
