<!-- pandoc -f commonmark -t html understanding-j.md -->
<!--
<head><meta http-equiv="refresh" content="5"/></head>
<style>
pre {
    background: #dddddd;
    padding: 1em;
    overflow: auto;
    border-radius: 0.3em;
}
code {
    background: #dddddd;
    border-radius: 0.3em;
}
blockquote {
    border-left: #dddddd 0.5em solid;
    padding: 0.5em;
}
</style>
-->

- J versions: 9.4.2,

Found anything wrong? File an issue at
<https://github.com/bugsbugsbux/understanding-j/issues>.

---

# Understanding J

> An introduction to the J programming language that gets to the point.

It is intended for those with (some) programming experience, but others
should be mostly fine after looking up some basic programming terms like
function, argument, class, instance, inheritance, statement, expression,
etc.

Don't treat this as a reference: Section titles do not introduce an
exhaustive explanation of a certain topic, but serve to give this
document a rough structure. Individual sections are not intended to be
read in isolation from the others and assume the knowledge of previous
sections.

**Run the examples and read the comments!**

If you have J installed you can open this file in JQt via the
file-selection dialog which opens with `ctrl+o` (make sure to set the
filetype-filter to "all"). Click on a line with J code and press
`ctrl+enter` to execute it.

Covered builtins are listed in the appendix with a short description.

Important links:

- Project Homepage: <https://jsoftware.com>
- Online J interpreter:
  <https://jsoftware.github.io/j-playground/bin/html2/>
- **All builtin operators**, with links to their wiki pages:
  <https://code.jsoftware.com/wiki/NuVoc>
- Good old wiki:
  <https://www.jsoftware.com/help/dictionary/contents.htm>
- Cheatsheet/Reference Card (not for beginners):
  <https://code.jsoftware.com/wiki/File:B.A4.pdf>

---

## History:

J was first released in 1990 as a successor to APL, an alternative
mathematical notation that is computer-executable and works well with
multi-dimensional array data. Most notably J switches from APL's custom
symbol-set to ASCII only, calling most basic builtins by a single symbol
or a symbol with appended dot or colon, and giving distinct meaning to
single symbols that usually appear in pairs like various braces and
quotes (`[]"{}` etc).

## Comments:

```J
NB. comments the rest of the line (latin: nota bene).
```

## Numbers:

J has many number-notations; the most important are:
```J
3                   NB. integer
3.14                NB. float
_3                  NB. negative numbers start with underscore
_                   NB. sole underscore is infinity: a number
__                  NB. negative infinity
12j34               NB. complex number (sometimes used as argument pair)
16bcoffee           NB. base-16 number (must be lowercase)
```

## Lists, Booleans:

The notation is simple: just put elements next to each other. All
elements must to have the same type but as an empty list has no elements
it can be combined with any other list. True and false are represented
simply by the two numbers `1` and `0`. See also: strings

```J
0 1                 NB. the booleans (truth values) in a 2-element list
0 1 2 3 4           NB. due to context a list of numbers not booleans
(0 1 2) (3 4)       NB. error: can't implicitly join lists, see below
```

## Strings:

Strings are lists thus an empty string is commonly used as empty list
notation. The default string type is called "literal" and is UTF-8
encoded which means the first 128 ASCII characters work well with arrays
as they are a single byte long. String literals are singlequoted, and
only this character is special; it is escaped by doubling it. See also:
string representation and unicode

```J
''                  NB. empty string is empty list
'this is a string. it''s always singlequoted'
```

## Nouns:

*Nouns* are data values such as the ones covered until now. *Boxes*,
which will be covered later, are nouns too. Lists are actually just a
basic case of *arrays* which can be thought of as nested lists.

Classes and their instances aren't nouns: they cannot be referenced
directly; but their name can be saved as a string.

Functions aren't nouns either; but by putting them into a special kind
of list, *gerunds*, they can effectively be made one (also covered
below).

### Special data types:

J comes with some specialized data types, which wont be covered here,
but shall at least be mentioned:

- **Symbols**: They are a specialized type of string, which is
  immutable, treated as a single element and has optimized performance
  with searching, sorting and comparison operations. However, only a
  few, often used ones should be created, as they are registered in a
  global table and cannot be removed from it anymore. More info at:
  <https://code.jsoftware.com/wiki/Vocabulary/sco>
  <https://www.jsoftware.com/help/dictionary/dsco.htm>
