Instead of storing who owns particular NFT in huge lists it is much more efficient to store such information in a map.

The best option would be not use external block-chain indexer for this job, but this is something for another day.

```clarity
(define-constant ERR_UNKNOWN_NFT (err u123123))

(define-map NFTLastIdx
  principal ;; who
  uint  ;; lastIdx
)

(define-map NFTIdx
  uint ;; nftId
  uint ;; idx
)

(define-map NFT
  {who: principal, idx: uint}
  uint ;; nftId
)

(define-read-only (get-5-nfts (who principal) (shift uint))
  (list
    (get-nft who (+ u1 shift))
    (get-nft who (+ u2 shift))
    (get-nft who (+ u3 shift))
    (get-nft who (+ u4 shift))
    (get-nft who (+ u5 shift))
  )
)

(define-read-only (get-nft (who principal) (idx uint))
  (map-get? NFT {who: who, idx: idx})
)

(define-read-only (get-last-idx (who principal))
  (default-to u0 (map-get? NFTLastIdx who))
)

(define-public (mint)
  (let
    (
      (newIdx (+ (get-last-idx tx-sender) u1))
      (nftId (generate-id)) ;; calculate somehow new nft id
    )
    ;; mint nft
    (map-insert NFT {who: tx-sender, idx: newIdx} nftId)
    (map-insert NFTIdx nftId newIdx)
    (map-set NFTLastIdx tx-sender newIdx)
    (ok true)
  )
)

(define-public (transfer (nftId uint) (sender principal) (recipient principal))
  (let
    (
      (senderLastIdx (get-last-idx sender))
      (recipientNewIdx (+ (get-last-idx recipient) u1))
      (nftIdx (unwrap! (map-get? NFTIdx nftId) ERR_UNKNOWN_NFT))
    )
    ;; recipient
    (map-set NFT {who: recipient, idx: recipientNewIdx} nftId)
    (map-set NFTIdx nftId recipientNewIdx)
    (map-set NFTLastIdx recipient recipientNewIdx)

    ;; sender
    (if (> senderLastIdx nftIdx)
      (map-set NFT {who: sender, idx: nftIdx} (unwrap! (map-get? NFT {who: sender, idx: senderLastIdx}) ERR_UNKNOWN_NFT))
      true
    )
    (map-delete NFT {who: sender, idx: senderLastIdx})
    (map-set NFTLastIdx sender (- senderLastIdx u1))
    (ok true)
  )
)


(define-data-var lastId uint u0)

(define-private (generate-id)
  (let
    ((newId (+ (var-get lastId) u1)))
    (var-set lastId newId)
    newId
  )
)
```