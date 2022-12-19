
# Code Coverage for GoAWK
    
### TL;DR

I contributed to [GoAWK](https://github.com/benhoyt/goawk) project by implementing the [code coverage](https://github.com/benhoyt/goawk/blob/master/docs/cover.md) functionality.

I'm grateful to [Ben Hoyt](https://benhoyt.com/) (the creator of GoAWK) for being really patient and helpful throughout my contribution process, providing thorough and insightful code reviews.
      
### What is code coverage?

In plain words [code coverage](https://en.wikipedia.org/wiki/Code_coverage) is a measure that tells how good is your source code covered by tests.

But why may we want it for [AWK](https://en.wikipedia.org/wiki/AWK)? 

### How the idea emerged, motivators

[Makesure](https://github.com/xonixx/makesure) is a task/command runner that I'm developing. It's sort of similar to the well-known `make` but without most of its idiosyncrasies (and with a couple of unique features!).

It may sound surprising, but I write it in AWK. I won't lie, it started more out of curiosity than out of real need. To me limits encourage creativity. Few people know nowadays that AWK is a full-fledged programming language, though really minimalistic. 

So the Makesure started more like an experiment to check how far can it go. It appears, pretty far. I think, this is not an exception, but rather a consistent pattern, due to the true genius of the AWK authors.

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
if      ($1 == "@goal")       handleGoal($2)
else if ($1 == "@depends_on") handleDependency($2)
else                          handleCodeLine($0)
```

Essentially, using the limited tool for the job (AWK) provokes you to comply with [Worse is better](https://en.wikipedia.org/wiki/Worse_is_better) principle, that I'm big proponent of. You don't invent fancy syntax, but rather rely on one that actually can be parsed straight forward.

Time passed and source code for the Makesure grew to a [pretty big awk file](https://github.com/xonixx/makesure/blob/main/makesure.awk). The tool had pretty extensive [test suite](https://github.com/xonixx/makesure/tree/main/tests), but I was not sure of how good the coverage is, whether all critical scenarios are tested or not.
Thus, the need to test coverage [became apparent](https://github.com/xonixx/makesure/issues/103). The research revealed that no AWK implementation had code coverage facility. And so I decided to add one to some AWK implementation. The most suitable for such addition appeared to be the GoAWK.
 
Of course, some of my other motivators were practicing my Golang and AWK skills. 
                
### How coverage works, in simple words
                                        
Essentially, to collect code coverage statistics the source code instrumentation is used. That is, each line of a code is annotated with a piece of tracking code. Thus, when the instrumented program runs, the tracking code is triggered each time the code line is hit. At the program completion all collected coverage profile data is written to a file. This file is then used to create the human-friendly coverage report like below. 

![](goawk_cover.png)

### How code coverage for AWK differs from mainstream languages

For testing `makesure` I'm using the [tush](https://github.com/adolfopa/tush) tool. It provides really nice and simple way to test a CLI-tool (as a black box).  

The tests in `tush` look like
```
$ command --that --should --execute correctly
| expected stdout output

$ command --that --will --cause error
@ expected stderr output
? expected-exit-code
```
(in my case `command` = `makesure`)

When running such test, `tush` runs all lines starting with `$` and simply compares the actual output with the expected one using a usual `diff`. If there is a difference, the test fails and the `diff` output is displayed to the user.

In essence, such tests are end-to-end tests that are on the top of well-known [testing pyramid](https://automationpanda.com/2018/08/01/the-testing-pyramid/). This is very good, since this style of testing is equivalent to how _real_ user uses the tool. Thus, much higher chances catching _real_ bugs. 

Using this approach I created the comprehensive test suite for the tool with a typical test file [looking like this](https://github.com/xonixx/makesure/blob/main/tests/10_define.tush).

The usual approach to testing (like in Go, Python or Java) is you create a set of tests (usually, in multiple source files), then you use some "test runner" to run all them at once. Here lays a subtle but principal difference from the approach described above.
So in case of test runner you run it once for all tests and so it can form the coverage profile file at completion for all tests.

In case with `tush` the program-under-test (`makesure`, and thus the `awk` underneath) is called lots of times (with different arguments). So it means we want to have a way to assemble the final coverage profile file incrementally. 

This poses additional implementation challenge. So we need that separate runs of AWK program be able to append to a single coverage profile file. This implies that not only the internal format of this file needs to be "appendable", but also requires that the reporting facility that consumes the file "understands" this "appendability".

### How I thought to approach the issue
         
Couple years ago I came across the article [Code Coverage for Solidity](https://blog.colony.io/code-coverage-for-solidity-eecfa88668c2/). The article was pretty insightful for me, as it gave enough of technical explanation and described typical challenges when implementing code coverage for particular programming language.  

The author based his implementation on [Instabul](https://istanbul.js.org/) coverage tool, initially targeted on JavaScript. But the author managed to generate the coverage profile for the other language (Solidity) in the format of the tool. Thus, he could reuse the reporting facility of Istanbul for free.

My plan was to use the similar approach. But my research have shown that their [coverage profile file format](https://github.com/gotwarlost/istanbul/blob/master/coverage.json.md) was not append-friendly. 
      
The other direction of my thought was defined by the classical book [The AWK Programming Language](https://archive.org/download/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) by Alfred V. Aho, Brian W. Kernighan, Peter J. Weinberger (the A., W., K. of AWK. By the way it's absolute pleasure to read and, amazingly, totally relevant, despite been published in 1988).

On pages 167-169 of this book there is a chapter "7.2. Profiling". It describes a very simple approach how one can implement the profiling of a program by just two AWK scripts `makeprof` and `printprof` of around 5 lines each (sic!). 

`makeprof` does (admittedly naive) transformation of input source by inserting `_LBcnt[i]++;` after each left brace `{`. Then it also adds the `END` clause that outputs the collected statement counts in `_LBcnt` to a file. 

`printprof` attaches the statement counts from this file to the original program.

I was completely fascinated by the simplicity and clarity of this approach! 

Despite it was far from perfect and practical, I really considered using some improved version of this approach. However, it was clear that to make it more precise and robust, the AST-level transformation was needed, not just naive source code modification.   
    
#### Chosen implementation approach
                          
Working closely with AWK for my project, one day I came across the GoAWK. Out of curiosity I tried running the Makesure with it. This revealed some bugs in GoAWK, that I've reported, and they were fixed promptly by Ben Hoyt, the creator. So eventually GoAWK passed the test suite of Makesure.

At that time I myself started getting interested in learning Golang. So I was happy to contribute myself some other minor bug I've encountered. Overall I really liked the project and especially the author behind it. I can't help but recommend [his technical blog](https://benhoyt.com/writings/) - very well written and interesting.

So I thought that whatever implementation approach I choose, I would like to use GoAWK as a base. Since GoAWK was written in Go, I decided to take a look on coverage story of Golang itself. This was when I came across [this article on Go's cover](https://go.dev/blog/cover). And it was an instant hit! 

The functionality was not as advanced as with Istanbul, but still enough. The implementation seemed to be relatively simple. But most importantly, their coverage profile file format appears to be designed with appendability in mind! (Though, to my knowledge this feature is not directly used by Golang itself). 

So decided! We add code coverage functionality to GoAWK using the Golang's coverage profile file format. And then reusing the Golang machinery (`go tool cover`) to render final report.

#### Failed (hacky) attempt

_Firstly it would be really beneficial for the reader in order to better understand the following reading to read first the [Ben's own writing on GoAWK](https://benhoyt.com/writings/goawk/)._

GoAWK uses pretty standard programming languages implementation design, and therefore execution strategy:

1. Firstly the input AWK source (as a string) is processed by [**Lexer**](https://benhoyt.com/writings/goawk/#lexer), outputting a list of tokens. 
2. Then the list of tokens serves as an input for [**Parser**](https://benhoyt.com/writings/goawk/#parser), and now the output is an AST (abstract syntax tree).
   - There is also a notion of [**Resolver**](https://benhoyt.com/writings/goawk/#resolver) that was initially a part of **Parser**. It analyzes the AST and annotates it with some additional data, like the information about inferred types. 
3. Afterwards, the AST [passes through](https://benhoyt.com/writings/goawk-compiler-vm/) the **Bytecode Compiler** producing list of opcodes (bytecodes, instructions) - the GoAWK assembly code.
4. Finally, the bytecode serves as input to **Interpreter**, which has a Virtual Machine inside, that just runs the opcodes instruction by instruction. 

Now I needed to understand how to fit the code coverage functionality into this scheme.

As I explained earlier, we need to instrument the source code for coverage tracking.
It's much easier to explain on example:

Given the source
```awk
BEGIN {
    print "will always run"
    if ((1 + 1) == 2) {
        print "should run"
    } else {
        print "won't run"
    }
}
```
the instrumented code will look like
```awk
BEGIN {
    __COVER["3"] = 1     # track coverage
    print "will always run"
    if ((1 + 1) == 2) {
        __COVER["1"] = 1 # track coverage
        print "should run"
    } else {
        __COVER["2"] = 1 # track coverage
        print "won't run"
    }
}
```
It was obvious that this tracking code insertion should take place on the AST level. So I imagined we'll need to add the additional step between 2 and 3. It will take AST as input, pass it through some **CoverageAnnotator** (yet to be added) and produce the transformed AST.

Not so simple in practice. And the main problem here was the tight connection of **Parser** and **Resolver**. In fact, both of them worked on same pass, so it was impossible to run them separately. 

Let me explain, why this is important. If I just change the AST by adding the required AST nodes for the `__COVER["N"] = 1` statement, those nodes will lack some tiny (but important) pieces of information filled by the Resolver. So somehow I needed to run the updated AST through the resolution step once again. But this was technically impossible, because Resolver (being part of Parser) could only consume what parser consumes (that is list of tokens). 

So I thought, why not just render AST back to AWK source and then simply start the whole process from step 1. Luckily, GoAWK already had `String()` implementation for all AST nodes, so it could render AST back to AWK. Unluckily, this AWK source [was not guaranteed to be correct](https://github.com/benhoyt/goawk/issues/142), because its only purpose was to output the parsed AST tree for debug purposes (flag `-d`).

This is when I added the first dirty hack - I patched the instrumented AST (more precisely, the inserted pieces) as if this was done by Resolver itself. 

The second hack I used was even nastier. You see, when we insert cover point like

```awk
__COVER["2"] = 1
```

we also internally store the information that describes it, like filename and source interval being covered, etc. Basically this information is captured as instance of `trackedBlock` structure: [link](https://github.com/benhoyt/goawk/blob/master/internal/cover/cover.go#L43).
So internally we have (Go pseudocode):
```go
trackedBlocks[2] = trackedBlock { 
   path:     "/path/to/script.awk", 
   start:    "2:3", 
   end:      "35:13", 
   numStmts: 34,
}
```

Storing this data is needed for building the resulting coverage profile file (in the format used by Go cover):
```
...
/path/to/script.awk:2.3,35.13 34 1
/path/to/script.awk:41.39,41.51 1 0
/path/to/script.awk:40.5,41.39 2 0
/path/to/script.awk:42.27,42.42 1 0
...
```
Explanation: each line describes collected coverage for some block of code and has format
```
<source_file_path>:<column_from>.<line_from>,<column_to>.<line_to> <num_stmts> <exec_count>
```


Now, remember, you can run AWK with multiple files like so

```
awk -f file1.awk -f file2.awk -f file3.awk
```

But GoAWK in step 1 just joins all source files into single string and uses it as an input to Lexer. 

This means, that by the time we've got the AST after step 2 there is absolutely no way to tell which AST node came from what input file. Thus, we aren't able to fill the `path` field for our `trackedBlock` during instrumentation.

So to pass the required `path` information to AST it needs to wire it through both Lexer and Parser, I thought. For this [I introduced a "fake" token](https://github.com/benhoyt/goawk/compare/070521a687628ad88f731b734077f15e6ec16f92...c067bfc8212836ad9cd6cb1783bfdfb6d61a0b7b) that I inserted at start of each file. Thus, when sources were joined this token represented files boundaries. It means, that at parse time I was able to track the filenames and local positions in each file.  


#### Refactorings
                           
Although the approach worked well in practice, it was clear that it was ugly, and that it had zero chances to be merged as is. Ben came up with a [list of code review remarks](https://github.com/benhoyt/goawk/issues/144#issuecomment-1223087337).
There was no other way than to undertake a couple of serious refactorings before we can even discuss the merging of my effort. 

Frankly, I was not upset at all, but rather excited! Firstly, I've got a chance to do even more useful work for the project I liked. And I was still glad to keep sharpening my Go fu further.

The biggest thing to start with was decoupling Parse and Resolve steps. This was a bit scary thing to touch. Even Ben described this part of code as 

> In fact, the resolver was one of the harder pieces of code I’ve written for a while. It’s the one piece of the GoAWK source I’m not particularly happy with. It works, but it’s messy, and I’m still not sure I’ve covered all the edge cases.
                            
But looked like I was getting overall how it worked and how can I do the job. Reading Ben's tech writings was of big help. 

I needed the way to traverse (and update) the AST for resolving step after the parsing. 
For this [I used the Visitor pattern](https://github.com/benhoyt/goawk/blob/master/internal/ast/walk.go) approach, very similar to one, [used in Golang](https://github.com/golang/go/blob/690ac4071fa3e07113bf371c9e74394ab54d6749/src/go/ast/walk.go). Once the visitor functionality was in place, the AST traversal needed for resolving [was easy](https://github.com/benhoyt/goawk/blob/d7911647f3b08af099b50c9e991e72cfacdd1e2e/internal/resolver/resolve.go#L59).

My [other refactoring](https://github.com/benhoyt/goawk/pull/153) served the purpose of resolving (global) positions in joined source back to (local) positions in input source files. This was needed to get rid of hack #2 mentioned above.    
           
When all needed refactorings were reviewed and merged to master I was able to submit my [final clean implementation](https://github.com/benhoyt/goawk/pull/154) on top.

Overall I highly recommend this approach to big contributions. You should not try to fit all your changes into a single pull request. Each PR should be "highly cohesive", i.e. solve only single problem. This is much easier for the project owner to review and accept such PR. And this is also because each of such PRs will be valuable by its own, even if not all of them manage to reach the master branch. This is because, no doubt, [problem decomposition is the most important idea in all of computer science](https://www.youtube.com/watch?v=bmSAYlu0NcY&t=187s). 

### Results

Running GoAWK coverage on Makesure revealed [uncovered places](https://github.com/xonixx/makesure/issues/111). This helped to find a bug (already fixed).
         
### What's lacking

As I've mentioned, the Go's coverage approach is not very advanced. It implements _statement_ coverage rather that _branch_ coverage (the same applies to GoAWK coverage). For simplicity, [it doesn't cover some corner cases](https://go.dev/blog/cover#basic-blocks). Other coverage tools, like Istanbul can provide more detailed coverage, like coverage for [both of ternary cases](https://blog.colony.io/code-coverage-for-solidity-eecfa88668c2/#the-ternary-operator) or [invisible else branch](https://blog.colony.io/code-coverage-for-solidity-eecfa88668c2/#invisible-else-branches).
To illustrate a point, let's imagine the code
```awk
if (codition) {
  print "body"
}
print "after"
```
You can have it all green, but still something not covered. The thing is, the (explicitly absent) else branch is not covered. In other words we want to make sure both variants (`true`/`false`) for `condition` are covered.

It would be really nice to improve this for GoAWK coverage and maybe even for Golang itself.
  

### Links

- [GoAWK code coverage article by Ben Hoyt](https://benhoyt.com/writings/goawk-coverage/)
- [GoAWK coverage documentation](https://github.com/benhoyt/goawk/blob/master/docs/cover.md)
- [GoAWK, an AWK interpreter written in Go](https://benhoyt.com/writings/goawk/)
- [The Go Blog - The cover story by Rob Pike](https://go.dev/blog/cover)
- [Code Coverage for Solidity](https://blog.colony.io/code-coverage-for-solidity-eecfa88668c2/)
- [The AWK Programming Language](https://archive.org/download/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) - the book by AWK creators, must read