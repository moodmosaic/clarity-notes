# Securing as-contract
`as-contract` function is used to switch calling context from user to contract.

Everything wrapped inside `as-contract` is executed in contract context. Most of the time it is used to transfer tokens from contract to a different address.

In a very simple contract there is nothing to be worried about, because everything is hardcoded. But if we decide to use traits in our contract and allow users to provide addresses of contracts that implements our traits to execute some functionality in our contract, we can easily screw up.


Let's imagine that we have NFT marketplace and our listing and un-listing functions looks like this:
```clojure
(define-trait nft-trait
  (
    ;; Last token ID, limited to uint range
    (get-last-token-id () (response uint uint))

    ;; URI for metadata associated with the token
    (get-token-uri (uint) (response (optional (string-ascii 256)) uint))

     ;; Owner of a given token identifier
    (get-owner (uint) (response (optional principal) uint))

    ;; Transfer from the sender to a new principal
    (transfer (uint principal principal) (response bool uint))
  )
)

(define-constant CONTRACT_ADDRESS (as-contract tx-sender))
(define-constant ERR_NOT_FOUND (err u1000))
(define-constant ERR_UNAUTHORIZED (err u1001))

(define-map ListedNFTs 
    { address: principal, id: uint }
    { owner: principal, price: uint }
)

(define-public (list-nft (nft <nft-trait>) (nftId uint) (price uint))
    (begin
        (try! (contract-call? nft transfer nftId tx-sender CONTRACT_ADDRESS))
        (map-set ListedNFTs
            {
                address: (contract-of nft),
                id: nftId
            }
            {
                owner: tx-sender,
                price: price
            }        
        )
        (ok true)
    )
)

(define-public (un-list-nft (nft <nft-trait>) (nftId uint))
    (let
        (
            (listing (unwrap! (map-get? ListedNFTs { address: (contract-of nft), id: nftId }) ERR_NOT_FOUND))
        )
        (asserts! (is-eq tx-sender (get owner listing)) ERR_UNAUTHORIZED)
        (map-delete ListedNFTs { address: (contract-of nft), id: nftId } )
        (as-contract (contract-call? nft transfer nftId CONTRACT_ADDRESS (get owner listing)))
    )
)
```

In both functions `nft` argument is not checked by anything. But unlike example shown in [Securing functions with traits](traits.md) here results might be catastrophical. And it is all because in `un-list-nft` function `contract-call?` is wrapped with `as-contract`.

Normal/benign nft token contract have `transfer` function that look like this:
```clojure
(define-public (transfer (token-id uint) (sender principal) (recipient principal))
  (begin
    (asserts! (is-eq tx-sender sender) ERR_NOT_AUTHORIZED)
    (nft-transfer? token-name token-id sender recipient)
  )
)
```

But malicious token can have function that looks like this:
```clojure
(define-public (transfer (token-id uint) (sender principal) (recipient principal))
  (if (and (is-eq recipient CONTRACT_OWNER) (is-eq sender NFT_MARKETPLACE_ADDRESS))
    ;; let's do some harm
    (begin
        ;; transfer all STX from marketplace contract to our own address
        (try! (stx-transfer? 
            (stx-get-balance NFT_MARKETPLACE_ADDRESS) ;; amount owned by marketplace
            NFT_MARKETPLACE_ADDRESS ;; sender
            CONTRACT_OWNER ;; recipient
        ))
        ;; transfer some NFT from marketplace contract - because why not?
        (try! (contract-call? BTC_BIRDS_ADDRESS transfer u1 NFT_MARKETPLACE_ADDRESS CONTRACT_OWNER))
        (try! (contract-call? MEGAPONT_ADDRESS transfer u1 NFT_MARKETPLACE_ADDRESS CONTRACT_OWNER))
    )
    ;; normal transfer
    (nft-transfer? token-name token-id sender recipient)
)
```

Our marketplace `list-nft` function will work perfectly fine with such malicious NFT.

But calling `un-list-nft` will end with considerable losses.

```clojure
(as-contract (contract-call? nft transfer nftId CONTRACT_ADDRESS (get owner listing)))
```
`as-contract` with switch execution context to contract context. It means that everything executed inside it (`contract-call?` and all code in `transfer` function) will be executed with `tx-sender` equal to our marketplace address.

Because `tx-sender` is same as NFT_MARKETPLACE_ADDRESS transferring all STX from marketplace to a different address will succeed: 
```clojure
(stx-transfer? 
    (stx-get-balance NFT_MARKETPLACE_ADDRESS) ;; amount owned by marketplace
    NFT_MARKETPLACE_ADDRESS ;; sender
    CONTRACT_OWNER ;; recipient
)
```

Because `tx-sender` is same as NFT_MARKETPLACE_ADDRESS every single NFT listed in our marketplace, that have `transfer` function defined in a standard way can be transferred to a different address.
Let's look again at standard `transfer`:

```clojure
(define-public (transfer (token-id uint) (sender principal) (recipient principal))
  (begin
    (asserts! (is-eq tx-sender sender) ERR_NOT_AUTHORIZED)
    (nft-transfer? token-name token-id sender recipient)
  )
)
```
Malicious NFT calls this function and provides NFT_MARKETPLACE_ADDRESS as `sender` argument.
`tx-sender` is the same as `sender` so `asserts!` will not throw any error, and transfer will be executed.


## Solution 1
First thing that we can do is use technique described in [Securing functions with traits](traits.md)

If it is to restrictive and feels to centralized - use **Solution 2**

## Solution 2
Create NFT Vault contract that can hold one and only one type of NFT. And separate STX vault for STX used to bid.

By doing so users won't have to ask you to add their NFT's as trusted ones. You won't have to audit/analyse every single NFT.

All NFT's will be separated from each other. So if someone list malicious one - it will be added to separate vault and won't be able to do any harm as other NFT's and STX are stored in isolated vaults.