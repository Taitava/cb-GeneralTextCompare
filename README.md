# PatternCompare()
This is a string comparison function for [CoolBasic](https://www.coolbasic.com), ported from [a C++ function called *FastWildCompare()* by K. J. Krauss](http://developforperformance.com/MatchingWildcards_AnImprovedAlgorithmForBigData.html).

`PatternCompare()` is a function that can be used to check if a string matches a certain *pattern*. A pattern can consist of *regular characters* and *wildcard characters*. `*` and `?` are wildcard characters. The former can match with any number of characters (including no characters at all). The latter can only match with one character. You can change the wildcard characters to be something else than `*`/`?`.

Examples:
- Pattern  `"a*c"` matches `"abc"`.
- Pattern  `"a?c"` matches `"abc"`.
- Pattern  `"a*c"` matches `"azxc"` too.
- Pattern `"a?c"`  does not match `"azxc"` because between `a` and `c` there are two characters instead of one character.
- Pattern `"*"` matches `""`.
- Pattern `"abc*"` matches `"abc"`.
- Pattern `"?"` does not match `""`, because it explicitly requires one character.
- Pattern `"abc?"` does not match `"abc"`.
- Pattern `"ABC"` matches or does not match `"abc"`, depending how you want letter case differences be handled.

## Usage
1. Include the `PatternCompare()` function to your CoolBasic program with the following command: `Include "PatternCompare\PatternCompare.cb"`. (You do not need to include the unit test file located in the `tests` folder).
2. Call the function like this: `PatternCompare(strWild$,strTame$, case_sensitive=1, wildcards$="*?")`. Parameters:
	- `strWild$`: A pattern string. This can contain wildcards.
	- `strTame$`: A comparison string. This is interpreted completely literally, meaning that if it happens to contain wildcards, they do not have any special meaning, they need to match the same way as other characters in this string.
	- `case_sensitive=1`: If `True`, letters *A* and *a* (etc.) are considered different. Optional and defaults to `True`.
	- `wildcards$="*?"`: A two-character string where the first character defines a *multi character wildcard* and the second character defines a *single character wildcard*. You can change the wildcards if you need to use `*` or `?` in your `strWild$` (pattern) string literally. Optional and defaults to `*?*`.
3. The function returns either `True` (the strings matched) or `False` (the strings didn't match). The function can also trigger `MakeError`, but only if you set a bad value to `wildcards$`.

## Modifying and contributing
You can modify the function and create [pull requests](https://github.com/Taitava/cb-PatternCompare/pulls) to discuss about merging your changes to this repository. Just a few things to consider if you are going to modify the function:
- Write to `PatternCompare.cb` comments about what you have changed. Git commit messages are not enough, because the original C++ function's license requires that modifications need to be documented. Write the same information to the [Author](#Author) section of this readme document.
- Write some unit tests that test that the feature you have implemented works the way you mean it to work. Even if it's a simple feature/change. Unit tests can be run with [cbUnit](https://github.com/Taitava/cbUnit). You can also ask me for help with creating unit tests.
- Discuss your modification idea in the [issues](https://github.com/Taitava/cb-PatternCompare/issues) before creating a pull request.

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