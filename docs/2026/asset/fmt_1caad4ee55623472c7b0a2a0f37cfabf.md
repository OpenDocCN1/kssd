

# Regular Expression Syntax Reference

## Escape Sequences

`\b` `\b` (but only inside a character class declaration).

`\cX` - An ASCII escape sequence - the character whose code point is `X % 32`

`\xdd` - A hexadecimal escape sequence - matches the single character whose code point is `0xdd`.

`\x{dddd}` - A hexadecimal escape sequence - matches the single character whose code point is `0xdddd`.

`\0ddd` - An octal escape sequence - matches the single character whose code point is `0ddd`.

`\N{name}` - Matches the single character which has the symbolic name `name`. For example `\N{newline}` matches the single character `\n`.

## Character Classes

"Single character" character classes:

Any escaped character `x`, if `x` is the name of a character class shall match any character that is a member of that class, and any escaped character `X`, if `x` is the name of a character class, shall match any character not in that class.

The following are supported by default:

| Escape sequence | Equivalent to |
| --- | --- |
| `\d` | `[[:digit:]]` |
| `\l` | `[[:lower:]]` |
| `\s` | `[[:space:]]` |
| `\u` | `[[:upper:]]` |
| `\w` | `[[:word:]]` |
| `\D` | `[^[:digit:]]` |
| `\L` | `[^[:lower:]]` |
| `\S` | `[^[:space:]]` |
| `\U` | `[^[:upper:]]` |
| `\W` | `[^[:word:]]` |

## Character Properties

The character property names in the following table are all equivalent to the names used in character classes.

| Form | Description | Equivalent character set form |
| --- | --- | --- |
| `\pX` | Matches any character that has the property `X`. | `[[:X:]]` |
| `\p{Name}` | Matches any character that has the property `Name`. | `[[:Name:]]` |
| `\PX` | Matches any character that does not have the property `X`. | `[^[:X:]]` |
| `\P{Name}` | Matches any character that does not have the property `Name`. | `[^[:Name:]]` |

## Word Boundaries

The following escape sequences match the boundaries of words:

- `\<` - Matches the start of a word.
- `\>` - Matches the end of a word.
- `\b` - Matches a word boundary (the start or end of a word).
- `\B` - Matches only when not at a word boundary.

## Buffer Boundaries

The following match only at buffer boundaries: a "buffer" in this context is the whole of the input text that is being matched against (note that `^` and `$` may match embedded newlines within the text).

