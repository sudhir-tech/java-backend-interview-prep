# DSA — Trie

A **Trie**, also called a **Prefix Tree**, is a tree-based data structure designed for storing and searching strings efficiently.

Tries are especially useful for:

- Prefix searches
- Autocomplete
- Dictionary lookup
- Spell checking
- Search suggestions
- Word games
- IP routing
- String matching
- Word Search II
- Prefix-based filtering

---

# 1. What is a Trie?

A Trie stores characters along paths.

Example words:

```text
cat
car
care
dog
```

A simplified Trie looks like:

```text
        root
       /    \
      c      d
      |      |
      a      o
     / \     |
    t   r    g
        |
        e
```

Words sharing a prefix share the same path.

---

# 2. Why Use a Trie?

Suppose we have:

```text
apple
application
apply
app
```

All words share:

```text
app
```

A Trie stores this shared prefix only once.

This makes prefix operations efficient.

---

# 3. Trie Node

For lowercase English letters, a node can contain:

```java
class TrieNode {

    TrieNode[] children =
        new TrieNode[26];

    boolean isWord;
}
```

Each index represents:

```text
'a' → 0
'b' → 1
...
'z' → 25
```

---

# 4. Trie Class

```java
class Trie {

    private final TrieNode root =
        new TrieNode();

    // methods
}
```

The root does not represent an actual character.

It represents:

```text
start of all words
```

---

# 5. Insert a Word

To insert:

```text
cat
```

follow:

```text
root
↓
c
↓
a
↓
t
```

At `t`:

```java
isWord = true;
```

---

# 6. Trie Insert — Java

```java
class Trie {

    private final TrieNode root =
        new TrieNode();

    public void insert(
            String word) {

        TrieNode current = root;

        for (char ch :
                word.toCharArray()) {

            int index =
                ch - 'a';

            if (current.children[index]
                    == null) {

                current.children[index] =
                    new TrieNode();
            }

            current =
                current.children[index];
        }

        current.isWord = true;
    }
}

class TrieNode {

    TrieNode[] children =
        new TrieNode[26];

    boolean isWord;
}
```

---

# 7. Search a Word

To search:

```text
cat
```

follow the same path.

At the final node:

```java
isWord == true
```

means the complete word exists.

---

# 8. Trie Search — Java

```java
public boolean search(
        String word) {

    TrieNode node =
        findNode(word);

    return node != null
        && node.isWord;
}

private TrieNode findNode(
        String word) {

    TrieNode current = root;

    for (char ch :
            word.toCharArray()) {

        int index =
            ch - 'a';

        if (current.children[index]
                == null) {

            return null;
        }

        current =
            current.children[index];
    }

    return current;
}
```

---

# 9. Search vs Prefix Search

These are different operations.

### Search

Does the exact word exist?

```text
car → true
```

### Prefix Search

Does any word start with this prefix?

```text
ca → true
```

Even if:

```text
ca
```

is not itself a complete word.

---

# 10. Starts With

```java
public boolean startsWith(
        String prefix) {

    return findNode(prefix)
        != null;
}
```

Complexity:

```text
O(L)
```

where `L` is the length of the searched word/prefix.

---

# 11. Trie Complexity

For a word of length:

```text
L
```

### Insert

```text
O(L)
```

### Search

```text
O(L)
```

### Prefix Search

```text
O(L)
```

This is independent of the number of stored words in terms of traversal length.

---

# 12. Trie Space Complexity

For:

```text
N words
```

with total character count:

```text
C
```

Trie space is approximately:

```text
O(C × alphabet factor)
```

for an array-based implementation.

More precisely, the number of allocated nodes is at most proportional to the total number of distinct prefixes.

---

# 13. Why `isWord` Is Necessary

Consider:

```text
app
apple
```

The path:

```text
a → p → p
```

exists because of both words.

But if only:

```text
apple
```

was inserted:

```text
search("app")
```

must return:

```text
false
```

Therefore:

```java
isWord
```

distinguishes:

```text
prefix
```

from:

```text
complete word
```

---

# 14. Trie Example

Insert:

```text
app
apple
ape
bat
```

Conceptually:

```text
             root
            /    \
           a      b
           |      |
           p      a
          / \      |
         p   e     t
         |
         l
         |
         e
```

The `app` node has:

```text
isWord = true
```

