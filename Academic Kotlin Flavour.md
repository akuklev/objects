## Motivation

There is a dream to make Kotlin a programming language suitable for every purpose in any context. In this memo, I explore what can be done to design a Kotlin flavour suitable and appealing to replace various ad hoc flavours of pseudocode used in academic and educational contexts.

## Significant indentation and the block-structure-first approach

Owing to the universal popularity of Python in the scientific community, significant indentation should be used for multiline code blocks. When reading the code, people see indentation-based block structure before anything else. Thus, contrary to Python, we want block structure to have priority over comments, string literals, and brackets. We propose to fix block indentation to two whitespaces once and for all, while treating any other number of indenting whitespaces as a continuation of the previous line. This allows using indented multi-line argument lists and similar constructions, as those always have longer indents:

```
fun example(files : List<File>,
            target : File)
  ...
  ...
  return something + somethingElse + somethingOther +
   yetSomething + rest
```

Single-whitespace indentation should be the preferred way to wrap long lines. In case of consequent dedents, black square `■` may be used as a visual aid:

```
fun main(args : List<String>)
  for (arg in args)
    println(arg)
  ■  
```

**With the block-structure-first approach, the lexer can detect indents and dedents without recurring to obscure techniques to bypass comments and quotations and look for unmatched brackets. This massively improves the performance of incremental parsing.** 

## Literate Kotlin + Plain Text Format for Kotlin Notebooks

Since mandatory indentation is used for all non-inline blocks, the only unindented code lines in ordinary Kotlin source files always begin with an annotation or a keyword (`import`, `class`, `fun`, etc). Should we prepend `@` to those lines not yet starting with `@`, we can freely interleave Kotlin source code with surrounding text accompanying the code written in a desired flavour of Markdown or TeX.

Instead of having comments as second-class citizens in source files, we treat text and code on par.

Many CS researchers write both their paper and their projects as LiterateAgda sources, where Agda code is interleaved with text, so that sources can be both compiled and rendered as a pdf for human reading. Our approach achieves the same, yet in a more elegant way.

We can also incorporate interactive internal documentation within the same format by introducing blocks of the form

```
@do
  ourFunction(1, 3)
```

We may also allow any custom parameters for those blocks:

```
@do(collapsed: true, autoexec: false)
  someComputation
```

Such blocks should be rendered as in internal REPL in the IDE so the user can edit and rerun them to play with the context of the source defined so far. The IDE should support rich (visual, animated and even interactive) output of such blocks, so that literate sources are suitable for Kotlin-notebooks. 

By adding optional blocks of expected output, we obtain a nice way to provide unit tests

```
@example Addition1
  1 + 2 + 3
@expect
  6
```

## Fancy operators, compliance with mathematical notation

Mathematicians love to use fancy non-ASCII operators, e.g. `≠`. In academic Kotlin, we should allow using non-ASCII aliases for common operators. IDE should automatically substitute usual digraphs by non-ASCII operators, e.g. != → `≠`. 

In particular, we propose to display

* `x.let f` as `x ▸ f` and `x?.let f` as `x?▸ f` or `x ?▸ f` ;
* the respective comparison operators as `≤`, `≥`, `=` and `≠`;
* logical 'not', 'and', and 'or' operators as `¬`, `∧`, and `∨`;
* `·` as alias for multiplication;
* `↦` in lambda-expressions, e.g. `{x ↦ x + 1}`,

The assignment operator should be then displayed as `≔` when introducing a fresh name
(e.g. `val a ≔ 5`) and for default argument values, or by an immediate colon `key: value` for named arguments and other “key-value” cases.

As it is customary in mathematics to have whitespaces around relation symbols and symbolic symmetric binary operators such as `+` (except for `a·b`), we require that for all such operators including the typing relation as in `n : Int`.

Additionally, any binary (or vararg) function should be allowed to be used as an infix operator by surrounding it by chevron quotation marks, e. g. `a ‹and› b` , `2 ‹Nat.plus› 3`.

