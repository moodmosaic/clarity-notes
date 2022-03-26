# Securing function that uses traits

Functions that takes address of a contract that implements specific trait as an argument are very easy to exploit if there is no verification if supplied contract is safe to work with or not.

In general these functions has to be written with the assumption that the supplied contract is malicious and cannot be trusted at any point in time.

**Example:**
Let's assume we have a contract from which user can withdraw an amount of STX multiplied by a rate that is provided by some form of oracle contract. And we have multiple oracles, so we decided to use trait. Each oracle returns rate between 1 and 3. So we have code like this: 

```clarity
(define-trait oracle-trait (
    (get-rate () (response uint uint))
))

(define-constant CONTRACT_ADDRESS (as-contract tx-sender))

(define-constant ERR_FAIL (err u1000))

(define-public (withdraw (amount uint) (recipient principal) (oracle <oracle-trait>))
    (let
        (
            (rate (unwrap! (contract-call? oracle get-rate) ERR_FAIL))
        )
        (as-contract (stx-transfer? amount CONTRACT_ADDRESS recipient))
    )
)
```
All 3 arguments aren't verified by anything, but let's focus only on the last one, `oracle`.

If user address of a trusted oracle, this function will work as expected. But what if user provide address to a contract that looks like this:
```clarity
(define-public (get-rate)
    (ok u2000)
)
```

It will allow user to withdraw a lot more than it should be able.

To avoid that every single trait argument provided by user must be verified before we use it:
```clarity
(define-trait oracle-trait (
    (get-rate () (response uint uint))
))

(define-constant CONTRACT_ADDRESS (as-contract tx-sender))
(define-constant ERR_FAIL (err u1000))
(define-constant ERR_ORACLE_NOT_TRUSTED (err u1001))

(define-map TrustedOracles principal bool)

(define-public (withdraw (amount uint) (recipient principal) (oracle <oracle-trait>))
    (let
        (
            (isOracleTrusted (asserts! (default-to false (map-get? TrustedOracles (contract-of oracle))) ERR_ORACLE_NOT_TRUSTED))
            (rate (unwrap! (contract-call? oracle get-rate) ERR_FAIL))
        )
        (as-contract (stx-transfer? amount CONTRACT_ADDRESS recipient))
    )
)
```
Now if address of supplied oracle is not stored in `Trusted Oracles` map, function will return ERR_ORACLE_NOT_TRUSTED error.