and the `apple` node also has:

```text
isWord = true
```

---

# 15. Trie with HashMap

An array of 26 children is excellent for lowercase English letters.

But if the character set is large, use:

```java
Map<Character, TrieNode>
```

Example:

```java
class TrieNode {

    Map<Character, TrieNode> children =
        new HashMap<>();

    boolean isWord;
}
```

This saves space when each node has only a few children.

---

# 16. Array vs HashMap Trie

### Array

```java
TrieNode[] children =
    new TrieNode[26];
```

Advantages:

```text
Fast
Simple
Predictable
O(1) child access
```

Disadvantage:

```text
Can waste memory
```

### HashMap

```java
Map<Character, TrieNode>
```

Advantages:

```text
Flexible alphabet
Potentially less memory
```

Disadvantages:

```text
HashMap overhead
More object allocations
```

---

# 17. Trie for Uppercase / Lowercase

If input contains:

```text
A-Z
a-z
```

you can either:

```text
normalize to lowercase
```

or increase the alphabet size.

For production systems, define clearly whether the search is:

```text
case-sensitive
```

or:

```text
case-insensitive
```

---

# 18. Trie for Autocomplete

Suppose the dictionary contains:

```text
apple
application
apply
apt
banana
```

For prefix:

```text
app
```

find the Trie node corresponding to:

```text
app
```

Then perform DFS from that node to collect words.

---

# 19. Autocomplete — Concept

```text
prefix
  ↓
Trie node
  ↓
DFS
  ↓
collect matching words
```

This is one of the most common practical Trie applications.

---

# 20. Collect Words from Trie

```java
static void collectWords(
        TrieNode node,
        StringBuilder path,
        List<String> result) {

    if (node.isWord) {
        result.add(
            path.toString()
        );
    }

    for (int i = 0;
         i < 26;
         i++) {

        if (node.children[i] == null) {
            continue;
        }

        path.append(
            (char) ('a' + i)
        );

        collectWords(
            node.children[i],
            path,
            result
        );

        path.deleteCharAt(
            path.length() - 1
        );
    }
}
```

This combines:

```text
Trie
+
DFS
+
Backtracking
```

---

# 21. Trie Delete

Deleting a word requires care.

Example:

```text
app
apple
```

If we delete:

```text
app
```

we must not delete the shared:

```text
app
```

path because:

```text
apple
```

still needs it.

Usually:

```text
set isWord = false
```

and remove nodes only when they are no longer needed.

---

# 22. Trie Delete — Java

```java
public boolean delete(
        String word) {

    return delete(
        root,
        word,
        0
    );
}

private boolean delete(
        TrieNode node,
        String word,
        int index) {

    if (index == word.length()) {

        if (!node.isWord) {
            return false;
        }

        node.isWord = false;

        return hasNoChildren(node);
    }

    int childIndex =
        word.charAt(index) - 'a';

    TrieNode child =
        node.children[childIndex];

    if (child == null) {
        return false;
    }

    boolean shouldDeleteChild =
        delete(
            child,
            word,
            index + 1
        );

    if (shouldDeleteChild) {

        node.children[childIndex] =
            null;
    }

    return !node.isWord
        && hasNoChildren(node);
}

private boolean hasNoChildren(
        TrieNode node) {

    for (TrieNode child :
            node.children) {

        if (child != null) {
            return false;
        }
    }

    return true;
}
```

---

# 23. Trie for Prefix Counting

Sometimes we need:

```text
How many words start with this prefix?
```

Add a field:

```java
int prefixCount;
```

Every time a word passes through a node:

```java
prefixCount++;
```

Then prefix frequency can be returned in:

```text
O(L)
```

---

# 24. Trie with Word Frequency

Another useful node design:

```java
class TrieNode {

    TrieNode[] children =
        new TrieNode[26];

    boolean isWord;

    int wordCount;
}
```

This can support:

```text
duplicate word frequency
```

---

# 25. Longest Common Prefix

Given:

```text
flower
flow
flight
```

common prefix:

```text
fl
```

A Trie can solve this by inserting all words and walking from the root while:

```text
exactly one child exists
```

and:

```text
current node is not a complete word
```

---

# 26. Longest Common Prefix — Simpler Approach

For interviews, a Trie is not always necessary.

A simpler approach:

```text
Sort strings
```

Then compare:

```text
first string
last string
```

Their common prefix is the common prefix of the entire array.

Always choose the simplest correct solution.

---

# 27. Word Search II

Given a board and a dictionary of words, find all words present in the board.

A very efficient approach is:

```text
Trie
+
DFS
+
Backtracking
```

Build a Trie containing all dictionary words.

Then search the board.

---

# 28. Why Trie Helps Word Search II

Without a Trie:

```text
For every word:
    search board
```

This can repeat large amounts of work.

With a Trie:

```text
Board path
↓
Trie prefix
```

If the current board path is not a prefix of any word:

```text
stop immediately
```

This provides strong pruning.

---

# 29. Trie + DFS Pattern

```text
Insert all words into Trie
↓
Start DFS from every board cell
↓
Follow matching Trie child
↓
If no child:
    return
↓
If Trie node contains a word:
    record it
↓
Continue
↓
Restore board state
```

This is a classic interview combination.

---

# 30. Replace Words

Given dictionary roots:

```text
cat
bat
rat
```

and sentence:

```text
the cattle was rattled
```

replace words with their shortest dictionary root.

Result:

```text
the cat was rat
```

Trie is a natural solution.

---

# 31. Word Dictionary with Wildcards

Design a dictionary supporting:

```text
addWord("bad")
search("bad")
search(".ad")
search("b..")
```

For:

```text
.
```

we may need to explore all Trie children.

Therefore:

```text
Trie
+
DFS
```

is used.

---

# 32. Word Dictionary — Search Idea

For a normal character:

```text
follow one child
```

For:

```text
.
```

try:

```text
all non-null children
```

This creates branching only at wildcard positions.

---

# 33. Trie for XOR Problems

A **binary Trie** can store bits instead of characters.

For integers:

```text
32-bit representation
```

store:

```text
0
1
```

at each level.

This is useful for:

```text
Maximum XOR
Minimum XOR
XOR pair problems
```

---

# 34. Binary Trie

```java
class BinaryTrieNode {

    BinaryTrieNode[] children =
        new BinaryTrieNode[2];
}
```

Insert bits from:

```text
31
```

down to:

```text
0
```

---

# 35. Maximum XOR

For a number:

```text
x
```

at each bit, prefer the opposite bit:

```text
0 → prefer 1
1 → prefer 0
```

because:

```text
1 XOR 0 = 1
0 XOR 1 = 1
```

This greedily maximizes the XOR value.

---

# 36. Maximum XOR — Core Java

```java
static int findMaximumXOR(
        int[] nums) {

    BinaryTrieNode root =
        new BinaryTrieNode();

    for (int num : nums) {
        insert(root, num);
    }

    int answer = 0;

    for (int num : nums) {

        int current = 0;
        BinaryTrieNode node = root;

        for (int bit = 31;
             bit >= 0;
             bit--) {

            int currentBit =
                (num >>> bit) & 1;

            int preferred =
                1 - currentBit;

            if (node.children[preferred]
                    != null) {

                current |=
                    (1 << bit);

                node =
                    node.children[preferred];

            } else {

                node =
                    node.children[currentBit];
            }
        }

        answer =
            Math.max(
                answer,
                current
            );
    }

    return answer;
}

static void insert(
        BinaryTrieNode root,
        int num) {

    BinaryTrieNode node = root;

    for (int bit = 31;
         bit >= 0;
         bit--) {

        int value =
            (num >>> bit) & 1;

        if (node.children[value]
                == null) {

            node.children[value] =
                new BinaryTrieNode();
        }

        node =
            node.children[value];
    }
}
```

Complexity:

```text
Time: O(32n)
Space: O(32n)
```

which is effectively:

```text
O(n)
```

for fixed-width integers.

---

# 37. Trie vs HashSet

### HashSet

Excellent for:

```text
Exact word lookup
```

Expected:

```text
O(1)
```

lookup.

### Trie

Excellent for:

```text
Prefix lookup
Autocomplete
Lexicographic traversal
Prefix counting
```

The choice depends on the operation required.

---

# 38. Trie vs HashMap

A HashMap stores:

```text
key → value
```

A Trie stores:

```text
character paths
```

Use a Trie when the structure of the key matters, especially:

```text
prefixes
```

---

# 39. Trie vs Sorting

For repeated prefix queries:

```text
Trie
```

can be convenient.

