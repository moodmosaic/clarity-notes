# Auditing contracts

My approach to auditing contracts consists of 5 stages. 
1. Preparation - gathering basic information about contracts
2. Preliminary review - dividing code into 4 groups based on what it does and how much damage it can do if it contains bugs
3. Review Round 1 - checking the most common bugs, dev mistakes and omits + making notes about things that feels off
4. Review Round 2 - checking contracts logic, all things that caught my attention during previous stages
5. Hacking - trying to find execution path that was not planned by developers (creating exploit contract)

With a bit of experience stages 1-3 can be executed at the same time and for average size contract it takes no more than 30min.
Stage 4 can takes up to few hours, but with help of the contract author it can be completed in 1-2 hours.

Usually I don't start last stage as long as **all** bugs found during previous steps are not fixed. Sometimes fixing one small bug can completely eliminate very serious threats. Most of the time in order to fix a bug developers must change the algorithm, call chain, add/remove functions/variables/maps thus starting most time consuming stage while simple bugs are still not fixed is pointless. 


## 1. Preparation
1. Try to understand general concept of the contracts you want to audit. What it does, why and how?

2. Make a list of all (public, read-only and private) functions. Highlight those that takes traits as arguments. If that's possible make a note what each function does.

3. Search `as-contract` and check if it is used in conjunction with `contract-call?` (directly or indirectly). Write down the name of the function where you found it or make a note in a list from 2nd point.

4. Search the name of each function and make a note how many times it is called. 
While auditing project that consists of 2 or more contracts make a note how many times function is called by each contract.

5. Mark functions which are most likely harmless, and those you think might cause damage if contract contains bugs. I use numeric scales 1-3, 1-6, 1-10 (depends on the project size)

## 2. Preliminary review
1. Go through all functions one by one, starting from those which are not called at all and are most likely least harmless. 
    - functions with no side-effects and 0 calls that for example returns value stored on-chain (variable, constant, token supply, map entry, token balance, url etc.), or calculates something and then returns put into GREEN category.

    - functions with no side-effects, and 1 call, but called only by functions in GREEN category, put also into GREEN category. Repeat this process for all functions with no side-effects.

    - all remaining functions with no side-effects, put into ORANGE category

    - functions with side-effects that plays no role in anything critical ie. changing token base URI, token name, token symbol, ape name etc. put into YELLOW category

    - functions with side-effects that changes important things ie. protocol fee (if it has an upper limit that can't be exceeded), but not super critical (most likely will not lead to theft or loss of funds) - ORANGE

    - functions with side-effects that changes critical parameters - RED

    - functions with side-effects that mints, burns, transfers tokens - RED

    - if you are not sure if function is GREEN, YELLOW, ORANGE or RED - RED

2. Go through all GREEN functions and make sure they are indeed GREEN.
3. Go through all ORANGE functions and make a note if they are called by any RED function. If they are - move to RED.


At the end of this stage we should have functions split into 4 different categories:
 - **GREEN** - harmless, do not participate in anything super important, in most cases it will be just a read-only function that returns value stored on-chain
 - **YELLOW** - can change value of variable of map entry, but they are not used to anything critical. In most cases it will functions that can modify meta-data stored on chain.
 - **ORANGE** - functions without side-effects used by functions with side-effects and functions with side-effects that can alter contract behavior but not in a way that can lead to theft, funds loss or contract lock.
 - **RED** - functions we couldn't decide which category we should put them in and functions with side-effects that can lead to theft, funds loss or contract lock.



## 3. Review Round 1
1. Go through all YELLOW functions and compare who **can** call them with who **should** be able to call them. 

    Functions that **can** be called by people who **shouldn't** be able to call them must be fixed.
    > **Example:** Function that allows to change token URI can be executed successfully by anyone, but only admin should be allowed to do that.

    Functions secured using `tx-sender` value must be triple checked. If values they change aren't critical (used to secure other functions) they are OK.
    > **Example:** 
    > ```clojure
    > (define-public (change-uri (new-uri (string-ascii 255)))
    >   (begin
    >     (asserts! (is-eq tx-sender CONTRACT_OWNER) ERR_NOT_AUTHORIZED)
    >     (ok (var-set token-uri new-uri))
    >   )
    > )
    > ```

2. Go through ORANGE functions **with side-effects**.
    - Compare who **can** call them with who **should** be able to call them.
    - Check if they are secured using `contract-caller` or any other guards that prevents phishing attacks.
        > See [Securing function calls](function-calls.md)


3. Go through RED functions.
    - Check all `(as-contract (contract-call? ...))` 
        > See [Securing as-contract](as-contract.md)
    
    - Check functions that takes traits as arguments
        > See [Securing functions with traits](traits.md)
    
    - Check functions without token side-effects (burn, transfer) that can't be secured with post-conditions. They must be secured using `contract-caller` or any other guards that prevents phishing attacks.
         > See [Securing function calls](function-calls.md)

    - Functions with token side-effects (burn, transfer) can be secured using `tx-sender` but they must be examined to ensure that they do not contain any logical errors

    - If you spot anything that grabs your attention for more than a moment - **highlight it, make a note what's on your mind and move on**. You will get back to it later.
        > **Example:** You see very complex logic used to calculate amount of tokens that will be transferred, or something that depends on values from the past, or stx/ft transfer with unguarded amount, or setting var/map entry value based on argument passed by user that is not validated etc. 


## 4. Review Round 2
Everything tha caught your attention during Round 1 needs to be reviewed thoroughly.
 - complex logic - verify if all potential edge cases are covered if there is no way to manipulate the algorithm to get more tokens that we eligible

 - calculated fees are passed directly to stx/nft transfer - make sure there is no way that fee can be 0 (this is logical error that leads to TX failure)

 - using values provided by user without validation

 - you can pause contract but there is no way to resume it

 - calculations based on historical values already stored on chain that are available to anyone and can be used to trick algorithm to do things it should not do

 - guarding functions using `contract-caller` so that they must be called directly and not guarding other important functions at all

 - unnecessary guarding functions with `contract-caller`

 - allowing users to mint X NFT during "pre-mint" and checking only if they have less than X in their wallet or not

## 5. Hacking
This stage requires in-depth knowledge about how contract works. It is all about picking code apart and finding multiple teeny-tiny logical flaws and edge cases, that seems to be impossible to combine and combining them using one or more contracts, preferably exploited in a single transaction.

Having direct contact with developers who created contracts during this stage is crucial, especially when they are planning to add more code and/or new contracts to the project. 
Simple question such as :
> - Who will be deploying these contracts?
> - Why you want keep this function secured with `tx-sender`?
> - Do you plan to use multisig wallet?
> - What are your plans when XYZ happen?
> - Do you have any plans to replace X with something else?
> - How are you going to call these functions? 
> - What will happen if XYZ?

can show whether a given attack is or will be possible, and how likely it is it would succeed.

Start with executing functions multiple times in the same transaction or block, using unconventional (stupid) order execution.

Try to call function in contract A passing contract A as argument. Execute functions via proxy contract using here and there `as-contract`. 

Use your knowledge about other contracts already deployed on mainnet and mix them in your attempts.

When function needs arguments A and B, where A is `<trait-a>`, B is `<trait-b>` see what you can do if you build one contract that conforms both `<trait-a>` and `<trait-b>`.

Try to execute coordinated attack that requires usage of more than one wallet.

Be creative.
