# Tuple
Tuple is a complex data type that groups multiple values together. Number of values it can holds is know and predefined, and each value is stored under know key.
It can be viewed as record of a table or JavaScript object.

Tuples can be defined using two syntax:

`(tuple (key1 value1-type) (key2 value2-type) (key3 value3-type) ... (keyN valueN-type))`

or
`{key1: value1-type, key2: value2-type, key3: value3-type, ... keyN: valueN-type}`

Second notation is much nicer, therefore I use only this one.


Tuples are useful if we want to store multiple information about something in map.
For example if we would like to store name and age for multiple principals we could do something like this:

```clarity
(define-map
    principal
    {name: (string-ascii 50), age: uint}
)
```


# Functions

## Extracting value from tuple
`get` is a function that extract value from tuple that is store under specific key.


```clarity
(define-data-var myTuple 
    {name: (string-ascii 50), location: (string-ascii 50), lucky-number: int}
    {name: "LNow", location: "Earth", lucky-number: 0}
)


(define-public (test-get)
    (begin
        ;; print whole tuple
        (print (var-get myTuple))

        ;; print only name
        (print (get name (var-get myTuple)))
        (ok true)
    )
)
```

## Merging two tuples
`merge` is super useful when we want to merge two tuples together or to replace value of single or few fields in one tuple.

```clarity
(define-data-var myTuple 
    {name: (string-ascii 50), location: (string-ascii 50), lucky-number: int}
    {name: "LNow", location: "Earth", lucky-number: 0}
)


(define-public (test-merge)
    (begin
        ;; print original tuple
        (print (var-get myTuple))

        ;; print original tuple merged with another tuple
        ;; notice that values from 2nd map has been added to original map
        (print (merge (var-get myTuple) {key1: u10, key2: u50, key3: "hello"}))


        ;; print original tuple merged with another that have same key
        ;; notice that 2nd tuple have same key as original one and value in original is replaced with new value
        (print (merge (var-get myTuple) {name: "Not LNow"}))
        (ok true)
    )
)
```