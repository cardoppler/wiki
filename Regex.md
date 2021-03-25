# Regex
A string that defines a pattern which can be used to validate, parse, splits, replace a target/subject string.

Concepts: Concatenation, Alternation, Repetition

- `\w` any word character. Same as `[a-zA-Z0-9_]`. `\W` anything not a word
- `\s` whitespace, tab, newline. `\S` not whitespace
- `\d` matches numbers. `\D` not-numbers
- `[abc]` character class. `[^abc]` not `a`, `b`, or `c`. `gr[ae]y` matches `grey` and `gray`. Looks for an `a` OR an `e`.
- `(cat|dog)`. The order matters. The regex considers the leftmost sequence in the target/subject string: the substring `dog` in `dogcat`:
```
PS C:\Users\cardo> 'dogcat' -match '(cat|dog)'
True
PS C:\Users\cardo> $Matches
Name                           Value
----                           -----
1                              dog
0                              dog

PS C:\Users\cardo> 'catzzzdog' | Select-String -Pattern '(dog)?|cat' -all | % Matches | % Value
dog

PS C:\Users\cardo> 'catzzzdog' | Select-String -Pattern '(dog)?|cat' -all | % matches | % value
dog

PS C:\Users\cardo> 'catzzzdog' | Select-String -Pattern '(cat)?|dog' -all | % matches | % value
cat

PS C:\Users\cardo> 'catzzzdog' | Select-String -Pattern 'cat|(dog)' -all | % matches | % value
cat
dog
```

## Anchors
They match positions, not characters.
Anchor the regex at a certain position.
Multi-line mode (i.e. match after a line break `\n`) might need to be activated.
- `^` start of string, before the first character in the string
- `$` end of string, after the last character.
- `\b` word boundary. Matches the position before/after the first/last word character. `\bWordBoundary?\b` matches the position before th `W` and it also matches the `?` because it is after the `y`
- `\A` 

```
> 'cat scatty cat scatter' | Select-String -Pattern '\bcat\b' -all | % matches | % value
cat
cat

> 'admin', 'admins', 'adminsss' -match 'admins?$'
admin
admins
```

## Quantifiers
`* + ?` are greedy quantifiers: they try to match as much as they can.

- `?` 0 or 1 (i.e. makes the preceding token optional). `colou?r`
- `*` 0 or more (aka Kleene star)
- `+` 1 or more
- `{n}` exactly `n` times; `{n,}` at least `n` times;  `{min,max}` at least `min` but not more than `max` times.

## Other
`.` single character except line break. Don't use it.
`\` escape, makes special chars literal
`\t` tab
`\r` return
`\n` new line

## Modifiers
- `(?i)abc`case insensitive
- `(?m)` multi-line; `(?s)` single-line
- `(?n)` do not capture unnamed groups
```
> 'abcd 1234' -match '(?<word>\w+)\s+(\d+)'
True
> $Matches
Name                           Value
----                           -----
word                           abcd
1                              1234
0                              abcd 1234


> 'abcd 1234' -match '(?n)(?<word>\w+)\s+(\d+)'
True
> $Matches
Name                           Value
----                           -----
word                           abcd
0                              abcd 1234
```

## Subexpressions, Groups, Capturing groups, Named groups, Backreferences, Lookaround
- `()?`. `Set(Value)?` matches `Set` and `SetValue` and the values matches are saved in variable $1, $2, ...
- `(?:)` is a non-capturing subexpression: the values are not going to be stored in a variables
- `\1` is equal to the text matched by the previous capturing group. `([abc])=\1` matches `a=a`, `b=b`, `c=c`
- `(?<group_name>[abc])=\k<group_name>`

### Examples
Group $0 is the whole match, $1 is the first subexpression from the left.
```
> '202-555-0148' -match '(\d{3}-)+(\d{4})'
True

> $Matches
Name                           Value
----                           -----
2                              0148
1                              555-
0                              202-555-0148

> '202-555-0148' -match '(?:\d{3}-)+(\d{4})'
True

> $Matches
Name                           Value
----                           -----
1                              0148
0                              202-555-0148

> '202-555-0148' -replace '(?:\d{3}-)+(\d{4})','$1'
0148
```

<details><summary>With `Select-String -Pattern`</summary>

```
> '202-555-0148' | Select-String -Pattern '(\d+)+' -All | % matches
Groups   : {0, 1}
Success  : True
Name     : 0
Captures : {0}
Index    : 0
Length   : 3
Value    : 202

Groups   : {0, 1}
Success  : True
Name     : 0
Captures : {0}
Index    : 4
Length   : 3
Value    : 555

Groups   : {0, 1}
Success  : True
Name     : 0
Captures : {0}
Index    : 8
Length   : 4
Value    : 0148
```
</details>

### Named Captures
- Modifier `(?n)` does not capture unnamed groups
```
> 'abcd 1234' -match '(?<word>\w+)\s+(\d+)'
True
> $Matches
Name                           Value
----                           -----
word                           abcd
1                              1234
0                              abcd 1234


