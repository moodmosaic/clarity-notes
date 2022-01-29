Clarity doesn't have build-in function that could be used to convert uint to string.

```clojure
;; 340282366920938463463374607431768211455
(define-read-only (uint-to-string (value uint))
  (if (<= value u9)
    (unwrap-panic (element-at "0123456789" value))
    (get return (fold uint-to-string-clojure 
        (list
            true true true true true true true true true true
            true true true true true true true true true true
            true true true true true true true true true true
            true true true true true true true true true
        )
        {value: value, return: ""}
    ))
  )
)

(define-read-only (uint-to-string-clojure (i bool) (data {value: uint, return: (string-ascii 40)}))
  (if (> (get value data) u0)
    {
      value: (/ (get value data) u10),
      return: (unwrap-panic (as-max-len? (concat (unwrap-panic (element-at "0123456789" (mod (get value data) u10))) (get return data)) u40))
    }
    data
  )
)
```