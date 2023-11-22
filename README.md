
# GOTO Considered Beneficial
## _Using GOTO for better error handling in GoLang, PHP, etc._

Programmers have demonized[[1]](#footnotes) using GOTO for decades, and embraced using multiple early returns[[2]](#footnotes) for years. This essay seeks to illustrate **_how source code can be improved_** by using GOTO where most programmers would use early returns,

## Usage Pattern (shown in GoLang)

Here is a contrived example showing the usage pattern:

```golang
var (
  ErrCannotDoFunc = errors.New("cannot do func")
  ErrCannotDoAnotherFunc = errors.New("cannot do another func")
)

func example() (value Value, err error) {
	
  value,err = funcMightError()
  if err != nil {
    err = fmt.Errorf("%w; %s", ErrCannotDoFunc, err)
    goto finally
  }

  value,err = anotherFuncMightError()
  if err != nil {
    err = fmt.Errorf("%w; %s", ErrCannotDoAnotherFunc, err)
    goto finally
  }

finally:

  if err != nil {
    err = fmt.Errorf( "unable to do example: %w, err )
  }
  return value,err
}
```

And if works just as well when there is no cleanup routine:
```golang
var (
   ErrCannotDoFunc = errors.New("cannot do func")
   ErrCannotDoAnotherFunc = errors.New("cannot do another func")
)

func example() (value Value, err error) {

  value,err = funcMightError()
  if err != nil {
     err = fmt.Errorf( "unable to do example; %w; %s, ErrCannotDoFunc, err.Error() )
     goto finally
  }

  value,err = anotherFuncMightError()
  if err != nil {
     err = fmt.Errof( "unable to do example; %w;  %s, ErrCannotDoAnotherFunc, err.Error() )
     goto finally
  }

finally:

   return value,err
}
```

And here is another way we might write that same function _(Notice how we do work after the `break` but before the `return`):_


## Benefits

### Immediate
The immediate obvious benefits are:

1. Supports [_"The Happy_](https://medium.com/@matryer/line-of-sight-in-code-186dd7cdea88) [_Path,"_ e.g. left-aligning code](https://maelvls.dev/go-happy-line-of-sight/)
2. You can still use [guard clauses](https://wiki.c2.com/?GuardClause), but using `goto finally` instead of early `return` statements.
3. Needs only one breakpoint for debugging to ensure a halt of execution before the function returns.
4. Run shared cleanup code with `goto` _guaranteed_ to run no matter how the exit occurs.
5. Eliminates having to return many zeroed values when used with multiple return _values_ and exiting on error _(GoLang-specific, not PHP.)_   
6. And, of course, there is only ever one (1) exit point in your functions.

### Longer Term
The longer term benefits are less obvious, but experience reveals them to be even more valuable:

1. The pattern is easy ti learn, is very clear, is easy and _repeatable_.
2. When there are many `gotos` it is an _obvious indicator_ a function should be refactored into two or more functions.
3. When logic dictates you need a 2nd `goto` label it is a _clear indicator_ you need to refactor into another function.
4. When there are many variables you need to declare at the top, that is a _good indicator_ you need to refactor.
5. When refactoring, the logic does not need to be _restructured_, unlike when using early returns. 


## Shortcomings

1. Using `goto`might be _infinitesimally slower_ than an early `return` if you need to use a variable when the early return would not have required it. If every nanosecond matters in your specific executions by all means use early return, but don't [optimize prematurely](https://stackify.com/premature-optimization-evil/).

2. You might need to _explicitly declare variables_ in contexts where you would not have to with early returns. When this occurs, you'll need to do so before your first `goto finally`. OTOH one can argue declaring variables at the function top adds to readability for the future maintenance programmers and is not in-fact a shortcoming.


## Usage in github.com/golang/go
The `goto` statement [appears over 800 times](https://sourcegraph.com/search?q=context:global+repo:%5Egithub.com/golang/go%24+lang:Go+count:all+content:%5Cn%5Cs*goto%5Cs%2B%5BA-Za-z%5D%5BA-Za-z0-9-_%5D&patternType=regexp) in Go's source code that I analyzed. Here are just a handful of the files in which is can be found:

- [/src/cmd/compile/internal/ssa/regalloc.go#L1461-L1528](https://github.com/golang/go/blob/master/src/cmd/compile/internal/ssa/regalloc.go#L1461-L1528)
- [/src/cmd/compile/internal/syntax/scanner.go#L202-L320](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/scanner.go#L202-L320)
- [/src/cmd/compile/internal/typecheck/typecheck.go#L1297-L1420](https://github.com/golang/go/blob/master/src/cmd/compile/internal/typecheck/typecheck.go#L1297-L1420)
- [/src/cmd/compile/internal/types2/call.go#L470-L601](https://github.com/golang/go/blob/master/src/cmd/compile/internal/types2/call.go#L470-L601)
- [/src/cmd/compile/internal/types2/expr.go#L789-L832](https://github.com/golang/go/blob/master/src/cmd/compile/internal/types2/expr.go#L789-L832)
- [/src/cmd/compile/internal/types2/expr.go#L1287-L1696](https://github.com/golang/go/blob/master/src/cmd/compile/internal/types2/expr.go#L1287-L1696)
- [/src/cmd/internal/obj/x86/asm6.go#L3303-L5231](https://github.com/golang/go/blob/master/src/cmd/internal/obj/x86/asm6.go#L3303-L5231)
- [/src/debug/dwarf/type.go#L513-L853](https://github.com/golang/go/blob/master/src/debug/dwarf/type.go#L513-L853)
- [/src/go/build/build.go#L595-L774](https://github.com/golang/go/blob/master/src/go/build/build.go#L595-L774)
- [/src/go/types/expr.go#L766-L1663](https://github.com/golang/go/blob/master/src/go/types/expr.go#L766-L1663)
- [/src/net/http/h2_bundle.go#L3087-L3158](https://github.com/golang/go/blob/master/src/net/http/h2_bundle.go#L3087-L3158)


Those are but a few of the many places `goto` is used in Go source. 

And while the Go source does not use `goto` quite as often as this essay advocates, clearly The Go Authors find that `goto` is not harmful for their use-cases.

## Points of View

- [What led to "Notes on Structured Programming"](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD13xx/EWD1308.html) by Edsger W. Dijkstra, author of _"The goto statement considered harmful"_, where he states _(**emphasis** mine) "which in later years would be most frequently referenced, **regrettably**, however, often by authors who had seen no more of it than its title"_
- ["GOTO Considered Harmful" Considered Harmful](https://web.archive.org/web/20090320002214/http://www.ecn.purdue.edu/ParaMount/papers/rubin87goto.pdf) has frank Rubin arguing against the "GOTO Considered Harmful" meme and advocating for GOTO's use for making code less complex.
- [Structured Programming with GOTO Statements](https://web.archive.org/web/20131023061601/http://cs.sjsu.edu/~mak/CS185C/KnuthStructuredProgrammingGoTo.pdf) by the legendary Donald Knuth advocated for use of GOTO.
- [Video of Donald Knuth](https://www.youtube.com/watch?v=XWR5Y3Wf8Fo&t=2610s) on why he believes a properly used GOTO statement is much better than the alternative and where he says when you use GOTOs you should add a comment to _"See [Structured Programming with GOTO Statements](https://web.archive.org/web/20131023061601/http://cs.sjsu.edu/~mak/CS185C/KnuthStructuredProgrammingGoTo.pdf)"_
- [Multiple Return Statements on Wikipedia](https://en.wikipedia.org/wiki/Return_statement#Multiple_return_statements) points out _"The most common problem in early exit is that cleanup or final statements are not executed"_
- [Are there any legitimate use-cases for "goto" in a language that supports loops and functions?](https://stackoverflow.com/questions/24451/are-there-any-legitimate-use-cases-for-goto-in-a-language-that-supports-loops) provides several a highly up-voted answers — over 900 for the top answer - that dispel with the notion that GOTO is harmful when used appropriately.
- [Is using goto ever worthwhile?](https://softwareengineering.stackexchange.com/questions/566/is-using-goto-ever-worthwhile) provides clear use-cases where GOTO is useful.
- [Argument for using both Multiple and Single Returns](https://blog.jdriven.com/2021/10/single-return-vs-multiple-returns/), but as he is using Java which he does not consider using [`break` to label](https://www.geeksforgeeks.org/g-fact-64/) — which is the Java equivalent of goto — so I argue his assessment is incomplete.
- [Argument for Single Return](https://stschindler.medium.com/having-only-one-return-statement-is-better-and-more-safe-5cd379b7e87c#d8a5)
- [Claims that Single Exits points are a Fantasy](http://www.fredosaurus.com/JavaBasics/methods/method-commentary/methcom-30-multiple-return.html) but then again he is using goto-less Java.
- [Multiple Return Statements](http://onthethought.blogspot.com/2004/12/multiple-return-statements.html) makes some great points about Dijkstra's famous _"goto considered harmful"_ being _"massively misunderstood"_ and how it can _"be hard to see all the places where you exit a method if they are distributed throughout the method."_
- [Multiple return statements](http://www.javapractices.com/topic/TopicAction.do?Id=114) argues for multiple returns statements but points out _"This technique can be easily abused, and should be used with care. It's mainly used to replace nested if structures"_, and of course, this is a post showing goto-less Java code.
- [Multiple Return Statements](https://nipafx.dev/java-multiple-return-statements/) agruges for multiple returns statements but again, this is a post showing goto-less Java code.
- [What Makes Multiple Return Statements Bad Practice?](https://therenegadecoder.com/code/what-makes-multiple-return-statements-bad-practice/) argues that multiple returns statements are not in-fact bad practice, but does so by strawmanning the arguements for single return statements, i.e. _"Finally, I noticed that many of the arguments for the SESE principle use examples that are convoluted. In many cases, the examples have more issues than multiple return statements."_ yet none of the examples he cites introduce the use of GOTOs.
- [Return Early, Return Often](https://medium.com/@pypmannetjies/return-early-return-often-f46f2f940c3d) argues for multiple early returns, but it appears her examples are in goto-less JavaScript.
- [Coding Tip: Have A Single Exit Point](https://www.tomdalling.com/blog/coding-tips/coding-tip-have-a-single-exit-point/) argues for having a single exit point because _"multiple exit points are bad is that they complicate control flow."_
- [Where did the notion of "one return only" come from?](https://softwareengineering.stackexchange.com/questions/118703/where-did-the-notion-of-one-return-only-come-from) has [an answer](https://softwareengineering.stackexchange.com/a/118771/9114) that makes the point that _"single return statements make logging easier, as well as forms of debugging that rely on logging"_ whereas [this answer](https://softwareengineering.stackexchange.com/a/359224/9114) points out that _"One return makes refactoring easier"_ and [this answer](https://softwareengineering.stackexchange.com/a/118811/9114) points out, ironically, that _"the fact that multiple return statements are equivalent to having GOTO's to a single return statement"_ and _"I don't consider these types of GOTO's harmful."_
- [Religious War #48,293: Single Vs. Multiple Returns](https://hackerchick.com/religious-war-48293-single-vs-multiple/) argues for multiple returns by asserting _" if you’re setting a value and you choose – even after setting the appropriate value for this scenario – to keep processing, you risk inadvertently changing the value from the right one to an incorrect one"_ but she does not contemplate using a GOTO which would address her concern.
- [Single-Entry, Single-Exit, Should It Still Be Applicable In Object-oriented Languages?](https://blogs.msmvps.com/peterritchie/2008/03/07/single-entry-single-exit-should-it-still-be-applicable-in-object-oriented-languages/) argues for multiple returns for C#, but nowhere does he or any of his commenters mention GOTOs _(except for me.)_
- [Single Function Exit Point](http://wiki.c2.com/?SingleFunctionExitPoint) is a discussion with too many perspectives to summarize.
- [Goto Statement In C#](https://www.c-sharpcorner.com/UploadFile/hirendra_singh/goto-statement-in-C-Sharp/) shows how to use `goto` with a `cleanup:` label.
- [Why Many Return Statements Are a Bad Idea in OOP](https://www.yegor256.com/2015/08/18/multiple-return-statements-in-oop.html) seemed to be about the topic — in case someone reading this finds it — but it goes down a rabbit hole of little value.
- [The Single Return Law](https://www.anthonysteele.co.uk/TheSingleReturnLaw) makes the argument that calling something a  _"law"_ is a high-bar and that single returns do not meet that bar which is a strawman argue; who besides the author called is a _"law?"_ The author further states _"In any event, a function with multiple exit points is a far lesser issue than a goto"_ while linking to Dijkstra's  _"The goto statement considered harmful"_ with no other argument against GOTO, this being the type of thing Dijkstra found lamentable. He claims that explicitly declaring a return variable _"worse"_ than just having a return; a position that is clearly opinion because my opinion is having an explicitly declared return variable is better for numerous reasons including refactorability, self-documenting, etc. He clearly dislikes needing a return variable, OTOH I dislike having functions that don't have a single place to breakpoint when debugging to ensure the debugger stops before leaving the function. _(IDEs could fix this issue, but they haven't, AFAIK.)_ 
- [Evolutionary Design — A Conversation with Martin Fowler, Part III]() is not a discussion about returns, but Martin Fowler makes the following point which IMO argues for the importance of the single return statement during breakpoint debugging: _"People underestimate the time they spend debugging. They underestimate how much time they can spend chasing a long bug. ... There are few things more frustrating or time wasting than debugging."_
- [Is there a "goto" statement in bash?](https://stackoverflow.com/questions/9639103/is-there-a-goto-statement-in-bash) has several highly upvoted comments where the commenters wrote _"He may have his reasons (for wanting GOTO)"_, _"I'm sick of this goto myth! There's nothing wrong with goto!"_, and _""Avoid goto" is a great rule. ... Third, learn good exceptions to the rule, based on a comprehensive understanding of how to follow the rule and the reasons for the rule"_

## Footnotes
1. Edgar Dijkstra wrote the seminal "[_Go To Statement Considered Harmful_](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf)" which was actually titled by Niklaus Wirth and which Dijkstra later wrote _(**emphasis** mine) "In 1968, the Communications of the ACM published a text of mine under the title "The goto statement considered harmful", which in later years would be most frequently referenced, **regrettably**, however, often by authors who had seen no more of it than its title."_
2. One of the main arguments for early returns was popularized by Martin Fowler and is using [guard clauses](https://refactoring.com/catalog/replaceNestedConditionalWithGuardClauses.html), which this author believes are a great idea that can be conceptually decoupled from early returns. Unfortunately Java does not support GOTO and thus when talking about Java, Martin Fowlers frequent example language, using guard clauses means early returns. However, for languages like Go and PHP that do support GOTO, guard clauses need to require early returns.  
