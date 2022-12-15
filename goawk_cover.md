
# Coverage for GoAWK
    
### TLDR

I contributed to [GoAWK project](https://github.com/benhoyt/goawk) by implementing the [code coverage](https://github.com/benhoyt/goawk/blob/master/docs/cover.md) functionality.

I'm grateful to [Ben Hoyt](https://benhoyt.com/) (the creator of GoAWK) for being really patient and helpful throughout my contribution process, providing thorough and insightful code reviews.
      
### What is code coverage?

In plain words [code coverage](https://en.wikipedia.org/wiki/Code_coverage) is a measure that tells how good is your source code covered by tests.

But why may we want it for [AWK](https://en.wikipedia.org/wiki/AWK)? 

### How the idea emerged, motivators

#### Makesure

[Makesure](https://github.com/xonixx/makesure) is a task/command runner that I'm developing. It's sort of similar to the well-known `make` but without most of its idiosyncrasies (and with a couple of unique features!).

It may sound surprising, but I write it in AWK. I won't lie, it started more out of curiosity than out of real need. To me limits encourage creativity. Few people know nowadays that AWK is a full-fledged programming language, though really minimalistic. 

So the Makesure started more like an experiment to check how far it can go. It appears, pretty far. I think, this is not an exception, but rather a consistent pattern, due to true genius of [A., W., K.](https://archive.org/download/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf):

> [I wrote a compiler in awk!](https://news.ycombinator.com/item?id=13452043)
>
> To bytecode; I wanted to use the awk-based compiler as the initial bootstrap stage for a self-hosted compiler. Disturbingly, it worked fine. Disappointingly, it was actually faster than the self-hosted version. But it's so not the right language to write compilers in. Not having actual datastructures was a problem. But it was a surprisingly clean 1.5kloc or so. awk's still my go-to language for tiny, one-shot programming and text processing tasks.
                   
The other idea behind choosing AWK for the Makesure is the ease of parsing 
```
@goal built
@depends_on tested
    gcc code.c 
```
If you know a bit of AWK, you understand that this syntax is fairly easy to parse with it. AWK already does words splitting for you, so all you need is
```awk
if      ($1 == "@goal")       handleGoal()
else if ($1 == "@depends_on") handleDependency()
else                          handleCodeLine()
```

Essentially, using the limited tool for the job (AWK) provokes you to comply with [Worse is better](https://en.wikipedia.org/wiki/Worse_is_better) principle, that I'm big proponent of. You don't invent fancy syntax, but rather rely on one that actually can be parsed straight forward.

Time passed and source code for the Makesure grew to a [pretty big awk file](https://github.com/xonixx/makesure/blob/main/makesure.awk). The tool had pretty extensive test suite, but I was not sure of how good the coverage is, whether all critical scenarios are tested or not.
Thus, the need to test coverage [became apparent](https://github.com/xonixx/makesure/issues/103). The research revealed that no AWK implementation had code coverage facility. And so I decided to add one to some AWK implementation. The most suitable for such addition appeared the [GoAWK](https://github.com/benhoyt/goawk).
 
Of course, some of my other motivators were practicing my Golang and AWK skills. 
                
### How coverage works, in simple words
                                        
Essentially, to collect code coverage statistics the source code instrumentation is used. That is, each line of a code is annotated with a piece of tracking code. Thus, when the instrumented program runs, the tracking code is triggered each time the code line is hit. At the program end all collected coverage profile data is written to a file. This file is then used to create the human-friendly coverage report like below. 

![](goawk_cover.png)

### How code coverage for AWK differs from mainstream languages

For testing `makesure` I'm using the [tush](https://github.com/adolfopa/tush) tool.
~~Accidentally, it's written in AWK, but it's not the point.~~ 
The tool provides really nice and simple way to test a CLI-tool as a black box.  
The tests in `tush` look like
```
$ command --that --should --execute correctly
| expected stdout output

$ command --that --will --cause error
@ expected stderr output
? expected-exit-code
```

When running the test, the tool runs all lines starting with `$` and simply compares the actual output with the expected one (`|` for stdout, `@` for stderr) using a usual `diff`. If there is a difference, the test fails and the `diff` output is displayed to the user.

In essence, such tests are End-to-end tests that are on the top of well-known testing pyramid.

Using this approach I created the comprehensive test suite for the tool with a typical test file [looking like this](https://github.com/xonixx/makesure/blob/main/tests/10_define.tush).

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
         
### What's lacking

### Links

- Please also take a look at [GoAWK code coverage article by Ben Hoyt](https://benhoyt.com/writings/goawk-coverage/).
- https://github.com/benhoyt/goawk/blob/master/docs/cover.md