- `` \` `` - Matches at the start of a buffer only.
- `\'` - Matches at the end of a buffer only.
- `\A` - Matches at the start of a buffer only (the same as `` \` ``).
- `\z` - Matches at the end of a buffer only (the same as `\'`).
- `\Z` - Matches an optional sequence of newlines at the end of a buffer: equivalent to the regular expression `\n\*\z`.

## Continuation Escape

The sequence `\G` matches only at the end of the last match found, or at the start of the text being matched if no previous match was found. This escape useful if you're iterating over the matches contained within a text, and you want each subsequence match to start where the last one ended.

## Quoting Escape

The escape sequence `\Q` begins a "quoted sequence": all the subsequent characters are treated as literals, until either the end of the regular expression or `\E` is found. For example the expression: `\Q\\*+\Ea+` would match either of:

```
\\*+a\\*+aaa
```

## Unicode Escapes

- `\C` - Matches a single code point: in Boost regex this has exactly the same effect as a `.` operator.
- `\X` - Matches a combining character sequence: that is any non-combining character followed by a sequence of zero or more combining characters.

## Any Other Escape

Any other escape sequence matches the character that is escaped, for example `\@` matches a literal `'@'`.

## Perl Extended Patterns

Perl-specific extensions to the regular expression syntax all start with `(?`.

### Comments

`(?# ... )` is treated as a comment, it's contents are ignored.

### Modifiers

`(?imsx-imsx ... )` alters which of the perl modifiers are in effect within the pattern; changes take effect from the point that the block is first seen and extend to any enclosing `)`. Letters before a `-` turn that perl modifier on, letters afterward turn it off.

`(?imsx-imsx:pattern)` applies the specified modifiers to `pattern` only.

### Non-marking Grouping

`(?:pattern)` lexically groups `pattern`, without generating an additional sub-expression.

### Lookahead

`(?=pattern)` consumes zero characters, only if `pattern` matches.

`(?!pattern)` consumes zero characters, only if `pattern` does not match.

Lookahead is typically used to create the logical AND of two regular expressions, for example if a password must contain a lower case letter, an upper case letter, a punctuation symbol, and be at least 6 characters long, then the expression:

```
(?=.\*[[:lower:]])(?=.\*[[:upper:]])(?=.\*[[:punct:]]).{6,}
```

could be used to validate the password.

### Lookbehind

`(?<=pattern)` consumes zero characters, only if `pattern` could be matched against the characters preceding the current position (pattern must be of fixed length).

`(?<!pattern)` consumes zero characters, only if `pattern` could not be matched against the characters preceding the current position (pattern must be of fixed length).

### Independent Sub-expressions

`(?>pattern)` - `pattern` is matched independently of the surrounding patterns; the expression will never backtrack into `pattern`.

Independent sub-expressions are typically used to improve performance; only the best possible match for `pattern` will be considered, if this doesn't allow the expression as a whole to match then no match is found at all.

### Conditional Expressions

`(?(condition)yes-pattern|no-pattern)` - attempts to match `yes-pattern` if the `condition` is true, otherwise attempts to match `no-pattern`.

`(?(condition)yes-pattern)` - attempts to match `yes-pattern` if the `condition` is true, otherwise fails.

`condition` may be either a forward lookahead assert, or the index of a marked sub-expression (the condition becomes true if the sub-expression has been matched).

## Operator Precedence

The order of precedence for of operators is as shown in the following list:

- `[==]`, `[::]`, `[..]` - Collation-related bracket symbols
- `\` - Escaped characters
- `[]` - Character set (bracket expression)
- `()` - Grouping
- `\*`, `+`, `?`, `{m,n}` - Single-character-ERE duplication
- *(no symbol)* - Concatenation
- `^`, `$` - Anchoring
- `|` - Alternation

## What Gets Matched

If you view the regular expression as a directed (possibly cyclic) graph, then the best match found is the first match found by a depth-first-search performed on that graph, while matching the input text.

Alternatively: the best match found is the leftmost match, with individual elements matched as follows;

**Construct What gets matches**

- `AtomA AtomB` - Locates the best match for `AtomA` that has a following match for `AtomB`.
- `Expression1 | Expression2` - If `Expresion1` can be matched then returns that match, otherwise attempts to match `Expression2`.
- `S{N}` - Matches `S` repeated exactly `N` times.
- `S{N,M}` - Matches `S` repeated between `N` and `M` times, and as many times as possible.
- `S{N,M}?` - Matches `S` repeated between `N` and `M` times, and as few times as possible.
- `S?`, `S\*`, `S+` - The same as `S{0,1}`, `S{0,UINT\_MAX}`, `S{1,UINT\_MAX}` respectively.
- `S??`, `S\*?`, `S+?` - The same as `S{0,1}?`, `S{0,UINT\_MAX}?`, `S{1,UINT\_MAX}?` respectively.
- `(?>S)` - Matches the best match for `S`, and only that.
- `(?=S)`, `(?<=S)` - Matches only the best match for `S` (this is only visible if there are capturing parentheses within `S`).
- `(?!S)`, `(?<!S)` - Considers only whether a match for `S` exists or not.
- `(?(condition)yes-pattern | no-pattern)` - If `condition` is true, then only `yes-pattern` is considered, otherwise only `no-pattern` is considered.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

