# Numeric
There are only two numeric data types in Clarity. `uint` and `int`. Both are represented using 128 bits.

There are no floats, doubles, decimals etc.


## int - signed 128-bit integer
**MIN VALUE:** `-170141183460469231731687303715884105728`

**MAX VALUE:** `170141183460469231731687303715884105727`

## uint - unsigned 128-bit integer
**MIN VALUE:** `u0`

**MAX VALUE:** `u340282366920938463463374607431768211455`

# Functions
If functions takes 2 or more arguments they all have to bee the same type. Either `uint` or `int`
Functions returns same data type as the one used for arguments.

All math functions can panic with overflow/underflow error.

Conversion function can also panic with overflow/underflow error.

### Subtraction
```clarity
;; int
(- 10 200)

;; uint
(- u300 u10)
```

### Addition
```clarity
;; int
(+ -238 49838)

;; uint
(+ u123 u9)
```

### Multiplication
```clarity
;; int
(* 2 19)

;; uint
(* u9 u9)
```

### Division
Division **rounds down** returned value to the nearest value.

Division panics iw we try to divide by `0` or `u0`.
```clarity
;; int
(/ 9 5)

;; uint
(/ u9 u5)
```

### Modulo
Returns reminder from division.

```clarity
;; int
(mod 2 3)


;; uint
(mod u7 u3)
```

### Log2
```clarity
;; int
(log2 8)

;; uint
(log2 u12)
```

### Power
```clarity
;; int
(pow 2 3)

;; uint
(pow u3 u4)
```

### Square root
Returns the largest integer that is less than or equal to the square root of `n`. Fails on a negative numbers.
```clarity
;; int
(sqrti 19)

;; uint
(sqrti u9)
```

### Exclusive-or
Returns value calculated by applying bitwise [exclusive or](https://en.wikipedia.org/wiki/Exclusive_or) function.
```clarity
;; int
(xor 123 9)

;; uint
(xor u123 u18)
```

### Conversion to uint
Accepts only `int` value and converts it to `uint` if thats possible.

Panics when negative value is supplied.
```clarity
;; int
(to-uint 123)
(to-uint 0)
(to-uint -1) ;; this one will panic
```

### Conversion to int
Accepts only `uint` value and converts it to `int` if thats possible.

Panics when value overflows.
```clarity
;; uint
(to-int u123)
(to-int u0)
(to-int u340282366920938463463374607431768211455) ;; this one will panic with overflow error
```