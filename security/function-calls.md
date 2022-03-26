# Securing function calls
Some contract functions should be callable only by a specific addresses.
There are two keywords we can use for that purpose: `tx-sender` and `contract-caller`


## Contract owner guard example
Basic primitives:

```clarity
;; we have to store who is contract owner and to do so we assign tx-sender to constant
(define-constant CONTRACT_OWNER tx-sender)

;; reusable error
(define-constant ERR_UNAUTHORIZED (err u401))

(define-private (isSenderTheOwner)
    (is-eq tx-sender CONTRACT_OWNER)
)

(define-private (isCallerTheOwner)
    (is-eq tx-sender CONTRACT_OWNER)
)
```

All functions with tokens side-effects (minting, transferring, burning STX/FT/NFT) can and should be guarded with `isSenderTheOwner` that uses `tx-sender`.
Transactions submitted in `deny-mode` in which we call functions with tokens side-effects are guarded by transaction post-conditions.

**Example:**
```clarity
(define-constant CONTRACT_ADDRESS (as-contract tx-sender))
(define-data-var accumulatedFees uint)

(define-public (withdraw-contract-fees (recipient principal))
    (begin
        (asserts! (isSenderTheOwner) ERR_UNAUTHORIZED)
        (as-contract (stx-transfer? (var-get accumulatedFees) CONTRACT_ADDRESS recipient))
    )
)
```

All functions without tokens side-effect, that can change important values in contract should be guarded with `isCallerTheOwner` that uses `contract-caller`.
Transactions submitted in `deny-mode` in which we call functions without any token side-effects are not guarded by anything and `tx-sender` is susceptible to **force contract-call** attacks.


## Extra solution
Securing functions with `contract-caller` makes them more rigid and eliminates possibility to call functions via proxy-contract (useful if we want to enclose multiple function calls in a single transaction).
If you don't want to gave up from flexibility offered by `tx-sender` and retain high security at the same time just add 1 micro STX transfer to every single function that you want to secure.

1 micro STX is worth next to nothing, but it will force clarity to use post-conditions as your security layer. 

___
https://github.com/stacks-network/stacks-blockchain/issues/2921