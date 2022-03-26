# Buffers
Buffer type is used to hold byte buffer data. Buffers are defined with pre-defined max length that can't be changed.

Buffers in Clarity are treated as subtype of `sequence` type, because they are just a sequence of bytes. Because of tha all functions that can be used with any other sequence types (lists, strings ) can be also used with buffers.

Definition: `(buff max-len)`

`max-len` is defined using `int` value and must always be greater or equal 0.

```clarity
(define-data-var myBuffer (buff 10) 0x0102ab01)
```

Empty buffer looks like this: `0x`
