# Lists
Lists are complex data types. They are defined by maximum number of elements they can hold and exact data type of those elements.

Lists can hold one and only one type of elements.

Definition: `(list max-len entry-type)`

`max-len` is defined using `int` value and must always be greater or equal 0.

`entry-type` can take form of any other data types (including another list).

List values are separated with **space** not **coma**.

```clojure
;; list that can hold up to 10 uint's
(define-data-var myList (list 10 uint) (list u1 u1 u2))

;; list that can hold up to 100 asci strings that can be up to 20 characters long each
(define-data-var myOtherList (list 100 (string-ascii 20)) (list "halo" "hi" "good morning" "gm"))
```

## Functions

### Appending
We can add new element at the end of the list using function append.
This function always returns **new** list of the same type as input list but with `max-length` **increased by 1**.

```clojure
;; list that can hold up to 4 uint's, and by default holds 3 values; u1, u2 and u3
(define-data-var myList (list 4 uint) (list u1 u2 u3))

(define-public (test-append)
    (let
        (
            ;; first we add new value to myList and store new list in variable newList
            (newList (append (var-get myList) u4))
        )
        ;; print content of newList
        (print newList)
        ;; print length of newList
        (print (len newList))
        ;; lets try to save newList in myList global variable
        (var-set myList newList)
        (ok true)
    )
)
```
Above code will not work. It will fail to "compile" with following error: `expecting expression of type '(list 4 uint)', found '(list 5 uint)'`

Why?
`myList` is defined as list that can hold up to 4 elements and by default it contains only 3.
When we `append` new element to this list we get new list with larger `max-len` limit. So `append` function transforms `(list 4 uint)` to `(list 5 uint)` and that is why we can't store our new list in `myList` variable.

To solve this problem we have to convert our `newList` to matching datatype using `as-max-len?` function.

```clojure
;; list that can hold up to 3 uint's, and by default holds 3 values; u1, u2 and u3
(define-data-var myList (list 4 uint) (list u1 u2 u3))

(define-public (test-append)
    (let
        (
            ;; first we add new value to myList and store new list in variable newList
            (newList (append (var-get myList) u4))
        )
        ;; print content of newList
        (print newList)
        ;; print length of newList
        (print (len newList))
        ;; lets try to convert our list to have max-len=4 and save it in myList global variable
        (var-set myList (unwrap! (as-max-len? newList u4) (err "failed to change max-len")))
        (ok true)
    )
)
```