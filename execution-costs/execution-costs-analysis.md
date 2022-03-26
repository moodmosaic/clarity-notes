## Intro

Every transaction executed on chain comes at a cost that needs to be payed to miners for work they do to confirm them. These costs should be very familiar to anyone who ever interact with any blockchain and they are called `transaction fees`. 

But there are other costs that not many are aware of. Those are `execution costs` and each block is limited by them. In addition to block size, these are what limit the number of transactions that can be fit into a single block. They are also used to reduce resource consumption when calling read-only functions (for example called by dapp UI) and in this case they are limiting a single function call.

## Clarity execution costs
In **Clarity** execution costs are defined using 5 different categories, each with its own limit. 

```
+----------------------+--------------+-----------------+
|                      | Block Limit  | Read-Only Limit |
+----------------------+--------------+-----------------+
| Runtime              | 5000000000   | 1000000000      |
+----------------------+--------------+-----------------+
| Read count           | 7750         | 30              |
+----------------------+--------------+-----------------+
| Read length (bytes)  | 100000000    | 100000          |
+----------------------+--------------+-----------------+
| Write count          | 7750         | 0               |
+----------------------+--------------+-----------------+
| Write length (bytes) | 15000000     | 0               |
+----------------------+--------------+-----------------+
```

**Runtime costs** - limits overall complexity of the code that can be executed. For example: Negating boolean value is less complex than calculating SHA512 hash, therefore `(not false)` will consume less runtime costs than `(sha512 1)`. This category is also affected by contract size.

**Read count** - limits how many times we can reach out to a memory or chain to extract piece of information. It is affected by reading constants, variables, intermediate/temporary (let) variables, maps but also by some functions that needs to save intermediate results during execution.

**Read length (bytes)** - limits how much data we can read from memory or chain. It is also affected by contract size.

**Write count** - limits how many times we can write data into chain (writing to variables and maps).

**Write length (bytes)** - limits how much data wi can write to chain.


Read and Write limits are quite self explanatory and practically static. The more data we read/write, the closer we get to the limits.
Runtime costs are more dynamic, and more difficult to grasp as each Clarity function have its own runtime cost which for some is static, and for the others it changes based on how much data they have to process.

As a developers we have to be careful how we structure our code, which functions we use and how we use them as it is quite easy to write code that will be 100% correct, yet it will be so expensive to execute that it will eat significant portion of execution costs set for all transaction in a block. And as a result we won't be able to call it more than few times in a block.


## Analysis

### Using Clarinet to analyze execution costs.

Use `::get_costs` or `::toggle_costs` in clarinet console to gater all information.

Eventually use `clarinet test --costs` (Disclaimer with regards to `clarinet test --costs`: https://github.com/hirosystems/clarinet/issues/133)

### Setting up baseline

Add to your contract following function and make a not how expensive it is to execute it. This will be your baseline.

Whenever you add more code, or remove significant portion of the code from your contract you should re-evaluate cost of this baseline function.

```clarity
(define-read-only (a) true)
```

Having a reference point is **VERY IMPORTANT** especially when we work with big contracts, because in big contracts functions that we think should be cheap can be quite expensive only due to contract size.

### Gathering initial data

Execute separate test suite that contains only green paths or execute manually every single public function and take a note in excel how expensive it is.


### Analyzing functions that requires a lot of upfront setup

If you don't have separate test suite that can be used with `clarinet test --costs`, create external contract that you can use to set-up your tested contract very quickly.

### How to find low hanging fruits, best bang for buck and bottle necks.

Almost every code repetition adds something to execution costs. If you calculate in one function same variable 2 or more times - calculate it once and store it as internal variable. 

If you call same external function more than once - store the result in variable.

If you suspect that part of your code might be very expensive - extract it into separate public function and test.

If you are using `let` and store in local variable value that you use only once - stop doing that. Replace usage of this variable with code that calculates. Every single local variable contributes to execution costs. 

If you calculate values that can't change over time - define them as constant. Ie.
```clarity
(define-constant CONTRACT_ADDRESS (as-contract tx-sender))
```


## Optimization techniques

1. Reducing obvious code complexity (first write correct code, then work on the costs)
2. Reducing amount of data we have to read/write
3. Reducing number of times we reach for the same data on-chain (especially reading same values from maps)
4. Reorganizing code (using different functions (`if` vs `asserts!`), different algorithms)
5. Splitting/combining maps and variables
6. Combining multiple contract calls to the same contract into single call

    If you have code like this:
    ```clarity
    (let (
        (some-value (unwrap-panic! (contract-call? .contract get-value)))
        (rate (unwrap-panic! (contract-call? .contract get-rate-for-value some-value)))
    ))
    ```
    Create extra function in `.contract` so that you could make only one contract call to it like this:
    ```clarity
    (let (
        (rate (unwrap-panic! (contract-call? .contract get-rate)))
    ))
    ```
7. Inlining and extracting logic
    - replacing `fold` with nested function call if there are only few iterations
    - reducing number of internal variables
    - extracting same/similar logic to a function
8. Reducing amount of data passed between functions
    - if you don't have to pass some data - don't do that
    - passing separate values is cheaper than passing tuples
    - passing tuples in `contract-call?` is super expensive
9. Reversing logic (example: instead of counting positive values in a list, it might be cheaper to count zeros if we expect that list should consists only positive values).
10. Bulk processing instead of loops (when we iterate over list of `uint` it might be cheaper to first check if all of them are positive (see prev. point) than doing it inside fold)
11. Reducing contract size by removing comments, using shorter names (functions, variables, keys in tuples)
    - don't use stupid asci logos in comments
    - don't comment code that is self explanatory

## Long term recommendations
> Premature Optimization Is the Root of All Evil

Start with working code and focus on functionality first.

Write tests. A lot of tests. Code refactoring is difficult and tests helps you make sure that your code behaves the same way before and after change. 
If you want to reduce execution costs and you don't have proper test suite - you are doomed.

Aim for low hanging fruits first (sometimes removing comments is all you need).

Small gains accumulates very quickly - snowball effect. Sometimes reducing costs of one function by mere 0.01% can result in 2% reduction in other.

If you create huge tuple in one function that is called sporadically and then read it and use only small portion of it in another function that is called very, very often, then it is a great candidate for data structure split (splitting one map into multiple - can save a lot).

If you execute `fold` over small list (up to 10 elements), but each element of that list is a tuple and you return tuple - test if un-winded (inlined) code won't be cheaper to execute than `fold` (most of the time it will).

If you create variables inside `let` and use them once and only once - check code inlining. You have a 50% chances it will be cheaper.

Analyze `read-only` functions. If they don't fit into read only limits, and you have them only for the UI/off-chain purpose - try to expose data used by these functions and handle calculations off-chain. **Example:**

Replace this:
```clarity
(define-read-only (get-something)
    (let (
        (value1 (map-get? Something u1))
        (value2 (map-get? Something2 u10))
    )
        ;; super complex logic with fold, map, filter and bunch of ifs 
    )
)
```
with this:
```clarity
(define-read-only (get-value1)
    (map-get? Something u1)
)

(define-read-only (get-value2)
    (map-get? Something2 u10)
)
```
and do the complex logic off-chain.