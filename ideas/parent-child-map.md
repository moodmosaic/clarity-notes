```clarity
(define-constant ERR_UNKNOWN_PARENT (err u123123))

(define-data-var lastParentId uint u0)
(define-map Parents
  uint ;; id
  {
    name: (string-ascii 32),
    lastChildId: uint,
  }
)

(define-map Childs
  { parentId: uint, id: uint }
  uint ;; some value
)

(define-public (new-parent (name (string-ascii 32)))
  (let
    ((newId (+ (var-get lastParentId) u1)))
    (map-set Parents newId {name: name, lastChildId: u0})
    (var-set lastParentId newId)
    (ok newId)
  )
)

(define-read-only (get-parent (parentId uint))
  (map-get? Parents parentId)
)

(define-public (new-child (parentId uint) (value uint))
  (let
    (
      (parent (unwrap! (get-parent parentId) ERR_UNKNOWN_PARENT))
      (newId (+ (get lastChildId parent) u1))
    )
    (map-set Childs {parentId: parentId, id: newId} value)
    (map-set Parents parentId (merge parent {lastChildId: newId}))
    (ok true)
  )
)

(define-read-only (get-10-childs (parentId uint) (shift uint))
  (filter is-some?
    (list
      (get-child parentId (+ u1 shift))
      (get-child parentId (+ u2 shift))
      (get-child parentId (+ u3 shift))
      (get-child parentId (+ u4 shift))
      (get-child parentId (+ u5 shift))
      (get-child parentId (+ u6 shift))
      (get-child parentId (+ u7 shift))
      (get-child parentId (+ u8 shift))
      (get-child parentId (+ u9 shift))
      (get-child parentId (+ u10 shift))
    )
  )
)

(define-read-only (get-child (parentId uint) (childId uint))
  (map-get? Childs {parentId: parentId, id: childId})
)

(define-private (is-some? (x (optional uint)))
  (is-some x)
)
```