For a one-time longest common prefix problem:

```text
sorting
```

may be simpler.

Interview rule:

> Do not use a Trie just because the problem contains strings. Use it when prefix-based operations make the Trie valuable.

---

# 40. Trie Complexity Summary

For word length `L`:

| Operation | Complexity |
|---|---:|
| Insert | O(L) |
| Search | O(L) |
| Starts With | O(L) |
| Delete | O(L) |
| Prefix Count | O(L) |
| Autocomplete | O(L + output) |
| Word Search II | Depends on board and dictionary |

---

# 41. Trie Memory Considerations

An array-based Trie:

```java
new TrieNode[26]
```

allocates a reference array for every node.

For very large dictionaries, this can use substantial memory.

Possible optimizations:

```text
HashMap children
Compressed Trie
Radix Tree
Ternary Search Tree
```

---

# 42. Compressed Trie / Radix Tree

Instead of storing one character per node, compress chains with only one child.

Example:

```text
a → p → p → l → e
```

can become:

```text
"apple"
```

as a compressed edge.

This is useful for memory optimization and is related to practical routing/search structures.

---

# 43. Autocomplete Ranking

A real autocomplete system may need:

```text
prefix
+
frequency
+
ranking
+
recency
```

A Trie can store metadata such as:

```java
int frequency;
```

or:

```text
top K suggestions
```

at nodes.

This moves the problem closer to real-world backend/system design.

---

# 44. Trie in Backend Systems

Tries can be useful for:

```text
Search suggestions
URL routing
Command completion
Dictionary services
Prefix filtering
IP routing
Content filtering
```

For very large-scale systems, specialized structures may be preferred depending on latency and memory requirements.

---

# 45. Common Trie Mistakes

### Mistake 1 — Forgetting `isWord`

A prefix is not necessarily a complete word.

---

### Mistake 2 — Wrong character index

For lowercase English:

```java
ch - 'a'
```

---

### Mistake 3 — Deleting shared prefixes

Never remove nodes needed by another word.

---

### Mistake 4 — Using Trie when HashSet is enough

Exact lookup does not require a Trie.

---

### Mistake 5 — Not sorting/handling duplicates when required

Trie traversal may produce duplicate results if the input model allows duplicate words.

---

### Mistake 6 — Forgetting to restore board state

In Trie + DFS problems:

```java
board[row][col] = original;
```

is often essential.

---

# 46. Edge Cases

Always test:

```text
Empty string
Empty dictionary
One word
Duplicate words
Word is prefix of another word
Prefix does not exist
Search before insertion
Delete nonexistent word
Delete a word that is a prefix of another
Case sensitivity
Large dictionary
Long words
```

---

# 47. Interview Questions — Easy

1. Implement Trie.
2. Insert and search words.
3. Prefix search.
4. Longest Common Prefix.
5. Word dictionary basics.
6. Prefix counting.

---

# 48. Interview Questions — Medium

7. Design Add and Search Words.
8. Replace Words.
9. Word Search II.
10. Autocomplete.
11. Maximum XOR.
12. Map Sum Pairs.
13. Search Suggestions System.
14. Longest Word in Dictionary.

---

# 49. Interview Questions — Advanced

15. Binary Trie.
16. Maximum XOR with constraints.
17. Trie + backtracking.
18. Trie + DFS optimization.
19. Compressed Trie.
20. Radix Tree.
21. Top-K autocomplete.
22. Trie-based routing.
23. Advanced prefix matching.

---

# 50. Quick Revision

```text
Trie
│
├── Basic
│   ├── Insert
│   ├── Search
│   ├── Starts With
│   └── Delete
│
├── Prefix Problems
│   ├── Autocomplete
│   ├── Prefix Count
│   ├── Replace Words
│   └── Longest Common Prefix
│
├── Trie + DFS
│   ├── Word Search II
│   └── Wildcard Dictionary
│
├── Binary Trie
│   ├── Maximum XOR
│   └── Minimum XOR
│
└── Advanced
    ├── Compressed Trie
    ├── Radix Tree
    └── Search Ranking
```

---

## Interview Rule

> **Use a Trie when the problem is about prefixes. Remember the three core operations: insert, exact search, and prefix search — all O(L), where L is the key length. For advanced problems, combine Trie with DFS/backtracking or use a binary Trie for XOR problems.**
