# DSA — Strings

Strings are one of the highest-frequency DSA topics in Java interviews. Many string problems are really array, hashing, two-pointer, sliding-window, stack, or frequency-counting problems applied to characters.

---

## 1. What is a String in Java?

`String` is an immutable sequence of characters.

```java
String name = "Sudhir";
```

Important:

```text
String → immutable
StringBuilder → mutable
StringBuffer → mutable and synchronized
```

---

## 2. String Indexing

```java
String s = "Java";

System.out.println(s.charAt(0)); // J
System.out.println(s.charAt(3)); // a
```

Valid indexes:

```text
0 to s.length() - 1
```

---

## 3. Basic String Traversal

```java
String s = "Java";

for (int i = 0; i < s.length(); i++) {
    System.out.println(s.charAt(i));
}
```

Enhanced traversal is also possible through character arrays:

```java
for (char c : s.toCharArray()) {
    System.out.println(c);
}
```

---

## 4. Reverse a String

Using `StringBuilder`:

```java
String reversed =
    new StringBuilder(s)
        .reverse()
        .toString();
```

Manual approach:

```java
char[] chars = s.toCharArray();

int left = 0;
int right = chars.length - 1;

while (left < right) {
    char temp = chars[left];
    chars[left] = chars[right];
    chars[right] = temp;

    left++;
    right--;
}

String reversed =
    new String(chars);
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

The character array requires additional storage.

---

# 5. Check if a String is a Palindrome

A palindrome reads the same forward and backward.

Example:

```text
madam → palindrome
java  → not palindrome
```

Two-pointer approach:

```java
int left = 0;
int right = s.length() - 1;

boolean palindrome = true;

