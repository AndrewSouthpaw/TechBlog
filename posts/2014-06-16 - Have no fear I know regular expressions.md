I know regular expressions, and at last get the joke to the XKCD comic:

![xkcd](https://lh6.googleusercontent.com/-KKa96Ws2OVE/U58fOw4jFFI/AAAAAAAAAz8/UhIjwBdEJs8/32aa7d697775eb67fda4e189cdbbf43e-2014-06-16-17-44.png)
( © Randall Monroe)

It does feel a little bit like I know kung fu. Regular expressions, which (essentially) search for a given pattern in a string, i.e. a set of characters (words, numbers, etc.), and then let you do something with the results. These powerful, albeit cryptic, expressions allow you to radically simplify many operations on strings.

Take, for instance, a problem presented on Coderbyte: given a string of words or letters, shift every letter over by 1 (so 'c' becomes 'd', 'z' becomes 'a'), and then capitalize any vowels excluding 'y'.

Here is a solution found sans regular expressions. Just gloss over the fine print for now and look at the size of it.

![num1](https://lh6.googleusercontent.com/-7LMXaa7_dlo/U58fOUbZyiI/AAAAAAAAAz0/Mpf7ecFD9YM/39ac6be34f3494e06e82e69372ae64b8-2014-06-16-17-44.png)

Here is a solution found using regular expressions (namely, mine).

![num2](https://lh6.googleusercontent.com/-pnRUO7qhxbs/U58fPc1ByUI/AAAAAAAAA0A/PZgh80EsqtQ/79fffbab2a1f790349a5671213a7a566-2014-06-16-17-44.png)

Slight difference, right? As an added benefit, regular expressions can often let you write my like how you would talk about solving the problem:

1. There's this function called 'LetterChanges,' you pass it a string.
2. Give me the result of taking all the alphabetic characters in the string and doing the following with each one:
3. If it's Z, change it to A
4. If it's z, change it to z
5. Otherwise shift the character over by one
7. Replace all the vowels with …
8. … an uppercase version

Neat, huh?