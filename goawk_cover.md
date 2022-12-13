
# Coverage for GoAWK
    
### TLDR

I contributed to [GoAWK project](https://github.com/benhoyt/goawk) by implementing the [code coverage](https://github.com/benhoyt/goawk/blob/master/docs/cover.md) functionality.

I'm grateful to [Ben Hoyt](https://benhoyt.com/) (the creater of GoAWK) for giving thorough and insightful code reviews and being patient.
      
### What is code coverage?

In plain words [code coverage](https://en.wikipedia.org/wiki/Code_coverage) is a measure that tells how good is your source code covered by tests. 

### How the idea emerged, motivators

#### AWK

I myself been dedicated Python lover in past for many years now came to a conclusion that what can be scripted with AWK, should be scripted in AWK (over Python, Ruby, Perl, etc.). I'm not saying that you should write big apps though, but for small scripts AWK is absolutely fine alternative to major scripting languages with lots of benefits. Been universally available (as part of POSIX) and very compliant (language standard is almost unchanged for over 30 years now). As they say: "Good programmer chooses the most powerful tool for the job, the best programmer chooses the least powerful tool for the job"

- (take material from my AWK article)
- https://github.com/vladcc/shawk/blob/7420a88ce2025f3fe7390efb2b11e29d5b7b6b80/README.md#why-shell--awk

#### Makesure

This is a task/command runner that I'm developing. It's sort of similar to the well-known `make` but without most of its idiosyncrasies.

It may sound surprising, but I write it in `AWK`. I won't lie, it started more out of curiosity than out of real need. To me limits encourage creativity. Few people know nowadays that AWK is a full-fledged programming language, though really minimalistic. I was absolutely astonished when I read a classical book [The AWK Programming Language](https://archive.org/download/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) by Alfred V. Aho, Brian W. Kernighan, Peter J. Weinberger (the A., W., K. of AWK). It was absolute pleasure to read. Amazing, but it's still totally relevant, despite been published in 1988.

So the Makesure started more like an experiment to check how far it can go. It appears, pretty far. I think, this is not an exception, but rather a consistent pattern, due to true genius of A., W., K.:

> [I wrote a compiler in awk!](https://news.ycombinator.com/item?id=13452043)
>
> To bytecode; I wanted to use the awk-based compiler as the initial bootstrap stage for a self-hosted compiler. Disturbingly, it worked fine. Disappointingly, it was actually faster than the self-hosted version. But it's so not the right language to write compilers in. Not having actual datastructures was a problem. But it was a surprisingly clean 1.5kloc or so. awk's still my go-to language for tiny, one-shot programming and text processing tasks.

- https://github.com/xonixx/makesure/issues/103
 
Practicing Go, AWK
                
### How coverage works, in simple words

 - code instrumentation
 - ...

### How code coverage for AWK differs from mainstream languages

 - need for `-coverappend`

### How I thought to approach the issue
                       
 - [Code Coverage for Solidity](https://blog.colony.io/code-coverage-for-solidity-eecfa88668c2/)
 - The AWK Programming Language by A. W. K. pp. 167-169 - 7.2. Profiling 
    
### Implementation

#### Chosen approach
    
Reuse Golang machinery.

- https://go.dev/blog/cover

#### Failed (hacky) attempt

#### Refactorings

The approach to big changes. 

- https://twitter.com/iamdevloper/status/397664295875805184?lang=en

### Results

Revealed [uncovered places](https://github.com/xonixx/makesure/issues/111) and eventually found (and fixed) a bug.

Please also take a look at [GoAWK code coverage article by Ben Hoyt](https://benhoyt.com/writings/goawk-coverage/).