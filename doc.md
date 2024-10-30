# The Case Against (X,HT)ML

* complex opening and closing tags
  * Redundancy of tag names
* only strings as attribute values
* Ability to not open and close tags in order
* self-closing tags omit self-close such as `<br>`.
  * It can be hard to determine whether a tag is without content (`br`) or incorrectly closed (`p` without `/p`).
* lack of strings and specialised text types awkwardly collapses whitespace
* extensions are rather clumsy, everyone uses `div`.

## The Solution

A language based on the syntax of C and Rust to organise XML (and subsequently XML) into a simpler, less redundant language.

### Open Questions

* nssymb: The namespace resolution operator is subject to change, as I dislike the "::".
* elemsep: Do lists of elements need semicolon separators?
  * for: more familiar to C and other languages.
  * against: unnecessary, added characters.
* listsep: see elemsep.
* headername: what should headers be called, headers, modules?

### The Features of the Language

#### Language Syntax

elements are given by an optional namespace and resolution operator, an identifier, an optional list of key-value pairs in parentheses and an optional block of further elements contained within braces.

```newxml
path::ident(attr1=literal, attr2=element) {
    element
    element
}
```

Comments are between `//` and a newline `\n` or `/*` and `*/`

Whitespace is collapsed and ignored

Allowing literals to be elements in their own right allows for attribute values to be themselves be elements. This allows for more expressive attributes.

```newxml
path::ident(attr1=element, attr2=element) { /* ... */ }
```

#### Bringing Namespaces into Scope

Identifiers can be brought into scope with the `use` directive, which references either a local path, a URL or a standard library element which is a part of the specification. It may use braces to avoid repetition and the asterix to bring all contents into scope.

```newxml
use "path/to/local/homespec.ns"::myelem

use "https://address.tld/file.ns"::*

use web::{web p}
```

#### Block as a Type

the elements inside the (optional) block are arranged as an ordered list, which is a natural extension to the type system. With the block afterwards having it's own special type, the block after the element becomes syntax sugar for a new `block` attribute present in all elements which demand or allow internal content.

```newxml
path::ident() {
    element
}
```

is therefore sugar for

```newxml
path::ident(block = {element})
```

#### Writing Definitions

the `elem` keyword starts an element definition followed by the identifier. An optional pair of parentheses contains a list of key: type, with an optional block at the end:

```newxml
elem ident(
    attr1: bool
    attr2: int
    attr3: str

    /* ... */

    block
)
```

where `block` is a keyword to allow for the block syntax sugar and marks where an element expects children.

to facilitate transpilation between this language and others, the impl syntax can be used.

```newxml
impl Into<HTML> for ident {
  /* ... */
}
```

the text within the braces is transcribed exactly during transpilation, with the excepction of characters within unescaped braces. These braces may access attribute values using their name.

```newxml
elem br

impl Into<HTML> for br {
    <br/> 
}
```

Each primitive type has a set of built in conversions, for example `str` to UTF-8 or `int` to 32-bit Big Endian. These representations are accessed though the `.` operator.

```newxml
elem a(
    destination: str
    label: str
)

impl Into<HTML> for a {
    <a href="{destination.urlencode}">{label.utf8}</a>
}
```

To facilitate custom indent styles, from each line leading whitespace is trimmed according to the line with the minimum leading whitespace of the block (excluding blank lines or lines only consisting of whitespace)

```newxml
elem complex_elem

impl Into<format> for complex_elem {
    some text here
    a newline there
        some curly braces \{ with text within \}

    and a fully blank line above.
}

/* this element becomes:
some text here
a newline there
    some curly braces { with text within }

and a fully blank line above.
*/
```

For elements with a block attribute, that block will have each element printed sequentially with a configurable separator. To this end, any primitive type that exists in the block also needs an impl for the relevant format.

```newxml
elem p(
    block
)

impl Into<HTML> for p {
    <p> {block(sep=" ")} </p>
}

impl Into<HTML> for str {
    {utf8}
}
```

In the example above, the p can only have str (or other elements with an `Into<HTML>` implementation). The `Into<HTML>` for `str` is also used for the separator when converting that from the primitive type to HTML.
