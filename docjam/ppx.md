# A Guide to PPX Extensions

Author: Perry Metzger @pmetzger

> This is a _very very_ rough work in progress. Please help the author
  improving it.

OCaml provides a _syntactic extension facility_ known as
PPX.

"Syntactic extension" means a way of adding entirely new features to
the OCaml language that would otherwise be impossible.

For example, the `ppx_regexp` extension adds a **match**-like
construct that matches strings with regular expressions, like:

```ocaml
match%pcre somestring with
  | "^foo"       -> some_expression1
  | "(bar){2,3}" -> some_expression2
  | _            -> some_default
```

PPX extensions are implemented as OCaml code that plugs in to the
compiler. The extensions look for small syntactic "hooks" that signal
that an extension should do its work. These syntactic signals that
extensions look for are defined by the OCaml language specification,
but mean nothing to the compiler itself without the plugin.

This document is presented in three parts. The first part gives a
brief informal explanation of how PPX extensions are used. The second
part gives some detail on how PPX works and the syntactic hooks OCaml
provides for use by PPX extensions.  The third section provides some
detail on how you can write new PPX extensions, and gives pointers to
other documentation that may be of interest to extension writers.

## How PPX extensions are used

We have already seen an example of the `ppx_regexp` extension, which
creates a version of the `match` construct that checks strings for
regular expression matches.

Another typical PPX extension is `ppx_deriving`, a family of
extensions that automatically generate code from OCaml data
structures. For example, if you add the annotation:

```ocaml
[@@deriving show]
```

to a data type, as in:

```ocaml
type point3d = float * float * float
[@@deriving show]
```

the `deriving show` plugin to `ppx_deriving` will construct a `show`
function for the `point3d` type that will pretty print it.

The standard OCaml compiler knows nothing about the `ppx_deriving`
extension, of course. The extension exists as a plug-in for the
compiler, and the compiler must be invoked in such a way as to run
that extension over your code.

The documentation for an extension will typically explain how to tell
the compiler how to invoke the plugin. (For example, for
`ppx_deriving` and its `show` component, one usually adds the flags
`-package ppx_deriving.show` when invoking `ocamlfind`.)

## How does a PPX extension work behind the scenes?

The OCaml compiler operates in several steps. When it reads your
program, the first thing it does is convert the OCaml code into an
internal data structure called an "Abstract Syntax Tree", or "AST."

When you add

```ocaml
[@@deriving show]
```

or syntax from the `lwt` extension

```ocaml
let%lwt foo = ...
```

or the like to your code, the OCaml compiler records the presence of
the `@@` or `%` or similar constructs in your code in the AST.

PPX syntactic extensions then walk the Abstract Syntax Tree looking
for constructs they are intended to respond to. (For example, the
`ppx_deriving` framework will look for attribute nodes labeled with
`@@deriving`, and the `ppx_regexp` extension will look for extension
nodes with the name `%pcre`.)

When they find such a construct, the extension can take fairly
arbitrary actions in response, though typically the extension will
rewrite a section of the the OCaml AST. (For example, `ppx_deriving`'s
`show` plugin will examine the AST of the type that has been annotated
`[@@deriving show]` and will then insert an appropriate `show`
function customized for the type into the AST.)

### Syntactic hooks for PPX extensions

Among other "hooks" are:

1. Operator names starting with a `#` character and containing more than
   one `#` character are reserved for extensions.
2. Int and float literals followed by an one-letter identifier in the
   range `[g..z|G..Z]` are reserved for extensions.
3. _Attributes_ [Insert explanation of attributes]
4. _Extension nodes_ [Insert explanation of extension nodes]

In addition, "Quoted" strings of the form `{|string|}` or
`{foo|string|foo}` are not per se reserved for extensions, but are
often of use in them because they allow for strings containing
unescaped `"` and similar characters.

## Writing PPX Extensions

_This section currently intentionally left blank. Please help me turn
it into something useful._