- **Sparse arrays**: If a sizable array (for example a list; see:
  arrays) consists of mostly the same repeated value, its size can be
  reduced by only saving the values which are not this so called
  sparse-element. Contrary to regular arrays, sparse arrays are
  displayed as a table of index-value pairs of the non-sparse-elements
  and need to be queried explicitly for their sparse-element.
  Computations with them are envisioned as equivalent to regular arrays
  but this is not fully implemented (for example, sparse arrays cannot
  be boxed (see: boxes) and strings (which are arrays) cannot be
  sparse); the results are usually sparse arrays too, however, the
  sparse-element might differ! More info at:
  <https://www.jsoftware.com/help/dictionary/d211.htm>

## Functions:

**Verbs** are functions that take nouns as their argument/s and return
nouns. A verb has access to its left argument via the local variable `x`
and to its right argument via the local variable `y`. If there are
arguments on both sides of a function it is called dyadic (such verbs
are *dyads*) otherwise the function is monadic (and such verbs are
called *monads*; not related to the haskell term).
```J
  -                 NB. fn (here minus) without arg does not execute
  - 1               NB. monad (1 arg, but may be list)
1 -                 NB. error: monads take their arg from the right only
1 - 2               NB. dyad (2 args, may be lists)

    - 1 2           NB. monad with list argument
0 1 2 , 3 4         NB. the dyad , joins lists
```
Note that the monadic and dyadic case here are two distinct functions:
negate-number and subtract. They share a name (the symbol `-`) and a
definition: the function `-` is *ambivalent*. A function does not have
to have both cases.

**Modifiers** are functions used to pre-process a statement/expression;
this means they run before the main evaluation step. Modifiers return a
new function that has, additionally to its own arguments `x` and `y`,
access to the original arguments as variables `u` (left, `m` may be used
instead to indicate a noun) and `v` (right, use `n` instead to indicate
a noun). The new function may return any entity, even more modifiers,
which would also be processed before the first verb evaluates!

```J
1 + 2 + 3           NB. dyadic + is addition
+  1 2 3j4          NB. monadic + gives the complex conjugates: 1 2 3j_4
+/ 1 2 3            NB. 6 because: 1 + 2 + 3 is 6
```

The last example showed that `+/y` is not the same as `+y` but rather
`y1 + y2 + y3 ...`. This is because `/` is an *adverb* (a modifier that
takes one argument to the *left*): It creates a new function that
inserts the original argument `+` between all elements of its own
argument `1 2 3`.

*Conjunctions* only differ insofar as they (meaning the original
modifiers not the returned entities) take two arguments. For example `&`
takes a dyad and a noun and creates a new verb which always uses the
given noun as one of its arguments and the argument to the newly created
monad as the other argument:
```J
2 ^ 3               NB. dyad ^ is power: "two to the three"
2&^ 0 1 2 3         NB. convert dyad ^ with left arg 2 to a monad
^&(2) 0 1 2 3       NB. convert dyad ^ with right arg 2 to a monad
```

## Assignments:

Assignments use the operators `=:` (global) and `=.` (function-local).
They return their values, but the interpreter does not display them, so
often monads `[` or `]`, which return their argument unchanged, are used
as prefix. When assigning to multiple variables the return value is
still the original argument.
```J
foo =: 1            NB. return value of assignment not shown
[ foo =: foo + 1    NB. global assignment
] foo =. foo + 1    NB. local assignment if in function otherwise global

NB. btw the dyadic versions return either the left or the right arg:
0 [ 1
0 ] 1
```

It is possible to assign multiple values at the same time, but the value
returned is the unchanged original argument.
```J
'foo bar' =: 'val'  NB. names as space joined strings
foo
bar
baz=:'foo bar'=:3 4 NB. multi assignment returns unchanged arg
baz                 NB. thus baz received the original arg
foo                 NB. but foo got first
bar                 NB. and bar got second value
'foo bar' =: 1 2 3  NB. error: same number of names as elements required
```

See also: importing code

## Importing Code:

To run the code from some file use one of these convenient verbs:

- `load`: Runs the specified file or shortname (list available
  shortnames with `scripts''`).
- `loadd`: Like `load` but displays the lines before executing them.
- `require`: Like `load` but files that were already loaded won't be
  loaded again.

As these verbs, like all functions, have their own scope, assignments in
the loaded file/s which use `=.` but target the current global scope get
captured by the loading function's scope and thus are not available
afterwards!

## Defining Entities (Functions, Nouns) by Explicit-Definition:

Explicit-definitions are created with the `:` conjunction which returns
an entity of the type indicated by the integer to its *left*:

- `0` for nouns,
- `1` for adverbs,
- `2` for conjunctions,
- `3` for monadic (optionally ambivalent) verbs,
- `4` for dyads.

The entities' value is specified as the *right* argument and is a string
for functions. A value of `0` always means to read in the following
lines as a string - until the next line containing `)` as its only
printable character.

