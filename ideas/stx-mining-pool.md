### Key concepts
* Pool owner
* Pool members
* Pool bond contract


**Pool owner** is also owner/operator of Pool bond contract.

**Pool members** are users who would like to join to pool.

**Pool bond** is a conceptual contract that ensures Pool members will get paid if Pool owner win something during mining process and they will not loose funds if he wants to scam them.
Pool bond contract have few phases:
 - STX accumulation
 - freeze/prepare
 - mining
 - mining prize distribution/STX refund


#### **STX accumulation**
In this phase Poll members puts/locks their STX into Pool bond contract. They can revoke their STX during this phase and only during this phase.

#### **Freeze/prepare**
In this phase funds are locked and stays locked untill last phase.
Also during this phase Pool owner must enter the conversion rate STX/BTC - it might be also pulled from some sort of oracle.

#### **Mining**
Funds are still locked.
Pool owner is mining using his own BTC, but he needs to make sure he'll bure more or less the same amount of BTC as the STX locked in contract.


#### **Mining prize distribution/STX refund**
Contract somehow checks based on conversion rate if specyfic amount of BTC has been spent on mining by Pool owner or not.
If Pool owner burned BTC on mining, he gets the funds from the Contract bond and if he won something, then prize is divided to all Pool members. 
If Pool owner didn't burn BTC (he didn't mine - he was dishonest), then funds from bond are automatically refunded to Pool members.


### Why this should work?

Pool members lock their funds in the Pool bond contract - their risk is limited to locking their funds for some time. If Pool owner will try to cheat - they will get an automatic refund.
If pool owner is honest - they can either "loose" STX, because miner didn't won anything (it can happen), or get a fair share (based on their contribution in bond) from the mining prize.


Pool owner burns his/her own BTC during mining - owner risks his own BTC (should be equivalent of what members put into bond). If he is honest, he'll get STX locked in bond. If he is not - he just burn his own BTC, and STX locked in bond will be refunded to Pool members.

Both parties ar protected by contract.