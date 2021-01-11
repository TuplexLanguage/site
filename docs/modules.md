---
layout: default
---
## Modules

Modules are the namespaces of Tuplex. Their names are dot-separated identifiers, for example `tx.c`. 
The namespaces form a hierarchy with the root namespace "" at the top.
(Entities - fields and types - cannot be declared in the root namespace, only modules.)

A source file has this high-level grammar:

    module <module_name> (optional)

    import_statements (optional)

    module_members


### Module Names

The module name does not need to be unique per file - multiple files can declare the same module name
and their contents will be added to that module. This way a module may be split up across several source files.

If a file has been explicitly specified on the compiler command line, it may declare any module name except for
the reserved namespace `tx`. 

Files that are indirectly loaded - import statements causing them to be
loaded from the source file path list - must declare the module name expected of that file.

The module declaration is optional. If it is not present the file is considered the $local namespace (see section below).

Module members are any number of:

- Type declarations
- Global field declarations
- Submodules

They can be declared in any order.

Submodules within a file are declared with similar grammar as the top-level file module, with the difference that they define a
sub-scope which starts with a colon and is demarcated using indentation (or braces, see indentation syntax). For example:

    module my.file.mod
    import tx.c.puts
    
    module my.file.mod.submod:
        import another.mod.*
    
        SUB_FIELD := c"42"
    
    main():                       ## this is outside the submodule, in my.file.mod
        puts( submod.SUB_FIELD )  ## relative qualified identifier

Submodules may contain submodules of their own, and so on. However it is usually inadvisable to nest them within
the same file.

Submodules declared within another module in the same file must be sub-namespaces of it,
i.e. start with the parent module's name.
A submodule declared with a *simple name* (unqualified, without any dot separators)
will automatically be have the parent module's name as its qualifier.
Submodules declared with a qualified name (two or more dot-separated simple names) are required to be
fully qualified names, and must match the parent module name in full, as `my.file.mod.submod` above.


### Importing

The import statements bring in other namespaces and enable referring to their fields and types by their simple name,
without having to write the fully qualified name every time. For example:

    ## import the puts function from the tx.c module, which can then be referred to with 'puts' in this file/submodule:
    import tx.c.puts
    
    ## import all the names in the tx.c module:
    import tx.c.*

When an import statement is encountered, the compiler will look for the source files for that module.
For each directory in the source path list, it will look first for a `.tx` file with the exact same name as
the fully qualified module name (e.g. `my.file.mod.submod.tx`).
If such a file is not present, it will look for a directory tree matching the module name (e.g. `my/file/mod/submod/`)
and load all the `.tx` files contained therein.

This way simple modules, test modules, and early prototype modules can simply be put in their own files,
and larger modules can be split up across multiple source files in their own directory.

The source path list by default contains the current directory where the compiler is being run.
It may be specified using the -sp command line argument, or with the `TUPLEX_PATH` environment variable.
(The command line takes precedence over the environment variable.)


### The `$local` Module

If there is no module declaration at the start of a file, it is considered the $local namespace.

Within $local, fields and types can refer to each other via their unqualified names, as usual within a module.
However it is not possible to refer back to $local from other modules ($ is not a legal identifier character in
source code). The $local module can thus only be a starting point for a program.

It is an error for any source file except *the first one processed in a compilation package* to not specify a module.
Note that the $local name cannot be declared as module name in source code.

These rules are to provide both simplicity and security.
It's easy to write small single-file scripts and programs without declaring module names;
to have a bunch of them in the same directory without name collisions;
and to seamlessly evolve them by adding other files (with module names) if they grow.

Since $local is effectively an anonymous namespace, and often the place for the main() function,
it would be more vulnerable to name conflation and malicious code injection.
But since only the first file in a compilation package may contain $local, this particular method of code injection
is impossible without full access to the compilation environment.


### The `tx` Module

The tx module contains the 'standard library' of Tuplex and is imported by default. 
(This can be disabled with a compiler switch.)
It includes such things as string formatting, ranges, sequences, iterators and collections.
It also enriches the feature set of the built-in type Array.

It contains the submodule `tx.c` for C library inter-op, and `tx.os` for file handling.
`tx` submodules are *not* imported by default, they need to be explicitly imported from the using files.

It is not permitted to declare entities in the `tx` namespace from user code.