```J
NB. nouns
0 : 'string'            NB. creates noun with value 'string'
1 + 0 : 100             NB. creates and uses noun 100 (the number)
echo '>', (0 : 0), '<'  NB. creates noun from next lines and uses it
A Multiline string.
Make sure to put a space between the left arg and : because otherwise it
is parsed as one of the functions __: or _9: to 0: or 1: to 9: or _:
that ignore their args and always return their (negative) digit/infinity
)

NB. verbs
1 (4 : 'x + y') 2
(3 : '-y') 1
(3 :'3 :''1+y'' y') 1   NB. nested
fn =: 3 : 0             NB. assign new (ambivalent) multiline verb
    echo 'First the body of a monad, then optionally the body of a dyad'
    echo 'separated by a line containing : as its only printable symbol'
    :
    echo 'Multiline explicit-defs cannot be nested but may contain'
    3 :'echo''one-line explicit-defs or (multiline) DDs (see below)''' 0
)
fn 1
1 fn 2

NB. adverb representing number in percent ( ": formats arg as string)
echo 0.01 (1 : '( ": u * 100), ''%'' ')
percent =: 1 : 0        NB. same as a multiline definition
    (": m * 100), '%'   NB. using m to indicate left arg is noun
)
echo 0.7 percent

NB. conjunction that swaps its arguments and the args of its result
swap =: 2 : 'y v u x'
4 % 2                   NB. division
% 2                     NB. 1 divided by y
4 % - 2                 NB. 4 divided by negative 2
4 - % 2                 NB. operators swapped: 4 minus a half
2 - % 4                 NB. swap args too: 2 minus a quarter
4 % swap - 2            NB. equivalent but done by the conjunction swap
```

## Index-Functions, Helpers:

As demonstrated, explicit definitions specify the type to create as a
number. This pattern of selecting functionality with a numeric index is
sort of J's guilty pleasure. In most cases one would look up the
functions in the docs and assign an alias to often used ones; in fact, J
already comes with a set of aliases (and other helpers): list them with
`names_z_''`. New users should definitely look over the docs for `!:`
(foreign function index) and `o.` (circle functions) to see what's
available:
<https://code.jsoftware.com/wiki/Vocabulary/Foreigns>
<https://code.jsoftware.com/wiki/Vocabulary/odot#dyadic>

## Direct Definitions:

Direct definitions, *DD*s for short, are another way to write explicit
definitions. They are wrapped in double-braces and assume their type
from the argument-variable-names used:

- If variables `v` or `n` are used, a *conjunction*,
- otherwise, if `u` or `m` are used, an *adverb*,
- otherwise, if `x` is used, a dyad or ambivalent *verb* is created.

A type may also be forced by appending `)` and one of the letters
`n`oun, `a`dverb, `c`onjunction, `m`onad or `d`yad to the opening
braces.

The following examples contain more info about DDs and should be
compared carefully with their actual output!

```J
]fn =: {{123}}       NB. monad; internally represented as explicit-def
echo 1 2 3 , {{123}} ''
]fn =: {{x + y [ echo 'dyad because uses x' }}
1 fn 2

{{'>',{{)nstring}},{{)n123}},'<'}} 'nested; one-line string definitions'

{{ echo '---'
    echo 'untyped multiline DD'
    echo '* may start on opening line'
    echo '* may contain other mutliline DDs'
    {{ NB. such as this one
    echo '* closing line doesn''t have to start with }}'
    echo '* may continue on closing line' }} 'like here'
}} ''

{{)m NB. only comment allowed on typed opening line
    echo '---'
    echo 'typed multiline DD is like an untyped one except:'
    echo '* makes sure it is of a certain type'
    echo '* must not start on opening line (comments allowed)'
    y NB. last value is returned

}} {{)n  NB.(not a comment) except a multiline DD noun *may* do so.
multiline DDs which define nouns:
  * must be typed
  * always produce a single list of characters (a string)
  * everything (incl whitespace and comment on opening line) becomes
    part of the string, except:
}} , {{)n
    the opening line if it is empty (as in this case -> no empty line)
  * must put closing braces as first characters of line:
    }} thus this does not end the definition, but this does:
}} , '< * as the < indicated: always end in a newline'
```

## Arrays:

There is no datatype "list", it is just a term to describe a
1-dimensional collection of values.

Arrays can be thought of as nested lists, however _every_ value is
an array -- even lists which are not nested and values which are not
even lists (also called **scalars** or **atoms**).

