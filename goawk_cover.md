
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

So the Makesure started more like an experiment to check how far it can go. It appears, pretty far. I think, this is not an exception, but rather a consistent pattern, due to the true genius of the AWK authors.

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
Thus, the need to test coverage [became apparent](https://github.com/xonixx/makesure/issues/103). The research revealed that no AWK implementation had code coverage facility. And so I decided to add one to some AWK implementation. The most suitable for such addition appeared to be the [GoAWK](https://github.com/benhoyt/goawk).
 
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

~~Luckily, the approach we took already had - describe in chosen approach section + `-coverappend`~~


### How I thought to approach the issue
         
Couple years ago I came across the article [Code Coverage for Solidity](https://blog.colony.io/code-coverage-for-solidity-eecfa88668c2/)~~, that explains how the author added code coverage support to Solidity programming language~~. The article was pretty insightful for me, as it gave enough of technical explanation and described typical challenges when implementing code coverage for particular programming language.  

The author based his implementation on [Instabul](https://istanbul.js.org/) coverage tool, initially targeted on JavaScript. But the author managed to generate the coverage profile for the other language (Solidity) in the format of the tool. Thus, he could reuse the reporting facility of Istanbul for free.

My plan was to use the similar approach. But my research have shown that their [coverage profile file format](https://github.com/gotwarlost/istanbul/blob/master/coverage.json.md) was not append-friendly. 
      
The other direction of my thought was defined by the classical book [The AWK Programming Language](https://archive.org/download/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) by Alfred V. Aho, Brian W. Kernighan, Peter J. Weinberger (the A., W., K. of AWK). By the way it's absolute pleasure to read and, amazingly, totally relevant, despite been published in 1988. ~~It's because the AWK language almost didn't change since then.~~

 ~~- The AWK Programming Language by A. W. K. pp. 167-169 - 7.2. Profiling~~ 

On pages 167-169 of this book there is a chapter "7.2. Profiling". It describes a very simple approach how one can implement the profiling of a program by just two AWK scripts `makeprof` and `printprof` of around 5 lines each (sic!). 

`makeprof` does (admittedly naive) transformation of input source by inserting `_LBcnt[i]++;` after each left brace `{`. Then it also adds the `END` clause that outputs the collected statement counts in `_LBcnt` to a file. 

`printprof` attaches the statement counts from this file to the original program.

I was completely fascinated by the simplicity and clarity of this approach! 

Despite it was far from perfect and practical, I really considered using some improved version of this approach. ~~I hoped it would be possible to make this completely in AWK.~~ However, it was clear that to make it more precise and robust, the AST-level transformation was needed, not just naive source code modification.   
    
#### Chosen implementation approach
                          
Working closely with AWK for my project one day I came across the GoAWK. Out of curiosity I tried running the Makesure with it. This revealed some bugs in GoAWK, that I've reported, and they were fixed promptly by Ben Hoyt, the creator. So eventually GoAWK passed the test suite of Makesure.

At that time I myself started getting interested in learning Golang. So I was happy to contribute myself some other minor bug I've encountered. Overall I really liked the project and especially the author behind it. I can't help but recommend [his technical blog](https://benhoyt.com/writings/) - very well written and interesting.

So I thought that whatever implementation approach I choose, I would like to use GoAWK as a base. Since GoAWK was written in Go, I decided to take a look on coverage story of Golang itself. This was when I came across [this article on Go's cover](https://go.dev/blog/cover). And it was instant hit! 

The functionality was not as advanced as with Istanbul, but still enough. 
The implementation seemed to be relatively simple. 
But most importantly, their coverage profile file format appears to be designed with appendability in mind! (Though, to my knowledge this feature is not directly used by Golang itself). 

So decided! We add code coverage functionality to GoAWK using the Golang's coverage profile file format. And then reusing the Golang machinery (`go tool cover`) to render final report.

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