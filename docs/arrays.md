---
layout: default
comments: true
---

## Arrays

### Array Type

An array contains zero or more objects of the same type serially arranged in memory.

Array is a builtin type and it is the base type for the ARRAY type class.

The formal declaration is:

    builtin type ~ Array{ E, C : UInt } derives Any, 
                                                Collection{ E },
                                                Updatable{ UInt, E }

E is the element type, and C is the capacity measured in number of elements.


### Array Indexing, Length, and Bounds Checking

Arrays are indexed starting from index 0.

Arrays have a special member L which is the currently initialized length measured in number of elements.
L is always less than or equal to C.

For example:

    a1 := [ 1, 2, 3 ]
    assert a1.C == a1.L

Array indices up to L-1 may be read from, and indices up to L may be written to, as long as they are less than C.
If an element index beyound those bounds is accessed a runtime error is generated.


### Array Type Syntax

An array type is specialized like other generic types, for example:

    Array{ UByte, 3 }    ## constant
    ~Array{ ~UByte, 3 }  ## modifiable
    ~Array{ UByte, 3 }   ## Error - inconsistant modifiability!
    Array{ ~UByte, 3 }   ## legal, can occur as member in other generic types,
                         ##  but is effectively constant

Array types can also be specialized using [] syntax, for example:

    [3]UByte    ## constant
    ~[3]UByte   ## modifiable (implicitly modifiable UByte elements)
    ~[3]~UByte  ## modifiable


### Array Value Literals

The first element in an array literal specifies the array element type.
The remaining elements will be cast to that type.
If an element can't be implicitly case to that type, an error is generated.

An array literal is enclosed in brackets and contains at least one value expression.
(An empty array literal can't be expressed this way, the element type would then be unspecified.)

The expression list can optionally be followed by an extra comma, this will not cause an extra element to be specified.
If the list has only one element, the comma is mandatory, or the backslash notation may be used -
preceeding the opening bracket with a backslash.

```
array_literal :
    [ <exp>, ]
    [ <exp>, <exp_list> ]
    [ <exp>, <exp_list>, ]
    \[ <exp> ]
    \[ <exp>, ]
    \[ <exp>, <exp_list> ]
    \[ <exp>, <exp_list>, ]

exp_list :
    <exp>
    <exp_list>, <exp>
```

In order to initialize a modifiable array via a value literal, the modifiability operator is prefixed the entire
array like so:

    a52 := ~ [ 1, 2, 3 ]


### Explicit Array Initialization

In order to express an empty array value, the constructor syntax must be used:

    a := [0]Int();           ## empty array of Ints

The constructor syntax can by used for all array types.
The element initializers can be specified via an array, or as individual arguments.
The following are equivalent:

    a1 := [ Int(1), 2, 3 ]
    a2 := [3]Int( 1, 2, 3 )
    a3 := [3]Int( [ 1, 2, 3 ] )

Arrays of dynamic length (i.e. not known until runtime) can be constructed.
An array can also be initialized with zero of more elements, up to the array's capacity.

    a31 := ~[i*3]Int()                 ## modifiable, initially empty
    a32 := Array{Int,i*3}()            ## initially empty
    a31 := [i*3]Int( [ 1, 2 ] )        ## if i == 0, a runtime error is generated
    a32 := Array{Int,i*3}( [ 1, 2 ] )  ## if i == 0, a runtime error is generated

The above examples create arrays on the stack. To create an array on the heap, a new-expression is used.

    ## allocates a modifiable array of 5 UByte elements,
    ## of which 2 are immediately initialized
    arr := new ~[5]UByte( [ 1, 2 ] )


### Array Equality

Arrays can be value-compared using the regular equality operators == and !=.

Two arrays are equal if and only if they have the same element type, the same length (number of elements),
and the two elements at each index of the two arrays are equal.

Note that the arrays' capacity does not affect the value equality.


### Methods

Array derives Any, Collection, and Updatable. It implements these methods:

    override equals( other : &Any ) -> Bool

    override empty() -> Bool

    override count() -> Ordinal

    override capacity() -> Ordinal

    override contains( val : E ) -> Bool

    override add( val : E ) ~ -> Bool

    override clear() ~

    override sequencer() -> Ref{ ~Iterator{ E } }

    override updater() ~ -> Ref{ ~Updater{ E } }

    override has( ix : UInt ) -> Bool

    override get( ix : UInt ) -> E

    override set( ix : UInt, element E ) ~ -> E

    override swap( ixA : UInt, ixB UInt ) ~

See tx/array.tx for further details.
