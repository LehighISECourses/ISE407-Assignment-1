# Assignment 1

This second assignment is to continue getting you familiar with and using C, 
Python, and Julia, building on what we did in Assignment 0. Completing this 
assignment will require some further research and familiarization with the 
programming environments we'll use during the rest of the semester. We'll learn 
in more detail how to do the kinds of testing and analysis asked for in this
assignment throughout the course. The objective is simply to get familiar with
the tools and take a first try at doing some more substantive computational
experiments.

 1. The first part of the assignment is to write maive matrix multiplication 
    functions in C, Python, and Julia. We have already seen several examples of matrix 
    multiplication in Julia in 
    [Lecture 2](https://coral.ie.lehigh.edu/~ted/ie407/lectures/Lecture2.pdf). 
    For example, here is naive matrix multiplication in Julia.
    ```julia
    function matmult_naive!(C, A, B) 
       fill!(C, 0)
       for i ∈ 1:size(A, 1), j ∈ 1:size(B, 2), k ∈ 1:size(A, 2)
           C[i, j] += A[i, k] * B[k, j]
       end
       return(C)
     end
     ```
     Here is a corresponding naive matrix multiplication method in C. 
     ```c
     void matmult_naive(int **C, int ** A, int * dimA, int ** B, int * dimB){
        int rowA = dimA[0];
        int colA = dimA[1];
        int rowB = dimB[0];
        int colB= dimB[1];
        
        for (int i = 0; i < rowA; ++i){
           for (int j = 0; j < colB; ++j){
              for (int k = 0; k < colA; ++k){
                 C[i][j]+=A[i][k]*B[k][j];
              }
           }
        }
        
        return;
     }
     ```
    In the above, `dimA` and `dimB` should be arrays containing the dimensions of
    `A` and `B`, respectively. You should implement a similiar function in Python. 
    In C, you will need to wrap your functions in a main program to compile them,
    which may necessitate including a few header files. Also note that the above 
    function assumes the memory is allocated by the caller, so your main function 
    needs to do this. The way in which this is done is important. To take full 
    advantage of the cache optimization, you should allocate the matrix as one giant 
    block of memory, not as separate blocks for each row/column (feel free to 
    experiment with the difference between these two ways of allocating memory!).
    The specification for what arguments your function should take is discussed 
    in the parts below. 
 1. The second part is to optimize the naive functions given above. We
    have already seen how to do that in Julia in 
    [Lecture 2](https://coral.ie.lehigh.edu/~ted/ie407/lectures/Lecture2.pdf).
    In C, you should write functions that (1) take the transpose and (2) attempt to 
    cache optimize in a fashion similar to what we did in Julia. You may find 
    [this article](http://lwn.net/Articles/250967/) useful for writing your C code
    (if you want to have the fastest possible code, the version from this article that
    uses pointer arithmetic, although harder to read, should make the dereferencing 
    faster). Your function signatures should be the same as above for your additional 
    functions, and the naming should be the same as the way we named the Julia functions 
    we saw in class. For Python, it probably doesn't make sense to cache optimize, but you 
    can try with and without taking the transpose. There are fancy ways to take the 
    transpose in Python that you will find with a little Googling. 
 1. Finally, you should do some testing and comparison of your codes. There are a number 
    of ways to go about doing this. You'll need a test program in each language that 
    generates random matrices in some "reasonable" way. Using your test programs, compare the 
    speeds of your implementations on one or more graphs. The graphs should show
    the effect of the size of the matrices and should show how the
    implementations compare to each other. It is probably easiest to do all this by
    writing a single wrappe script in either bash, Python, or Julia that runs the programs, 
    obtains the timing information, and compiles plots. 
    
    As an example, here is how you might call a C code from Python and collect data. 
    ```python
    import subprocess
    import sys, os
    import matplotlib.pyplot as plt
    import pickle
    import itertools

    markers = itertools.cycle((',', '+', '.', 'o', '*')) 

    read_cached = False

    opt_level = "-O1"

    if not read_cached:
        versions = {'naive' : 1, 'transpose' : 2, 'cache' : 3}
        sizes = [800, 1600, 2400, 3200, 4000]
        results = {}
        for j in versions:
            results[j] = []
            for k in sizes:
                print ("%s with size %d" % (j, k))
                c = subprocess.run([os.getcwd()+"/matmult"+opt_level, "%d" % k,
                                    "%d" % versions[j], "0"], 
                                   stdout=subprocess.PIPE)
                results[j].append(float(c.stdout.decode('ascii').split()[6]))
        with open('results'+opt_level+'.dat', 'wb') as f:
            pickle.dump(results, f)
            pickle.dump(sizes, f)
            pickle.dump(versions, f)
    else:
        with open('results'+opt_level+'.dat', 'rb') as f:
            results = pickle.load(f)
            sizes = pickle.load(f)
            versions = pickle.load(f)

    for j in versions:
        if j == 'naive':
            continue
        plt.plot(sizes, results[(i,j)], marker = next(markers), label="%s" % j)

    plt.legend()
    plt.show()
    ```
    Note that the C code is assumed to run just one experiment each time, 
    taking the size of the matrix (assumed square) as an input. The memory
    allocation and the random generation of the matrices takes place inside the 
    code, but you should measure and output only the time associated with the matrix 
    multiplication function call itself. 
    
    When building your C code, it is important o keep in mind the optimization level.
    In general, you should call the compiler with `-O2` to get the highest level of 
    optimization.
 1. You should provide a Makefile with a target `experiment` that builds your code, 
    runs your experiments and produces the results. You should also include a target
    `test`, as before, that will be automatically run to test your code. This target
    should simply compare the matrices produced with different methods to make sure
    they're the same. For C, you can simply use the `assert` funtion to do some simple
    testing and compile with debugging enabled. We'll discuss how to do this in class.
 1. Compare and contrast the two languages in light of your results and
    explain the outcomes of the experiments in as much depth as possible.
 1. Explain what the following assembly code shown in Lecture does line by line. 
    You may find [this explanation](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)
    helpful. 
    ```assembly_x86
                .text
            cmpq    %rcx, %rdx
            jle     L17
            movq    $0, (%rdi)
            movb    $2, %dl
            xorl    %eax, %eax
            retq
    L17:
            xorl    %r8d, %r8d
            nopw    %cs:(%rax,%rax)
            nop
    L32:
            addq    %rdx, %r8
            leaq    (%rdx,%r8), %rax
            cmpq    %rcx, %rax
            jle     L32
            movq    (%rsi), %rax
            movq    -8(%rax,%r8,8), %rax
            movq    %rax, (%rdi)
            movb    $1, %dl
            xorl    %eax, %eax
            retq
            nopl    (%rax)
    ```
    
