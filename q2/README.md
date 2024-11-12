# Question 2: Anonymizing Names

This question assumes you are using Python, but you are free to use your preferred programming language.

## 2a
 
Write a Python function that accepts a string as input and replaces all occurrences of names with the string "ANON.”  In the function’s docstring, list at least three test strings and the expected output. **Assume names are capitalized words and may be separated by spaces or by punctuation. No other words are capitalized.**
 
## 2b

Suppose instead that you cannot make any assumptions about a name’s capitalization or the capitalization of other words. In other words, suppose that you need to write a Python function to anonymize an arbitrary string of text by replacing all names with the string “ANON.” Briefly describe 2-3 heuristics you might use to identify names in an arbitrary string of text. (No approach will be perfect! Simple solutions are OK.)

## Additional Notes

 If you'd like, use this space to include thoughts, questions, or documentation you'd like to supplement your code.

Notes:
- 2a: 
    - Based on punctuation only incuding periods, dashes, and apostrophes as these are the most commonly seen in names
- 2b:
    - Heuristics:
        - 1: Proper nouns like names will usually not follow the words "the", "a", "an" and usually comes after prepositions (ex: with, towards). Places usually also come after prepositions which will be exluded.
        - 2: Usually names appear as the first word in the sentence/text if it does not start with "A", "The", "An", "But", "Also", etc
        - 3: A name followed by the word "and" (ex: Ann and ...) likely the word following this will be a name as well as long it is not a pronoun (ex: "his", "hers", "him", "her", etc.)
    - Heuristics made based on the provided examples in "partb_test_strings".