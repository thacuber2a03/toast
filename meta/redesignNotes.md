- find a way *NOT* to "race the stack trace", i.e., count manually how deep in the stack is a function at any given time
	- maybe maintain my own stack? I'm told it's not a good idea but there's no other easy solution
	- also, fuck the `start`/`endCustom` functions
	- I might have just figured it out
		- still, gotta get custom functions working somehow

- as much as I liked it at first, the "immediate mode" part of this is just not working out anymore; better maintain each test separately and print them all afterwards
	- maybe not print them all afterwards, but supply the user with the results?
		- list of strings or list of TestResult structs that are formatted later?

- a friend suggested fuzzing; how the hell does that work?
	- "aggregation of test results from multiple files" that sounds like it'd benefit from a full separate executable (see next point)

- consider making a whole separate executable specialized on testing throughout different files
	- kinda like Lunar Modules' [busted](https://lunarmodules.github.io/busted/)

- test caching is a *bad* idea (maybe feature creep) but doesn't hurt to give it a go
	- the two hardest problems in programming are naming, cache invalidation and off-by-one errors

- maybe omit the mandatory return from assertion functions
	- so, like, no `if !assert(...) { return }` stuff
	- that would require turning off error checking past that point and ignoring any other incoming assertion functions
	- except for the ones that are *required* to pass, so like, the boolean return is a good idea, still, implement this too
