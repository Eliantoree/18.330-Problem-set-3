# 18.330-Problem-set-3
18.330 Problem set 3

**Download Link:https://programming.engineering/product/15-418-618-exercise-3/**

Description
5/5 – (3 votes)
Overview

This exercise is designed to help you better understand the lecture material and be prepared for the style of questions you will get on the exams. The questions are designed to have simple answers. Any explanation you provide can be brief—at most 3 sentences. You should work on this on your own, since that’s how things will be when you take an exam.

You will submit an electronic version of this assignment to Gradescope as a PDF file. For those of you familiar with the LATEX text formatter, you can download the template and configuation files at:

http://www.cs.cmu.edu/˜418/exercises/config-ex3.tex

http://www.cs.cmu.edu/˜418/exercises/ex3.tex

Instructions for how to use this template are included as comments in the file. Otherwise, you can use this PDF document as your starting point. You can either: 1) electronically modify the PDF, or 2) print it out, write your answers by hand, and scan it. In any case, we expect your solution to follow the formatting of this document.

Problem 1: GPU Programming

Your friends have been hearing that you now know how to program a GPU and take advantage of all its capabilities. Eager to bask in your knowledge, they come to you with different problems that they think might benefit from GPU parallelism.

One such friend heard that GPUs can launch thousands of parallel “threads.” They want to identify the indices of a large array of n elements in an even larger array (of size m) using binary search. They suggest to you that every thread on the GPU will be searching for one element in the array, so that with say, 1024 threads, they’ll be able to search get near 1024 performance.


First off, how do you explain that threads on a CPU are not the same as threads on a GPU? (What’s the difference between the implementation of p threads and CUDA threads?) Give at least 3 important distinctions.

Still convinced of their binary search’s potential, they write this kernel function.

Array S and R are of size n

Array A is of size m, with its elements ordered.

__global__ void cudaBinarySearch(int n, int m, int *S, int *A, int *R) { int i = blockDim.x * blockIdx.x + threadIdx.x;

if (i >= n) return;

int search = S[i];

int left = 0, right = m – 1, middle = 0; R[i] = -1; // In case not found // Binary Search

while (left <= right) {

middle = left + (right – left) / 2;

if (search < A[middle])

right = middle – 1;

else if (search > A[middle])

left = middle + 1;

else

R[i] = middle;

break;

}

}

After testing it out, they find that they get very little performance gain relative to the number of threads that they are running. Sullenly, they come back to you and ask where they went wrong. Give at least two reasons for this code’s poor performance.


Problem 2: Problem Scaling

This problem is a three-dimensional extension of the grid problem described in the lecture on workload-driven performance evaluation:

http://www.cs.cmu.edu/˜418/lectures/10 perfeval.pdf.

Consider an iterative solver that operates on a three-dimensional grid of size N N N. It requires N iterations to reach convergence. We refer to the parameter N as the problem size. Increasing N by a factor of 2 increases the number of grid elements 8 and the number of iterations 2 .

The state of the system is represented as a three-dimensional array of values state. The function update computes a new value of the state based on the current state. For each element state[x][y][z], update computes the new value as a function of its current value and that of its six neighbors:

Self: state[x][y][z]

West: state[x-1][y][z]

East: state[x+1][y][z]

North: state[x][y-1][z]

South: state[x][y+1][z]

Down: state[x][y][z-1]

Up: state[x][y][z+1]

(You don’t need to know the exact function.)

The overall operation can then be described in a pseudo-C notation as:

// For each iteration

for (iter = 0; iter < N; iter++) {

// Main computation

for all x, y, z in 0..N

nstate[x][y][z] = update(state, x, y, z);


(a)

N

2N

(b)

P

8P

Figure 1: Scaling a problem by doubling the grid size N (a), or by increasing the number of processors 8 (b).

for all x, y, z in 0..N state[x][y][z] = nstate[x][y][z]

}

In mapping the computation onto P processors, the state array is split into P cubic blocks, each containing N3=P array elements. Each iteration (labeled “Main computation”) involves having each processor 1) communicate the boundary values with its (up to six) neighbors, and 2) computing the update function for all N3=P of its assigned elements. The program uses M gigabytes of memory per processor and runs in total time T .

Consider the following two scaling possibilities, yielding a problem of size N0 running on P 0 proces-sors, as illustrated in Figure 1.

Double the value of N, while holding P constant: N0 = 2N, P 0 = P

Hold N constant, while increasing P by 8 : N0 = N, P 0 = 8P

Fill in the table below showing how the amounts of computation and communication for a single processor, performing a single iteration would scale, relative to the original problem. (For example, 2 would indicate growth by a factor of two.)

Computation

Communication

(a): N0 = 2N, P 0 = P

(b): N0 = N, P 0 = 8P

Now assume both N and P can vary. Based on these specific cases, give formulas for how the compu-tation, communication, and arithmetic intensity would scale as functions of N and P . Again, quantify these values with respect to the computation and communication each processor would perform dur-ing a single iteration. (By way of reference, the formulas for the two-dimensional version were given in slide 7 of the lecture.)


A. Is the memory system still coherent? What impact does this change have on the system?


You are hired by Intel to design an invalidation based cache-coherent processor with a large number of cores. The main application for this processor will be to train a chess AI by playing games against itself for experiments. It is critical that it doesn’t play too many games during training to maintain the validity of the experiment, so we store how many games are played in a shared counter:

int num_games = 100000;

int games_played = 0;

This code is run on each core while (true) {

Atomically get value of count prior to increment, and

write incremented value to games_played

int val = atomic_add(&games_played, 1);

if (val > num_games) break;

play_game()

}

Unsure on how to design the cache coherence of this processor, you check with one of the senior design-ers. She suggests that you should use a bus-based, snooping coherence implementation, rather than a directory-based protocol, because the broadcasting it performs would be more efficient in this case. Do you agree or disagree? Why or why not?


You are now tasked with finding the best performing chess AI. You plan to run the following code on the processor:

int num_experiments = 10000;

int best_score = 0;

#pragma omp parallel for

for (int i = 0; i < num_experiments; i++) { int score = perform_experiment(i);

if (score > best_score) {

Atomically retrieve best score, compute max,

and write result to best_score

atomic_max(&best_score, score);

}

}

Trying to improve the performance of this code, you remove the if statement prior to the atomic_max, noticing that it is redundant. Suprisingly, the performance of the program is drastically reduced. Why did this happen?


You are hired as a TA for 15-418/618. In an attempt to speed up exam grading, you decide to multithread the grading program on a cache coherent system as shown below

int scores[NUM_STUDENTS];

#pragma omp parallel for schedule(dynamic)

for (int student = 0; student < NUM_STUDENTS; student++) {

for (int problem = 0; problem < NUM_EXAM_PROBLEMS; problem++) {

// performs a write to scores[student]

grade_and_update(student, problem, &scores[student]);

}

}

Dismayed by the speed of this program, you analyze the runtime of this program and find that there is a large amount of bus communication during the execution.

What kind of communication is happening in this program? What performance problem is this a result of?

9