> 'abcd 1234' -match '(?n)(?<word>\w+)\s+(\d+)'
True
> $Matches
Name                           Value
----                           -----
word                           abcd
0                              abcd 1234
```
- `(?x)` ignore spaces and comments in a regex, so it becomes more readable
```
> '202-555-0148' -match '(?x) (\d{3} - ) + (\d{4}) # a comment here'
True
```


<details> <summary>Using methods `Object.Match()`and `Object.Matches()`: (??? Captures did **not** work)</summary>

```
> $line = 'abcd 1234 efg 567'
> [regex]$regex = '(?<word>[a-z]+) (?<num>[0-9]+)'
> $regex.Match($line)

Groups   : {0, word, num}
Success  : True
Name     : 0
Captures : {0}
Index    : 0
Length   : 9
Value    : abcd 1234


> $string = 'abcd 1234 efg 567'
> $regex.Matches($string)

Groups   : {0, word, num}
Success  : True
Name     : 0
Captures : {0}
Index    : 0
Length   : 9
Value    : abcd 1234

Groups   : {0, word, num}
Success  : True
Name     : 0
Captures : {0}
Index    : 10
Length   : 7
Value    : efg 567
```
</details>

Without assigning to variables:
```
> [regex]::Matches('abcd 1234 efg 567','(?<word>[a-z]+) (?<num>[0-9]+)').value
abcd 1234
efg 567
```

## Lookaround
Only assert whether a match is possible or not at a given position. Matches each position (like anchors) and gives up the match - zero length assertions, they do not consume characters. 
- `(?=)` positive lookahead. Successfull if the regex can match to the right.

Look for a position where at its right you have 3 digits at the end of the string; if you find it insert a comma in that position:
```
> '1000' -replace '(?=\d{3}$)',','
1,000
```
- `(?!)` negative lookahead. Succesfull it the regex cannot match to the right
- `(?<=)` positive lookbehind
- `(?<!)` negative lookbehind

Lookahead and lookbehind:
```
> $LocalAdmins = net localgroup administrators | Out-String
> $LocalAdmins
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
cardo
The command completed successfully.


> [regex]::Matches($LocalAdmins,'(?s)(?<=\-\r+\n).*(?=The)').value.trim()
Administrator
cardo
```
## Examples
### PowerShell
the keywords are: `-match`, `-replace`
```
> 'CN=username,CN=Users,DC=example,DC=com' -match 'example,dc=com'
True
> $Matches
Name                           Value
----                           -----
0                              example,DC=com
```

Show the matches
```
> 'gray and grey' | Select-String -Pattern 'gr[ae]y' -all | % Matches
Groups   : {0}
Success  : True
Name     : 0
Captures : {0}
Index    : 0
Length   : 4
Value    : gray

Groups   : {0}
Success  : True
Name     : 0
Captures : {0}
Index    : 9
Length   : 4
Value    : grey
```

Replace `[` and `.` with nothing:
```
> 'Us[er Na.me' -replace '[[\.]',''
User Name
```
Initialise a variable with a list of string and then match with a character range:
```
> $IPAddresses = '192.168.20.1','192.168.20.72','192.168.20.3'
> $IPAddresses -match '192.168.20.[1-6]'
192.168.20.1
192.168.20.3
```

PS is case insensitive by default. use `-cmatch` to specify sensitivity. Here only match lower case `input`:
```
> 'INPUT','input' -cmatch '[a-z]'
input
```

**Split** an email address using any non-alphabetical character as delimiter; in this case the `@` and the `.` :
```
> 'cardoppler@gmail.com' -split '[^a-z]'
cardoppler
gmail
com
```

**where** and **alternation**.
```
> Get-EventLog -LogName System -EntryType Error | Where Message -Match 'CldFlt|Printer'

   Index Time          EntryType   Source                 InstanceID Message
   ----- ----          ---------   ------                 ---------- -------
     242 Apr 12 11:05  Error       Service Control M...   3221232472 The CldFlt service failed to start due to the following error: ...
     156 Apr 12 11:00  Error       Service Control M...   3221232502 The Printer Extensions and Notifications service is marked as an interactive service.  However, the syste...
      40 Apr 12 10:58  Error       Service Control M...   3221232472 The CldFlt service failed to start due to the following error: ...
```      


## .NET syntax

## Capture groups and output to csv (macos)
```
pcregrep -o1 -o2 -o3 -o4 --om-separator=',' '\S+ \(\S+\) - - \[(?<DATETIME>\S+)\] "(?<HTTP_METHOD>\w+) (?<HTTP_PATH>\S+) \S+ (?<HTTP_RESPONSE_CODE>\d{3})' Notes.md > output.csv
```
