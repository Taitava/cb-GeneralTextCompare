# PatternCompare()
This is a string comparison function for [CoolBasic](https://www.coolbasic.com), ported from [a C++ function called *FastWildCompare()* by K. J. Krauss](http://developforperformance.com/MatchingWildcards_AnImprovedAlgorithmForBigData.html).

**Actual user manual is still under development!**



## Under the hood
This section discusses about certain decision details that I made when porting this function. They are not relevant for knowing how to use the function.

### Why Mid() is used so much
The original C++ function uses *array like* syntax to access specific characters in strings, e.g. `strWild[iWild]`. In CoolBasic, strings cannot be handled like arrays. I could have converted `strWild` and `strTame` to arrays, but that would have two downsides:
- All arrays in CoolBasic are global, meaning I would have to give them longer names like `PatternCompare_strWild`  and `PatternCompare_strTame` to avoid namespace collisions with user code.
- The original function relies on the fact that over-reading a string will result in a falsy value, but will not crash the program. In CoolBasic, over-reading an array will crash the program. I do not know the original algorithm well enough to be able to say when an over-reading might happen and when not, so applying boundary checks to every place where `strWild` and `strTame` are read, would have been an overkill.

With this in mind, I decided to read strings using CoolBasic's `Mid()` function everywhere. In general, it's used to get a substring from a string by defining the left starting point and the length of the substring. While it could be used to get a multiple characters long substring, in this project it's always used for getting just one character (or an empty string, in case of over-reading a string).

Calling `Mid()` is slower than reading arrays. But it's a trade-off that I am willing to accept, at least until I will come up with a use case that would really benefit from a better performance. Occasional text comparison with reasonably small strings should not take too much computing time.

Anyway, here is a simple test code that compares the speeds of reading characters using an array, versus using `Mid()`:

```
count = 1000000
stri$ = "g934ti34herguiiuerguiergui"

// Setup a helper array for Test 1
Dim stri_array$(0)
ReDim stri_array(Len(stri))
For i = 1 To Len(stri)
	stri_array(i) = Mid(stri, i,1)
Next i

// Test 1
t1 = Timer()
For i = 1 To count
	n = Rand(1,Len(stri))
	character$ = stri_array(n)
Next i
t1 = Timer()-t1

// Test 2
t2 = Timer()
For i = 1 To count
	n = Rand(1,Len(stri))
	character$ = Mid(stri, n,1)
Next i
t2 = Timer()-t2

// Results
MakeError "Array: "+t1+"ms, Mid(): "+t2+"ms"
```
With `count = 1000000` and `stri$ = "g934ti34herguiiuerguiergui"`, I got this result: *Array: 5781ms, Mid(): 10293ms*. 

The relative difference is remarkable in this example, but it narrows if you shorten `stri$`. With `count = 1000000` and `stri$ = "g"`, I got this result: *Array: 1371ms, Mid(): 1439ms*. But of course a string of only one character is an extreme example; usually comparison strings are longer.

If need be, `PatternCompare()` can be converted to use arrays, but this is the approach for now.

### Why not to store characters from `Mid()` to a variable?

For example:
```
Dim chrWild$, chrTame$
chrWild = Mid(strWild, iWild,1)
chrTame = Mid(strTame, iTame,1)
```
Then reading these variables in `If` and `While` statements would be cleaner.

I considered this approach first, and maybe it would have made some of the condition checkings a little bit cleaner to read, but it would have had consequences:
- Index variables `iWild` and `iTame` are changed in so many places, that updating the related character variables would become messy and hard to miss.
- It would have made the ported function look more different from the original one, making it harder to debug possible bugs by comparing the ported function to the original one.
- I doubt it would not give much performance benefits, considering that it would be needed to constantly update the character variables.


## Author
- [The original C++ function called *FastWildCompare()* is done by K. J. Krauss](http://developforperformance.com/MatchingWildcards_AnImprovedAlgorithmForBigData.html).
- Porting to CoolBasic and other modifications are done by me, Jarkko Linnanvirta.
- License details can be found in the same file with the function definition.