Arrays must only contain atoms of the same datatype and all lists on the
same nesting-level (**dimension**/**axis**) must be of the same length!

Therefore an array can be described by its datatype and **shape**, which
is a list of the lengths of the lists on each nesting level. The length
of the shape is called the **rank** (or dimensionality) of the array.

An array which only consists of an atom which is not part of any list
has an empty shape and thus rank 0 ("0-dimensional"). A list of one atom
is a 1-dimensional array because the length of its shape (`1`) is 1;
wrapping this list in another list _prepends_ another dimension of
length 1 to the shape which is now `1 1` and thus rank 2.

While an array which only consists of an atom which is not part of any
list has an empty shape, an empty array, meaning one that does not
contain any atoms, must have at least one `0` in its shape. An array of
2 lists of 0 atoms each differs due to its shape (`2 0`) from an array
of 0 lists of 2 atoms each (shape `0 2`) despite both being empty
arrays.

The length of an array is the length of its highest dimension (leftmost
number in the shape) and always 1 for atoms!

```J
$ 0                 NB. monad $ gives shape; scalars have an empty shape
# $ 0               NB. and therefore rank 0; monad # gives arg's length
$ 0 1 2             NB. shape
# $ 0 1 2           NB. rank
2 1 $ 10 20         NB. dyad $ reshapes array; new: 2 lists of 1 element
$ 2 1 $ 10 20       NB. shape
2 $ 10 20 30        NB. reshaping may drop elements
5 $ 100 200         NB. or repeat the elements as needed
'ten', 20           NB. error: incompatible types
10, 3.14            NB. ok: both are numbers
2 2 $ 'abc'         NB. don't forget that strings are arrays too!
'a',LF,'b'          NB. strings 'a' and 'b' combined with a newline
NB. ^^ predefined variable containing newline, also useful: TAB,CR,CRLF
```

## Boxes, Trees:

The container-type box is useful to get around the restrictions of
arrays: A boxed value always appears as scalar of type box; therefore
any values, each in their own box, can be put into the same array!

A box not containing another box may be called **leaf** when talking
about **trees**, structures consisting of nested boxes. J comes with
special functions to simplify working with trees, such as `{::` to index
them (see: indexing).

```J
<3                  NB. monad < puts the value passed as arg into a box
><3                 NB. monad > opens/unboxes the outer box of a value
<''                 NB. an empty box is just a boxed empty array
a:                  NB. equivalent; called "ace", a noun

1 ; 3               NB. dyad ; joins args as boxes
1 ; <3              NB. it removes 1 boxing level from its right arg
(<1); <3            NB. but not from its left arg; therefore
(<1),<( (<2),<<3 )  NB. building trees with , helps seeing nesting level

1 2 , 'abc'         NB. error: different types and lengths
1 2 ; 'abc'         NB. ok because ; boxes the values first
(<1 2) , <'abc'     NB. here we manually boxed the values first -> ok

NB. multi assignments unpack top-level boxes:
]'foo bar' =: 0 1;2 NB. removes one boxing level but returns original
foo, bar

NB. comparison of boxed values
'bar' = 'baz'
(<'bar') = <'baz'
(<'bar') = <'bar'
'bar' = <'bar'
```

## (Explicit) Control-Structures and Control-Words:

J has all common control-structures and -words, however, they can only
be used within explicit functions (explicit definitions or DDs). They
are not idiomatic J and hinder new users from learning to think in array
programming style (see: idiomatic replacements for explicit
control-structures). Nevertheless, some problems are simpler to solve
this way.

- Assert: *All atoms* have to be `1`.
  ```J
  {{
      assert. 'aa'='aa' NB. dyad = compares corresponding atoms: 1 1
      assert. 'aa'='ab' NB. error because not all 1s: 1 0
  }}''
  ```
- Conditionals: *First atom* must not be `0`.
  ```J
  {{ if. 0 1 do.        NB. first atom is 0 -> false
      echo 'if block'
  elseif. 1 0 do.       NB. first atom is not 0 -> true
      echo 'elseif block'
  elseif. 1 do.         NB. only considered if no previous block ran
      echo 'elseif 2 block'
  else.                 NB. runs if no previous block ran
      echo 'else-block'
  end. }}''

  NB. therefore be careful with tests such as:
  {{ if 'bar' = 'baz' do. echo 'true' else. echo 'false' end. }}''
  ```
- Select: It first boxes unboxed arguments to `select.`, `fcase.`
  or`case.` then compares whether one of the boxes passed to `select.`
  is the same as a box passed to a `case.` or ` fcase.` statement and
  executes the respective block. An empty `case.` condition always
  matches. After the evaluation of a `case.` block execution jumps to
  `end.`, while after executing an `fcase.` (fallthrough) block the
  following `case.` or `fcase.` runs unconditionally.
  ```J
  match =: {{
  select. y
  case. 'baz' do. echo 'does not match ''bar'' due to both being boxed'
  case. 'bar' do. echo 'after case. jumps to end.'
  case. 1 2 3 do. echo 'due to the boxing only matches the list 1 2 3'
  case. 1; 2; 3 do. echo 'box it your