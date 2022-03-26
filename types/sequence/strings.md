# Strings
There are two types of strings that we can use in clarity `string-ascii` and `string-utf8`

Both have predefined `max-length` that can't be exceeded.

Strings in Clarity are treated as subtype of `sequence` type, because they are just a sequence of characters. Because of tha all functions that can be used with any other sequence types (lists, buffers, ) can be also used with strings.


## string-ascii
Definition: `(string-ascii max-len)`

`max-len` is defined using `int` value and must always be greater or equal 0.

Ascii strings always are enclosed within double quotes.

```clarity
(define-data-var myString (string-ascii 12) "hello world")
```

## string-utf8
Definition: `(string-utf max-len)`

`max-len` is defined using `int` value and must always be greater or equal 0.

UTF-8 strings always are enclosed within double quotes. Opening quote must be preceded with letter `u`
```clarity
(define-data-var myString (string-utf8 12) u"hello world")
```