while (left < right) {
    if (s.charAt(left) != s.charAt(right)) {
        palindrome = false;
        break;
    }

    left++;
    right--;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 6. Case-Insensitive Palindrome

A simple approach:

```java
String normalized =
    s.toLowerCase(Locale.ROOT);
```

Then use the two-pointer approach.

If the problem also says to ignore punctuation and spaces, normalize those characters according to the problem's exact requirements.

---

# 7. Count Characters

For lowercase English letters:

```java
int[] frequency = new int[26];

for (char c : s.toCharArray()) {
    frequency[c - 'a']++;
}
```

Example:

```text
Input:
banana

Frequency:
a → 3
b → 1
n → 2
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

The space is O(1) because the array always has 26 positions.

---

# 8. Character Frequency with HashMap

Use a HashMap when the character set is not limited to lowercase English letters.

```java
Map<Character, Integer> frequency =
    new HashMap<>();

for (char c : s.toCharArray()) {
    frequency.merge(
        c,
        1,
        Integer::sum
    );
}
```

This can handle a wider range of characters than a fixed 26-element array.

---

# 9. First Non-Repeating Character

Approach:

1. Count frequencies.
2. Traverse again.
3. Return the first character with frequency `1`.

```java
Map<Character, Integer> frequency =
    new HashMap<>();

for (char c : s.toCharArray()) {
    frequency.merge(
        c,
        1,
        Integer::sum
    );
}

for (char c : s.toCharArray()) {
    if (frequency.get(c) == 1) {
        return c;
    }
}

return '\0';
```

### Complexity

```text
Time:  O(n)
Space: O(k)
```

where `k` is the number of distinct characters.

---

# 10. Check if Two Strings are Anagrams

Two strings are anagrams if they contain the same characters with the same frequencies.

Example:

```text
listen
silent
```

### Frequency-array approach

```java
if (s.length() != t.length()) {
    return false;
}

int[] frequency = new int[26];

for (int i = 0; i < s.length(); i++) {
    frequency[s.charAt(i) - 'a']++;
    frequency[t.charAt(i) - 'a']--;
}

for (int count : frequency) {
    if (count != 0) {
        return false;
    }
}

return true;
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

Assumes lowercase English letters.

---

# 11. Anagram Using Sorting

Another approach:

```java
char[] a = s.toCharArray();
char[] b = t.toCharArray();

Arrays.sort(a);
Arrays.sort(b);

return Arrays.equals(a, b);
```

### Complexity

```text
Time:  O(n log n)
Space: depends on implementation
```

Frequency counting is generally more efficient when the character domain is known.

---

# 12. Remove Duplicate Characters

Example:

```text
Input:
programming

Output:
progamin
```

Using a `LinkedHashSet`:

```java
Set<Character> set =
    new LinkedHashSet<>();

for (char c : s.toCharArray()) {
    set.add(c);
}

StringBuilder result =
    new StringBuilder();

for (char c : set) {
    result.append(c);
}
```

`LinkedHashSet` preserves insertion order.

---

# 13. Check if All Characters Are Unique

Using a boolean array for lowercase letters:

```java
boolean[] seen =
    new boolean[26];

for (char c : s.toCharArray()) {
    int index = c - 'a';

    if (seen[index]) {
        return false;
    }

    seen[index] = true;
}

return true;
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 14. Longest Common Prefix

Example:

```text
flower
flow
flight
```

Output:

```text
fl
```

Vertical scanning:

```java
String prefix = strs[0];

for (int i = 1; i < strs.length; i++) {
    while (!strs[i].startsWith(prefix)) {
        prefix =
            prefix.substring(
                0,
                prefix.length() - 1
            );

        if (prefix.isEmpty()) {
            return "";
        }
    }
}

return prefix;
```

### Interview idea

Start with the first string as a candidate and keep reducing the prefix until every string matches it.

---

# 15. String Compression

Example:

```text
Input:
aaabbc

Output:
a3b2c1
```

Use a run-length approach:

```java
StringBuilder result =
    new StringBuilder();

int count = 1;

for (int i = 1; i <= s.length(); i++) {

    if (i < s.length()
            && s.charAt(i) == s.charAt(i - 1)) {

        count++;

    } else {

        result.append(
            s.charAt(i - 1)
        );

        result.append(count);

        count = 1;
    }
}
```

---

# 16. Reverse Words in a String

Example:

```text
Input:
"the sky is blue"

Output:
"blue is sky the"
```

Simple approach:

```java
String[] words =
    s.trim().split("\\s+");

StringBuilder result =
    new StringBuilder();

for (int i = words.length - 1;
     i >= 0;
     i--) {

    result.append(words[i]);

    if (i != 0) {
        result.append(" ");
    }
}

return result.toString();
```

---

# 17. String Rotation

A string `s` is a rotation of `goal` if:

```java
s.length() == goal.length()
```

and:

```java
(s + s).contains(goal)
```

Example:

```text
s = "abcde"
goal = "cdeab"
```

Since:

```text
abcdeabcde
```

contains:

```text
cdeab
```

the strings are rotations.

---

# 18. Valid Parentheses

Although this is a string problem, the correct data structure is usually a stack.

```java
Deque<Character> stack =
    new ArrayDeque<>();

for (char c : s.toCharArray()) {

    if (c == '(' ||
        c == '{' ||
        c == '[') {

        stack.push(c);

    } else {

        if (stack.isEmpty()) {
            return false;
        }

        char top = stack.pop();

        if ((c == ')' && top != '(') ||
            (c == '}' && top != '{') ||
            (c == ']' && top != '[')) {

            return false;
        }
    }
}

return stack.isEmpty();
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 19. Longest Substring Without Repeating Characters

This is one of the most important sliding-window string problems.

Example:

```text
abcabcbb
```

Answer:

```text
3
```

because:

```text
abc
```

is the longest substring without duplicate characters.

Using a HashSet:

```java
Set<Character> set =
    new HashSet<>();

int left = 0;
int maximum = 0;

for (int right = 0;
     right < s.length();
     right++) {

    while (set.contains(
            s.charAt(right))) {

        set.remove(
            s.charAt(left)
        );

        left++;
    }

    set.add(s.charAt(right));

    maximum = Math.max(
        maximum,
        right - left + 1
    );
}

return maximum;
```

### Complexity

```text
Time:  O(n)
Space: O(k)
```

---

# 20. Longest Substring Without Repeating Characters — Optimized

Instead of removing characters one by one, store their latest index.

```java
Map<Character, Integer> lastSeen =
    new HashMap<>();

int left = 0;
int maximum = 0;

for (int right = 0;
     right < s.length();
     right++) {

    char c = s.charAt(right);

    if (lastSeen.containsKey(c)) {
        left = Math.max(
            left,
            lastSeen.get(c) + 1
        );
    }

    lastSeen.put(c, right);

    maximum = Math.max(
        maximum,
        right - left + 1
    );
}

return maximum;
```

### Important

This is a classic **sliding window** pattern.

---

# 21. Longest Substring with At Most K Distinct Characters

Maintain:

```text
left
right
frequency map
distinct count
```

When the number of distinct characters exceeds `k`, shrink from the left.

Conceptual pattern:

```java
Map<Character, Integer> frequency =
    new HashMap<>();

int left = 0;
int maximum = 0;

for (int right = 0;
     right < s.length();
     right++) {

    char c = s.charAt(right);

    frequency.merge(
        c,
        1,
        Integer::sum
    );

    while (frequency.size() > k) {

        char leftChar =
            s.charAt(left++);

        frequency.merge(
            leftChar,
            -1,
            Integer::sum
        );

        if (frequency.get(leftChar) == 0) {
            frequency.remove(leftChar);
        }
    }

    maximum = Math.max(
        maximum,
        right - left + 1
    );
}
```

---

# 22. Longest Repeating Character Replacement

A common sliding-window problem.

For a window:

```text
window length - most frequent character count
```

represents the number of replacements required.

If replacements exceed `k`, shrink the window.

Key idea:

```text
windowSize - maxFrequency <= k
```

This is a very important interview pattern.

---

# 23. Minimum Window Substring

Given strings `s` and `t`, find the smallest substring of `s` containing all characters of `t`.

This is an advanced sliding-window problem.

Typical approach:

```text
1. Count required characters.
2. Expand right.
3. Track characters in current window.
4. Once valid, shrink left.
5. Keep the smallest valid window.
```

Complexity can be:

```text
Time:  O(n)
Space: O(k)
```

where `k` represents the relevant character set.

---

# 24. Group Anagrams

Given:

```text
["eat", "tea", "tan", "ate", "nat", "bat"]
```

Group:

```text
["eat", "tea", "ate"]
["tan", "nat"]
["bat"]
```

A common approach is to use the sorted string as a key:

```java
Map<String, List<String>> groups =
    new HashMap<>();

for (String word : words) {

    char[] chars =
        word.toCharArray();

    Arrays.sort(chars);

    String key =
        new String(chars);

    groups
        .computeIfAbsent(
            key,
            k -> new ArrayList<>()
        )
        .add(word);
}
```

### Complexity

If average word length is `m`:

```text
Time: O(n * m log m)
```

A frequency-vector key can improve this when the alphabet is fixed.

---

# 25. Isomorphic Strings

Two strings are isomorphic if characters in one string can be mapped one-to-one to characters in the other.

Example:

```text
egg
add
```

is valid.

But:

```text
foo
bar
```

is not.

Use two mappings:

```text
s → t
t → s
```

This prevents two different characters from mapping to the same character.

---

# 26. Word Pattern

Example:

```text
pattern = "abba"
sentence = "dog cat cat dog"
```

This matches.

Use a bidirectional mapping:

```text
character → word
word → character
```

The same idea appears in isomorphic-string problems.

---

# 27. Substring vs Subsequence

### Substring

Must be contiguous.

```text
String:
abcdef

Substring:
bcd
```

### Subsequence

Does not have to be contiguous, but relative order is preserved.

```text
ace
```

is a subsequence of:

```text
abcde
```

This distinction is extremely important.

---

# 28. Check if One String Is a Subsequence of Another

```java
int i = 0;
int j = 0;

while (i < s.length()
        && j < t.length()) {

    if (s.charAt(i) == t.charAt(j)) {
        i++;
    }

    j++;
}

return i == s.length();
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 29. Find All Anagrams in a String

Use a sliding window with character frequencies.

Conceptually:

```text
Target:
abc

Window:
length = 3

Move window one character at a time.
```

At each step compare the required frequency state with the window frequency state.

For lowercase English letters, fixed-size arrays are efficient.

---

# 30. Permutation in String

Determine whether one string's permutation occurs as a substring of another.

Again:

```text
Fixed-size sliding window
+
Character frequency
```

is the key pattern.

---

# 31. Count Palindromic Substrings

A straightforward approach is **expand around center**.

Every palindrome has a center:

```text
odd length:
    center is a character

even length:
    center is between two characters
```

For each possible center:

```text
expand left
expand right
while characters match
```

### Complexity

```text
Time:  O(n²)
Space: O(1)
```

---

# 32. Longest Palindromic Substring

Also commonly solved using expand-around-center.

```text
For every character:
    expand around (i, i)

For every adjacent pair:
    expand around (i, i + 1)
```

Track the longest palindrome found.

### Complexity

```text
Time:  O(n²)
Space: O(1)
```

There are more advanced approaches such as Manacher's algorithm, but the center-expansion method is usually the first interview solution to know.

---

# 33. String Hashing

String hashing maps a string to a numeric representation.

It can be useful for:

```text
Comparing substrings
Pattern matching
Deduplication
Hash-based algorithms
```

Be careful:

```text
Hash collision ≠ string equality
```

A hash is not automatically a proof that two strings are identical.

---

# 34. KMP — Knuth-Morris-Pratt

KMP is an efficient string pattern-matching algorithm.

It preprocesses the pattern using the:

```text
LPS array
```

LPS means:

```text
Longest Proper Prefix
which is also
Suffix
```

It avoids repeatedly moving backward in the text.

### Complexity

```text
Time:  O(n + m)
Space: O(m)
```

where:

```text
n = text length
m = pattern length
```

---

# 35. KMP Interview Level

For most Java backend interviews:

```text
Understand the idea
Understand LPS
Know the O(n + m) complexity
Be able to implement it if DSA-heavy
```

You do not need to memorize the implementation without understanding why it works.

---

# 36. Trie

A Trie is a tree-like data structure used for strings.

Useful for:

```text
Prefix search
Autocomplete
Dictionary lookup
Word search
```

Example:

```text
cat
car
can
```

share:

```text
c
└── a
    ├── t
    ├── r
    └── n
```

Typical operations:

```text
Insert → O(L)
Search → O(L)
Prefix search → O(L)
```

where `L` is the word length.

---

# 37. Java Trie Node

A basic Trie node:

```java
class TrieNode {

    TrieNode[] children =
        new TrieNode[26];

    boolean isWord;
}
```

Insert:

```java
void insert(String word) {

    TrieNode current = root;

    for (char c : word.toCharArray()) {

        int index = c - 'a';

        if (current.children[index] == null) {
            current.children[index] =
                new TrieNode();
        }

        current =
            current.children[index];
    }

    current.isWord = true;
}
```

---

# 38. Common String Patterns

When you see a string problem, ask:

```text
1. Character frequency?
   → Array / HashMap

2. Same characters?
   → Frequency / Sorting

3. Contiguous substring?
   → Sliding Window

4. Palindrome?
   → Two Pointers / Expand Around Center

5. Matching parentheses?
   → Stack

6. Prefix-related?
   → Trie

7. Pattern matching?
   → KMP / Z Algorithm

8. Mapping characters?
   → HashMap

9. Longest/shortest valid substring?
   → Sliding Window

10. Need all combinations?
    → Backtracking
```

---

# 39. Important Java String Methods

Know these well:

```java
s.length();
```

```java
s.charAt(index);
```

```java
s.substring(begin);
```

```java
s.substring(begin, end);
```

```java
s.indexOf("abc");
```

```java
s.lastIndexOf("a");
```

```java
s.contains("java");
```

```java
s.startsWith("Ja");
```

```java
s.endsWith("va");
```

```java
s.equals(other);
```

```java
s.equalsIgnoreCase(other);
```

```java
s.toLowerCase(Locale.ROOT);
```

```java
s.toUpperCase(Locale.ROOT);
```

```java
s.trim();
```

```java
s.isBlank();
```

```java
s.split("\\s+");
```

```java
s.toCharArray();
```

---

# 40. `==` vs `equals()` for Strings

Never use:

```java
if (s1 == s2)
```

to test string content equality.

Use:

```java
if (s1.equals(s2))
```

`==` compares references.

`equals()` compares content for `String`.

---

# 41. Safe String Comparison

If a value might be null:

```java
"ACTIVE".equals(status)
```

is safer than:

```java
status.equals("ACTIVE")
```

because the first form does not throw if `status` is null.

---

# 42. StringBuilder Best Practices

Instead of:

```java
String result = "";

for (...) {
    result += value;
}
```

prefer:

```java
StringBuilder result =
    new StringBuilder();

for (...) {
    result.append(value);
}
```

Then:

```java
return result.toString();
```

This is especially important inside loops with many concatenations.

---

# 43. StringBuilder vs StringBuffer

### StringBuilder

```text
Mutable
Not synchronized
Usually preferred for local/single-threaded string construction
```

### StringBuffer

```text
Mutable
Synchronized
Usually unnecessary for modern local string construction
```

---

# 44. Unicode Consideration

A Java `char` is a UTF-16 code unit, not necessarily a complete Unicode character.

For basic ASCII interview problems:

```java
char
```

is usually sufficient.

For full Unicode code-point processing, APIs such as:

```java
codePointAt()
codePoints()
```

may be more appropriate.

This distinction matters in production internationalized systems.

---

# 45. Complexity Summary

| Problem | Typical Approach | Time |
|---|---|---:|
| Reverse String | Two pointers | O(n) |
| Palindrome | Two pointers | O(n) |
| Character frequency | Array / HashMap | O(n) |
| Anagram | Frequency | O(n) |
| First unique character | Frequency + scan | O(n) |
| Longest unique substring | Sliding window | O(n) |
| Group Anagrams | Hashing | O(n·m log m)* |
| Longest palindrome | Center expansion | O(n²) |
| Valid Parentheses | Stack | O(n) |
| Longest Common Prefix | Scanning | O(total input) |
| KMP | Pattern matching | O(n + m) |
| Trie search | Trie | O(L) |

`*` When sorted strings are used as keys.

---

# 46. Interview Questions — Easy

1. Reverse a string.
2. Check whether a string is a palindrome.
3. Count vowels and consonants.
4. Count character frequencies.
5. Find the first non-repeating character.
6. Find duplicate characters.
7. Remove duplicate characters.
8. Check whether two strings are anagrams.
9. Check whether a string contains only unique characters.
10. Reverse words in a string.

---

# 47. Interview Questions — Medium

11. Longest substring without repeating characters.
12. Longest common prefix.
13. Group anagrams.
14. Valid parentheses.
15. String compression.
16. String rotation.
17. Isomorphic strings.
18. Word pattern.
19. Find all anagrams in a string.
20. Permutation in string.
21. Longest substring with at most K distinct characters.
22. Longest repeating character replacement.
23. Count palindromic substrings.
24. Longest palindromic substring.
25. Minimum window substring.

---

# 48. Interview Questions — Advanced

26. Implement KMP.
27. Build a Trie.
28. Word Search II using Trie.
29. Implement substring search.
30. Shortest palindrome.
31. Palindrome partitioning.
32. Regular expression matching.
33. Wildcard matching.
34. Edit distance.
35. Longest common subsequence.

---

# 49. Common Mistakes

### Mistake 1 — Using `==`

```java
s1 == s2
```

does not compare string contents.

Use:

```java
s1.equals(s2)
```

---

### Mistake 2 — Ignoring case requirements

Check whether the problem expects:

```text
case-sensitive
```

or:

```text
case-insensitive
```

---

### Mistake 3 — Ignoring spaces

For problems involving sentences, clarify whether:

```text
spaces
punctuation
special characters
```

should count.

---

### Mistake 4 — Using HashMap when a fixed array is enough

For lowercase English letters:

```java
int[] frequency = new int[26];
```

is often simpler and faster.

---

### Mistake 5 — Building strings with `+` inside large loops

Prefer:

```java
StringBuilder
```

---

### Mistake 6 — Forgetting empty strings

Always consider:

```text
""
```

and:

```text
single-character strings
```

---

# 50. String Problem-Solving Checklist

Before coding:

```text
□ Is the problem about characters or words?
□ Is case significant?
□ Are spaces significant?
□ Is punctuation significant?
□ Is Unicode relevant?
□ Is the problem about a substring?
□ Is it about a subsequence?
□ Is frequency important?
□ Is a sliding window possible?
□ Is a stack needed?
□ Is a Trie useful?
□ What are the constraints?
□ What is the target complexity?
```

---

# 51. Quick Revision

```text
Strings
│
├── Basic
│   ├── Traversal
│   ├── Reverse
│   ├── Palindrome
│   └── Character access
│
├── Hashing
│   ├── Frequency
│   ├── Anagrams
│   ├── Isomorphic Strings
│   └── First Unique Character
│
├── Two Pointers
│   ├── Palindrome
│   └── String comparison
│
├── Sliding Window
│   ├── Longest Unique Substring
│   ├── K Distinct Characters
│   ├── Character Replacement
│   └── Minimum Window
│
├── Stack
│   └── Valid Parentheses
│
├── Palindrome
│   ├── Center Expansion
│   └── Palindrome Partitioning
│
├── Trie
│   ├── Insert
│   ├── Search
│   └── Prefix Search
│
└── Pattern Matching
    ├── KMP
    └── Z Algorithm
```

---

## Interview Rule

> **For string problems, don't immediately think "String." Think in patterns: frequency counting, hashing, two pointers, sliding window, stack, Trie, or pattern matching. The string is usually just the input format.**
