# Response
`response` is a complex data type that is quite similar to enum data type used in other languages, with the difference that `response` data type is always either `(ok <something-ok>)` or `(err <something-err>)`

It is a good practice to follow convention defined for build-in functions, therefore if user-defined function returns `response` type `err` part should return `uint` value ie.

`(response <something-ok> uint)`

`something-ok` can have any data type. `uint`, `int`, `string-ascii`, `tuple`, `list` it can be anything.