# Binary search in PoXL

It is block winner duty in PoXL protocol, to check if he won and claim the reward.

Testing every single user is too expensive, but with proper data structure it is possible to build relatively cheap function that will find block winner using binary search, mark it as winner, and mint block reward. Such function could be called by anyone or any other PoXL function.

```clarity
(define-map BlockLast
  uint
  {idx: uint, high: uint}
)

(define-map Deposits
  { blockId: uint, idx: uint }
  { low: uint, high: uint }
)

(define-read-only (get-last (blockId uint))
  (default-to {idx: u0, high: u0 } (map-get? BlockLast blockId))
)

(define-public (deposit (amount uint))
  (let
    (
      (last (get-last block-height))
      (newIdx (+ (get idx last) u1))
      (lastHigh (get high last))
    )
    (map-set Deposits
      { blockId: block-height, idx: newIdx }
      { low: lastHigh, high: (+ lastHigh amount) }
    )
    (map-set BlockLast block-height
      { idx: newIdx, high: (+ lastHigh amount) }
    )
    (ok true)
  )
)

(define-constant BLIST (list 
  true true true true true true true true true true
  true true true true true true true true true true
))


(define-public (find (value uint))
  (ok 
    (fold bin-search BLIST
      {
        search: value,
        low: u0,
        high: (get idx (get-last block-height)),
        winner: u0
      }
    )
  )
)

(define-private (bin-search (i bool) (data { search: uint, low: uint, high: uint, winner: uint }))
  (if (is-eq (get winner data) u0)
    (let
      (
        (mid_val (/ (+ (get high data) (get low data)) u2))
        (mid (if (is-eq mid_val u0) u1 mid_val))
        (val (unwrap-panic (map-get? Deposits { blockId: block-height, idx: mid })))
      )
      (print mid)
      (if (< (get search data) (get low val))
        (begin
          (print "down")
          (merge data { high: (- mid u1) })
        )
        (if (>= (get search data) (get high val))
          (begin
            (print "up")
            (merge data { low: (+ mid u1) })
          )
          (begin
            (print "winner")
            (merge data { winner: mid })
          )
        )
      )
    )
    data
  )
)

(deposit u1) 
(deposit u2) 
(deposit u3)
(deposit u4)
(deposit u5)
(deposit u6)
(deposit u6)
(deposit u6)
(deposit u6)
(deposit u7)
(deposit u8)
(deposit u9)
(deposit u10)
```