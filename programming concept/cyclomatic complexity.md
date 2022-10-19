The cyclomatic complexity is a measurement of the code complexity proposed by McCabe which is often considered as a magic number which allows us to measure the complexity of a program. It is common to say that a function with a cyclomatic complexity higher than 10 is difficult to maintain due to an over-reliance on branches, switch/cases or loops. These functions are often a good candidate for a redesign.

How good is this metric? Would the number of lines of code give us the same result? If that were true, it would make more sense to the simpler metric since everybody intuitively understands it.

To answer this question, I will expose how to compute it and then analyze some properties.

## How to Compute the Cyclomatic Complexity

The formula of the cyclomatic complexity of a function is based on a graph representation of its code. Once this is produced, it is simply:

_M = E – N + 2_

_E_ is the number of edges of the graph, _N_ is the number of nodes and _M_ is McCabe’s complexity.

To compute a graph representation of code, we can simply disassemble its assembly code and create a graph following the rules:

1.  Create one node per instruction.
2.  Connect a node to each node which is potentially the next instruction.
3.  One exception: for a recursive call or function call, it is necessary to consider it as a normal sequential instruction. If we don’t do that, we would analyze the complexity of a function and all its called subroutines.

Let have a try with a small sample:
```cpp
#include <stdio.h>

int main( int argc, char *argv[] )
{
    if ( argc > 1 )
    {
        fprintf( stderr,
                "This program does not accept arguments!\n"
               );
        return 1;
    }
    else
    {
        fprintf( stdout, "Hello world!\n" );
        return 0;
    }
}
```