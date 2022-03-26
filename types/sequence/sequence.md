# Sequence
Sequence is a meta type used in clarity. It is not a normal data type per-se, but rather an alias that groups all data types that represents sequence of something.

* `string-ascii` - sequence of ASCII characters
* `string-utf8` - sequence of UTF8 characters
* `buff` - sequence of bytes
* `list` - sequence of a predefined data type (ie. sequence of unit's, sequence of asci strings, sequence of buffers, sequence of tuples etc.)


All sequence types are defined with `max-len` expressed as `int` value that  must always be greater or equal 0.

I have no clue why someone would like to define 0 length sequence, but it is possible.

# Functions

## Changing max-len
`as-max-len?` can be used to convert sequence with max-len X to sequence of the same type but with max-len Y.

`(as-max-len? sequence max_length)`

If sequence already contains more elements than `max_length` function returns `none`. Otherwise it returns `(some <new-sequence>)`

Super useful when we want to append element to a list, or concat two small sequences and then store them in a variable with known predefined max-len.

## Concatenation
`concat` concatenates two sequences of the same type and returns it as new sequence with `max-len` that is equal sum of max-len of sequences used as inputs.

`(concat sequence1 sequence2)`

If `sequence1` is defined with max-len = 10, and `sequence2` with max-len=200, then sequence returned by `concat` function will have max-len=210

```clarity
(concat (list 1 2) (list 3 4)) ;; Returns (1 2 3 4)
(concat "hello " "world") ;; Returns "hello world"
(concat 0x0102 0x0304) ;; Returns 0x01020304
```

## Getting element at specific index/position
`element-at` returns sequence element at specific position/index.

If element is found it is returned as value wrapped with `some` ie. `(some <element>). Otherwise it returns `none`

`(element-at sequence index)`

Useful in data-types conversion (uint -> byte, uint -> string etc.)

## Filtering sequence
`filter` applies function on all elements of a sequence. If this function returns `true` for an element, then this element will be added to output sequence, if it returns `false` it will be omitted.

`(filter func sequence)`

Function `func` can take one, and only one input argument and its type must be the same as the type of sequence element. It can be build in function or user-defined one.

**Example:** We have a list of uint's and would like to remove from it all values smaller than 3 we could it like this:

```clarity
(define-data-var myList (list 10 uint) (list u1 u8 u3 u4 u5 u2 u6 u7 u9 u10))

(define-private (is-greater-than-3 (input uint))
    (> input u3)
)

(define-public (test-filter)
    (begin
        ;; our input list
        (print (var-get myList))

        ;; list with values greater than 3
        (print
            (filter is-greater-than-3 (var-get myList))
        )
        (ok true)
    )
)
```

## Fold
`fold` applies recursively same function on all elements of the sequence and returns one and only one value at the end.

`(fold func sequence_A initial_B)`

Function `func` must take 2 and only 2 arguments. First arguments must have exactly the same data type as element of our sequence. Second argument must have exactly the same data type as value we want to return and `initial_B`. It can be build-in function or user-defined one.

`initial_B` is an initial return value. It defines `fold` return data type and data type of 2nd argument of `func`.

**Example:** We have a list of uint's and would like to sum only values between 3 and 7.

```clarity
(define-data-var myList (list 10 uint) (list u1 u8 u3 u4 u5 u2 u6 u7 u9 u10))

(define-private (sum-between-3-and-7 (input uint) (return-value uint))
    (if (and (> input u3) (< input u7))
        ;; if input is between 3 and 7 we add it to return-value
        (+ return-value input)
        ;; otherwise we return initial-return-value
        return-value
    )
)

(define-public (test-fold)
    (begin
        ;; our input list
        (print (var-get myList))

        (print
            (fold sum-between-3-and-7 (var-get myList) u0)
        )
        (ok true)
    )
)
```

We start with `return-value` = `u0` as it is defined in fold function.

| input | return-value | input between 3 and 7 | value returned by function
|-|-|-|-|
|u1|u0|false|u0|
|u8|u0|false|u0|
|u3|u0|false|u0|
|u4|u0|true|u0 + u4 = u4|
|u5|u4|true|u4 + u5 = u9|
|u2|u9|false|u9|
|u6|u9|true|u9 + u6 = u15|
|u7|u15|false|u15|
|u9|u15|false|u15|
|u10|u15|false|u15|

`input` in ever iteration is taken from our list `myList` and `return-value` is either equal to our initial value defined in `fold` or value returned by previous iteration.

## Finding index at which element can be found.
`index-of` function returns **first** index at which we can find specific element in our sequence.
If element is found, then returned index is wrapped with `some` ie. `(some u14)` otherwise function returns `none`.

`(index-of sequence item)`

Datatype of `item` must be exactly the same as data type of elements stored in `sequence`.

Useful in datatype conversions, or testing if particular element exists in our list or not.

## Peeking how much elements are stored in sequence
`len` returns number of elements stored in sequence. It should not be confused with `max-len`.

If sequence is defined with `max-len` = 10, and it contains only one element, then `len` will return `u1`.

```clarity
(define-data-var myList (list 10 uint) (list u1))

(define-public (test-len)
    (begin
        ;; our input list
        (print (len (var-get myList)))
        (ok true)
    )
)
```

## Map
`map` applies function on all corresponding elements of input sequences and returns list of elements that have same type as data type returned by supplied function.

Returned list have always same amount of elements as the smallest sequence supplied.

`(map func sequence_A sequence_B ... sequence_N)`


`func` can be either build-in function or user-defined function.

Some build in functions works on dynamic number of arguments ie. addition `(+ u1 u2 u3 u4 u5 .... n)`, but user defined functions don't. Therefore user-defined function must be tailored to number of sequences we want to work with. If we want to work on 2 sequences - UDF must take 3 input arguments, if we want to work with 10 sequences - UDF must take 10 input arguments etc.

**Example:** 
```clarity
(define-data-var myListA (list 10 uint) (list u1 u2 u3 u4 u5 u6 u7 u8 u9 u10))
(define-data-var myListB (list 10 uint) (list u10 u9 u8 u7 u6 u5 u4 u3 u2 u1))
(define-data-var myListC (list 10 uint) (list u11 u4 u9))

(define-private (pick-greater (inputA uint) (inputB uint))
    (if (> inputA inputB)
        inputA
        inputB
    )
)

(define-public (test-map)
    (begin
        ;; pick greater values only from myListA and myListB
        (print (map pick-greater (var-get myListA) (var-get myListB)))

        ;; pick greater values from myListB and myListC
        (print (map pick-greater (var-get myListB) (var-get myListC)))

        ;; sum values on each index from 3 lists
        (print (map + (var-get myListC) (var-get myListB) (var-get myListC)))
        (ok true)
    )
)
```
