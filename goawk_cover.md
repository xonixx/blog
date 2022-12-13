
# Coverage for GoAWK
    
### TLDR

I contributed to [GoAWK project]() by implementing the code coverage functionality.

I'm grateful to [Ben Hoyt](https://benhoyt.com/) (the creater of GoAWK) for giving thorough and insightful code reviews and being patient.
      
### What is code coverage?

https://en.wikipedia.org/wiki/Code_coverage

### How the idea emerged, motivators

#### AWK

I myself been dedicated Python lover in past for many years now came to a conclusion that what can be scripted with AWK, should be scripted in AWK (over Python, Ruby, Perl, etc.). I'm not saying that you should write big apps though, but for small scripts AWK is absolutely fine alternative to major scripting languages with lots of benefits. Been universally available (as part of POSIX) and very compliant (language standard is almost unchanged for over 30 years now). As they say: "Good programmer chooses the most powerful tool for the job, the best programmer chooses the least powerful tool for the job"

- (take material from my AWK article)
- https://github.com/vladcc/shawk/blob/7420a88ce2025f3fe7390efb2b11e29d5b7b6b80/README.md#why-shell--awk

#### Makesure

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