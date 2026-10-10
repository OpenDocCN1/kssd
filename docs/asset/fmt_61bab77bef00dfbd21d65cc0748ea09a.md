
2006 年 09 月 22 日 17:00:00

附：正则表达式语法说明（PERL，javascript 中正则表达式的语法规则）

In regular expressions, all characters match themselves except for the following special characters:

`.\[{()\\\*+?|^$`

### Wildcard

The single character `'.'` when used outside of a character set will match any single character except:

### Anchors

A `'^'` character shall match the start of a line.

A `'$'` character shall match the end of a line.

### Marked sub-expressions

A section beginning `(` and ending `)` acts as a marked sub-expression. Whatever matched the sub-expression
is split out in a separate field by the matching algorithms. Marked sub-expressions can also repeated,
or referred to by a back-reference.

### Non-marking grouping

A marked sub-expression is useful to lexically group part of a regular expression, but has the side-effect
of spitting out an extra field in the result. As an alternative you can lexically group part of a regular
expression, without generating a marked sub-expression by using `(?:` and `)`, for example `(?:ab)+` will repeat
`"ab"` without splitting out any separate sub-expressions.

### Repeats

Any atom (a single character, a marked sub-expression, or a character class) can be repeated with the `\*`, `+`, `?`, and `{}` operators.

The `\*` operator will match the preceding atom zero or more times, for example the expression `a\*b` will match any of the following:

```
b
ab
aaaaaaaab
```

The `+` operator will match the preceding atom one or more times, for example the expression `a+b` will match any of the following:

```
ab
aaaaaaaab
```

But will not match:

```
b
```

The `?` operator will match the preceding atom zero or one times, for example the expression `ca?b` will match any of the following:

```
cb
cab
```

But will not match:

```
caab
```

An atom can also be repeated with a bounded repeat:

`a{n}` Matches `'a'` repeated exactly n times.

`a{n,}` Matches `'a'` repeated n or more times.

`a{n, m}` Matches `'a'` repeated between n and m times inclusive.

For example:

```
^a{2,3}$
```

Will match either of:

```
aa
aaa
```

But neither of:

```
a
aaaa
```

It is an error to use a repeat operator, if the preceding construct can not be repeated, for example:

```
a(\*)
```

Will raise an error, as there is nothing for the `\*` operator to be applied to.

### Non greedy repeats

The normal repeat operators are "greedy", that is to say they will consume as much input as possible. There are non-greedy versions
available that will consume as little input as possible while still producing a match.

`\*?` Matches the previous atom zero or more times, while consuming as little input as possible.

`+?` Matches the previous atom one or more times, while consuming as little input as possible.

`??` Matches the previous atom zero or one times, while consuming as little input as possible.

`{n,}?` Matches the previous atom n or more times, while consuming as little input as possible.

`{n,m}?` Matches the previous atom between n and m times, while consuming as little input as possible.

### Back references

An escape character followed by a digit n, where n is in the range 1-9, matches the same string that was matched by sub-expression n.

For example the expression:

```
^(a\*).*\\1$
```

Will match the string:

```
aaabbaaa
```

But not the string:

```
aaabba
```

### Alternation

The `|` operator will match either of its arguments, so for example: `abc|def` will match either `"abc"` or `"def"`.

Parenthesis can be used to group alternations, for example: `ab(d|ef)` will match either of `"abd"` or `"abef"`.

Empty alternatives are not allowed (these are almost always a mistake), but if you really want an empty alternative use `(?:)` as a
placeholder, for example:

`"|abc"` is not a valid expression, but

`"(?:)|abc"` is and is equivalent, also the expression:

`"(?:abc)??"` has exactly the same effect.

### Character sets

A character set is a bracket-expression starting with `\[` and ending with `\]`, it defines a set of characters, and matches any single
character that is a member of that set.

A bracket expression may contain any combination of the following:

#### Single characters

For example `\[abc\]`, will match any of the characters `'a'`, `'b'`, or `'c'`.

#### Character ranges

For example `\[a-c\]` will match any single character in the range `'a'` to `'c'`.

#### Negation

If the bracket-expression begins with the `^` character, then it matches the complement of the characters it contains, for example `[^a-c]`
matches any character that is not in the range a-c.

#### Character classes

An expression of the form `\[\[:name:\]\]` matches the named character class "name", for example `\[\[:lower:\]\]` matches any lower case character.

See character class names.

#### Collating Elements

An expression of the form `\[\[.col.\]` matches the collating element col. A collating element is any single character, or any sequence of
characters that collates as a single unit. Collating elements may also be used as the end point of a range, for example: `\[\[.ae.\]-c\]`
matches the character sequence `"ae"`, plus any single character in the range `"ae"`-c, assuming that `"ae"` is treated as a single collating
element in the current locale.

As an extension, a collating element may also be specified via it's symbolic name, for example:

`\[\[.NUL.\]\]`

matches a NUL character.

#### Equivalence classes

An expression oftheform `\[\[=col=\]\]`, matches any character or collating element whose primary sort key is the same as that for collating
element col, as with colating elements the name col may be a symbolic name. A primary sort key is one that ignores case, accentation,
or locale-specific tailorings; so for example `\[\[=a=\]\]` matches any of the characters: a, à, á, a, ?, ?, ?, A, à, á, ?, ?, ? and ?.

Unfortunately implementation of this is reliant on the platform's collation and localisation support; this feature can not be relied
upon to work portably across all platforms, or even all locales on one platform.

#### Escapes

All the escape sequences that match a single character, or a single character class are permitted within a character class definition,
except the negated character classes (`\\D \\W` etc).

#### Combinations

All of the above can be combined in one character set declaration, for example: `\[\[:digit:\]a-c\[.NUL.\]\]`.

### Escapes

Any special character preceded by an escape shall match itself.

The following escape sequences are also supported:

#### Escapes matching a specific character

The following escape sequences are all synonyms for single characters:

Escape Character

`\\a` `'\\a'`

`\\e` `0x1B`

`\\f` `\\f`

`\\n` `\\n`

`\\r` `\\r`

`\\t` `\\t`

`\\v` `\\v`

`\\b` `\\b` (but only inside a character class declaration).

`\\cX` An ASCII escape sequence - the character whose code point is X % 32

`\\xdd` A hexadecimal escape sequence - matches the single character whose code point is 0xdd.

`\\x{dddd}` A hexadecimal escape sequence - matches the single character whose code point is 0xdddd.

`\\0ddd` An octal escape sequence - matches the single character whose code point is 0ddd.

`\\N{name}` Matches the single character which has the symbolic name name. For example `\\N{newline}` matches the single character `\\n`.

#### "Single character" character classes

Any escaped character x, if x is the name of a character class shall match any character that is a member of that class, and any
escaped character X, if x is the name of a character class, shall match any character not in that class.

The following are supported by default:

Escape sequence Equivalent to

`\\d` `\[\[:digit:\]\]`

`\\l` `\[\[:lower:\]\]`

`\\s` `\[\[:space:\]\]`

`\\u` `\[\[:upper:\]\]`

`\\w` `\[\[:word:\]\]`

`\\D` `\[^\[:digit:\]\]`

`\\L` `\[^\[:lower:\]\]`

`\\S` `\[^\[:space:\]\]`