*NB: We should consider using `#` followed by a whitespace and displayed as `■` for end-of-line-comments instead of \`//\`. The latter should be reserved for integer division as in Python. While following a long tradition of ALGOL-family languages, it looks weird for many people.*

## Disambiguating methods and properties

In Kotlin `obj.name(...)` can mean invocation of the method `name` of `obj` or extraction
of its property `name` of a callable type and its application. One cannot find out which one is used without looking up the definition of the class of `obj`, and this is ambiguity is a major problem in an academic context.

To resolve this ambiguity and to improve typographical properties of the code, we suggest reserving dots `.` per se are reserved for properties only and use `▸` in case of method calls following the long tradition of using arrows for method calls started by PL/I in the late 60s:

```
files ▸dropLast(n) ▸withIndex ▸last  
```

```
fun example(files : List<File>,
            target : File)
  files ▸filter
    it.size > 0 &&
    it.type = "image/png"
  ▸map { it.name }
  ▸withIndex ▸map fun(idx, item) 
    ...
  ...
```

Since triangles are not present on the standard keyboard, so we assume the IDE to convert the dots into such triangles when appropriate.

## Semantic considerations

Academic pseudocode languages do, in general, assume the default integer type to be overflow-free (as in Python), the operator `/` to be the true division operator even when both operands are integer. For integer division, an additional operator should be introduced.

In accordance with their mathematical semantics, expressions like `1 + 1` are interpreted as `Int.plus(1, 1)` rather than `1.plus(1)`, i.e. arithmetical operators are considered to be functions belonging to companion objects of the given numeric type rather than methods of number objects themselves.

Since we mentioned companion objects containing operators like “plus”, we should also mention that the notion of type-classes is indispensable in many academic contexts. In Kotlin, one can define
both nested classes and extension functions. The type-classes are, in a sense, extension nested data classes with quite a bit of additional syntactic sugar.

Consider the following definition of a monoid-structure on a type `T`:

```
data class <T>.Monoid(operator val ‹compose› : (vararg xs : T)-> T)
  val unit ≔ compose() // unit is the nullary composition
  
  contracts {
    unit ‹compose› x = x
    x ‹compose› unit = x
    x ‹compose› y ‹compose› z = x ‹compose› (y ‹compose› z)

    compose(x, *xs) = x ‹compose› compose(*xs)
  }
```

With such a definition, we now can write functions like this:

```
fun<T : Monoid> square(x : T)
  x ‹T.compose› x
    
or, equivalently
    
fun<T : Monoid((*))> square(x : T)
  x * x
```

## End-of-line literals and block literals

In Kotlin, there is a special syntax for code block arguments. Instead of `a.map(::println)` one can write `a.map { println(it) }`. We propose to allow the same for `String` arguments and introduce
special syntax for end-of-line string literals and block string literals:

```kotlin
fun greet(name : String)
  println~ Hello, $name!
  println
  ~ Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    Cras rutrum rhoncus vulputate. Sed sit amet vulputate leo,
    nec hendrerit turpis.
  ~ Aliquam erat volutpat. Nam nisl neque, condimentum ornare
    ornare eget.
```

Here we use tilde with a mandatory trailing whitespace which is necessary to disambiguate from enclosing backticks for complex identifier names.

In multiline text blocks, base level indentation is removed, and line breaks are replaced by whitespaces. If a line break should be retained, start the next line block by another `~ ` (see above), when a line break should be removed completely, use one whitespace less of indentation.

End-of-line and block literals also apply to templates (i.e. strings with additional context `Ctx.()->String`) and other kinds of literals should they appear in Kotlin.

EOL-literals play nicely with key-value lists:

```kotlin
address: Address
  country:~ NO
  city:~ Oslo
  street: ...
```

One could consider using inline `~` with without the trailing whitespace for single-keyword-litkerals:

```kotlin
allowedCountries: listOf(~NO, ~NL, ~DE)
```
