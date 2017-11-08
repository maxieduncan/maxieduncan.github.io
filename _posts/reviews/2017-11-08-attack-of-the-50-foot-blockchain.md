---
layout: post
category : books
title: "Review: Attack of the 50 foot Blockchain"
tags : [book, review, blockchain]
---
{% include JB/setup %}

This book does a great job countering a lot of the hype around Blockchain.

### No Regulation, No Guarantees
It spends a lot of time at the start diving into the history of Bitcoin and the Ponzi schemes and fake exchanges used to rip people off. By its nature the system rewards early adopters and those joining later aren't doing much but helping to inflate the bubble and fund the early adopters. 

There are no regulations and no guarantees backing anything in any Blockchain and there are plenty of examples of nefarious characters taking full advantage of this. Exchanges are the method that most people will use to get hold of Bitcoins (or other cryptocurrencies) but they're putting a lot of trust into random entities by doing so; not surprisingly many people have been burned by exchanges folding and all the assets "disappearing".

### Slow and Inefficient
The implementation of Bitcoin using "Proof of Work"  for distributed consensus to select who gets to write a block has led to it essentially becoming centralised, with a few organisations who have optimised the mining process monopolising the majority of transactions. It's also a highly inefficient method requiring expensive hardware and somewhat ridiculous amounts of energy to run.

There are other methods but they have their own flaws; "Proof of Stake" rewards those who have a large share already, "Proof of Burn" where cryptocurrency is "burned" by sending it to a verifiably un-spendable address is so far dependent on "Proof of Work" mined cryptocurrencies as the fuel.

It's the distributed nature of Blockchain, that is touted as one of it's key benefits, that makes it slow and inefficient. For platforms that are meant to handle transactions at a global scale the leading contenders have some serious limitations; Blockchain has a theoretical maximum of 7 transactions per second while Ethereum can manage around 14. These may be fine for experimentation but they're not appropriate for real world use. Indeed at peak times the load can't be handled and transactions that can't be processed in time are eventually dropped and never added to the Blockchain.

Ethereum talks about changing to "Proof of Stake" and various methods such as sub chains and sharding to improve the number of transactions that can be processed. Progress has been slow on these changes and they're yet to be realised.

### Unstoppable Code
Smart Contracts are something that sound good in theory but that have some serious flaws in reality. There's an assumption that a lot can be coded here where in reality most contracts involve a lot of grey areas.

The immutable nature of Blockchains means that there's an assumption that developers will be writing bug free code, something that anyone involved in coding will know is not true. The quality of code found on the likes of Ethereum doesn't do anything to change this point of view. There's also a dependency on the Blockchain implementation; flaws in the Blockchain implementation, the programming language used for contracts and the Smart Contracts themselves having already been exploited to steal cryptocurrency.  With Smart Contracts being "unstoppable code" what happens when an error in the contract is identified or a bug is exploited?

### Immutable except when it's not
After The DAO was hacked due to an exploit in the application, Ethereum was hard forked to roll back the invalid transactions. This resulted in two Ethereum currencies: Ethereum and Ethereum Classic and two separate and incompatible Blockchains.

This is a solution that "worked" because a large number of stakeholders were impacted and could agree on the change. What happens when the exploit is in a small application that only affects a few users?

### Trust
Blockchains require a lot of trust with nothing to back them up. The idea is that they should make trust easier because there's a verifiable ledger of transactions but the it's open to abuse in a number of ways.

Smart contracts are still dependent on outside inputs which can easily be abused; garbage in, garbage out.

How well coded is that Smart Contract? Who's going to verify it? Even though it's code a) not many people will able to read it b) even those who can may not understand everything that's going on.

### Security
Flaws in the platform code and in Smart Contracts are open to abuse and already have been.

The distributed nature of the Blockchain means that random entities will be processing transactions. This doesn't mean that they can fake transactions but they can be selective in which ones get processed. The distributed nature does make the platform harder to hack to gain a consensus, but not impossible.

The overhead of processing transactions means that Blockchain is open to DDOS attacks, however the fact there is a cost to execute transactions mitigates the risk of this substantially.

### Money
Getting your money out isn't as easy as you might expect. Both your coins and actual currency are just numbers stored in an exchange. You have to hope that a) the exchange has actual money, b) banks will let them transfer it out, c) the transaction will actually be added to the Blockchain.

Spending your Bitcoins is problematic as well, it takes time for them to processed and as mentioned earlier there's no guarantee that they will be. Until the transaction is added to the Blockchain it essentially doesn't exist, this makes the use of cryptocurrencies as an actual currency problematic, not least because the entity that wins the right to write a block chooses the transactions that included.

### Potential
Everything about Blockchain still seems to be very experimental, there's certainly some potential but what's there doesn't seem to match the hype. There are some big issues to overcome and it doesn't seem that some of the key ones will be any time soon.

The main benefit seems to be a verifiable ledger, there's some argument for that but it's debatable if Blockchain is required.

The distributed nature of Blockchain means that everyone has a verifiable copy of the data rather than relying on a centralised source that you can supposedly trust, I can certainly see the appeal in this. The flip side is that it also dramatically increases the storage and processing requirements and has significant performance issues. Relying on consensus for determining what is added to a Blockchain is certainly more Democratic, whether it's more secure or reliable than a centralised source remains very much debatable.

It's questionable how many solutions require an open distributed ledger and even in the situations where it may be beneficial it's entirely dependent on the data being of good quality. A Blockchain will happily record data that is invalid and if a transaction isn't recorded it doesn't exist. Many of the problems that Blockchain can supposedly solve are actually caused by bad or missing data; fix the data and the other technologies will do the job just as well if not better.
