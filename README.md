# <img src="https://windingtree.com/images/winding-tree-logo-v2.svg"> Winding Tree Technical FAQ

# Purpose
The purpose of this document is to document and post answers to common technical questions and concepts that have arisen during the Winding Tree project and that in general relate to travel and blockchain. This is a live document and is being constantly refined and improved. For any questions that are not available on this document, [please post them here](https://windingtree.rocket.chat/channel/technical) to have them answered.

Where answers have not been received the community has made their own assumption. Please note that a lot of the content in this document has been assumed as a place to start and may not reflect the beliefs of Winding Tree themselves.

# Privacy and Identity

**Identity and anonymity – how does a person maintain enough privacy that their name isn’t publicly available but they can prove they are them?**
Current protocols that use public key / secret key are highly effective in confirming transactions without needing to affirm the identity of both parties, but that both parties come with the prerequisites to transact. However for cases where identity does need to be confirmed e.g. a customer needs to know that the payment they are making are with the entity that they are wishing to transact with there is still room for commercial solutions. As an interim solution, posting a public key on a website (or embedded in their booking form) may be sufficient for now. 
Individuals will need a user-friendly mechanism that builds on the public key / secret key retrieval process that provides an approved party with access to their personal information. The assumption is that this is not something that is being designed for as part of the Winding Tree platform and will require other 3rd party centralised or decentralised solutions to solve. A bit like how SWIFT works as part of the banking processes it expected that processes will be developed so that private key / public concept need not to be perfectly understood by everyone but function behind the scenes.

Current in-progress solutions for identity on the blockchain and individual identity governance include Sovrin, uPort, Civic, æternity, Sphere Identity.

**What is the process of verifying that the hotel you’ve booked is who they say they are?**
This will mostly operate in the same way as it does now. Reviews, websites and contact details attached to a public identifier which has historical transactions against will create credibility and reference material for suppliers. Applications using Winding Tree will also need to augment the information here and be able to caution customers when a supplier is new and hasn’t built up the prerequisite history yet.

The Ethereum Name System (ENS) also allows to bind an DNS domain and an ethereum address using DNSSEC (see technical details here). So a booking party could verify that the supplier ethereum address is legit, assuming there is a way from the DNS name to deduce the actual property (which might not be the case for small property owners without a dedicated website).

The historical transactions is an indicator of the legimity of an hotel, but is not sufficient since a hacker could book his own rooms and rotate the LIF.

Also there is a need to identify duplicates in case the same property is listed on different contracts. This could be done using GPS coordinates for instance.

It may also be useful for a hotel contract to point to a JSON file at an https URL at the hotel website. There could be references/hashes to check that the JSON file, website and hotel contract are all related. The JSON file would have the hotel's info about the hotel, e.g. room descriptions, image URLs, etc.

From https://blog.windingtree.com/the-three-inquirers-9ef3a5b5d57d 

*"Easily possible fake hotel scam” is actually impossible. This is not a 					problem that we need to solve since it’s been solved many times by the industry. AirBnB solved it, Expedia solved it, etc. How? First, reputation system. Second, if there is a new hotel with no reviews, the payment won’t’ be sent to them until there are 10–100–1000 positive reviews of the hotel/airline/etc.”*

**How does a supplier know who the customer is? Everything seems to be an “address” in Ethereum (and presumably some of those are cryptographically protected with private keys), but I don’t see links with the “real world”, unless suppliers and travel agents are maintaining their own lists of “addresses” of known other parties, or potentially existing domains and HTTPS can be used somehow in conjunction. Often suppliers will want (and may legally be required) to know who is staying/travelling with them. There may be loyalty programs involved here too.**

Winding Tree is mainly aiming at B2B transactions (although B2C is possible too), so the booker is not necessarily the traveller. Should an authentication be needed, an authentication with ENS could be used.

In all cases the regular booking data needs to be passed from the booking party to the hotel party; the mechanism of which is not defined so far and involves data privacy protection. One possible implementation would be to expose an REST API endpoint in the hotel smart contract (eg: api.myhotel.com) and defines out of the blockchain world a common API for any hotel in order to pass the booking data (Customer Name, Address, Phone and potentially even Credit Card number for post-paid scenarios)

**How will privacy be maintained on a public blockchain?**
Quote from WT Team:

*"Storage, scalability and privacy are currently the main concerns for the WT that are our main concerns, alongside with privacy. For now our MVP (only for the hotel industry) will have everything on chain buts that's not how we planning to do it. We want to store the less data as possible on chain, the metadata only necessary to index and connect contracts with each other, the rest of the information we plan to host it in Swarm or IPFS."*

However SWARM/IPFS being both decentralized file transfer systems, a bad actor could easily store the booking data and use it. Even if the data is stored encrypted, it made be tedious since an encryption flaw might be found several years later and the data breached.

A better mechanism is to transfer directly the booking data from the booker and the supplier. This could be handled in many ways but in all cases the booking data should never be stored on the Ethereum blockchain. Some possible mechanisms:
- Agree on a “Rendez-vous” using state channel, then transact with other technology (eg: HTTPS REST, mail, phone call)
- [TBC with WT team if that’s the intent] Use the “TransferData” feature of ERC827 extension over state channels.

Also separately from the traveller privacy protection, there might be a need to protect the potential private business agreements between the two parties (eg: negotiated rates)

# Technical Requirements

## Accessing the WT Smart contracts

Both the booker and the supplier will need to access at least one ethereum node in order to process transactions. There are public ethereum nodes but likely for intense professional usage one would consider having a dedicated full ethereum node. Also the syncing could be key to access low availabilities (such as special discounts).
Communication from the backend to the ethereum node can be using JSON RPC or IPC. 

Should the node be public, the caller would need to sign the messages with its private key before calling the node so that he gets debited of the appropriate ETH/LIF amounts. This means from that the caller needs to maintain a key infrastructure to avoid unauthorized access to the key.
Should the node be private, the key could be stored on the node and therefore security measures should be in place on the node itself (actually in this case it is not recommended to use the JSON RPC but rather IPC to benefit from local server security)

In order to ease this situation, there might be some dedicated companies offering a more classic REST API service wrapped around the smart contracts. Such services could have their own billing model, it can not be 100% free (unless sponsored) due to the gas fee that they will need to pay to the network to write on the blockchain.

## Data Stored on WT smart contracts

Currently on the MVP proposal for Hotel, the smart contract stores the following data:

 - Hotel Description
	 - Name
	 - Description
	 - Physical Address
	 - GPS Coordinates
	 - Link to pictures
 - Room Descriptions
	 - Amenities
	 - Default Price
	 - Number of guests
	 - Link to pictures
	 - Number of rooms
 - Room Inventory
	 - Availability
	 - Special prices

All the static data is a good candidate to be moved out of the blockchain in order to reduce the cost of creating/updating the hotels. One possibility would be to store it in SWARM/IPFS/CDN using a JSON data structure enforced with a schema.

The inventory (availability/rates) is handled using state channels. On-chain is not a good idea for various reasons:

 - Scaling: maximum transaction rate on the Ethereum blockchain is estimated to 20tps. With state channel the maximum rate is linear to the number of nodes.
 - Cost: transactions off-chain do not need to pay for the gas fee, only the last one to commit the final state.
 - Consistency: transactions can take some times until they get mined by the ethereum miners and there is a risk to have the transaction rejected as uncle (when two different nodes are not in sync). The difficult scenario here is the “Last Room availability”.

**Will there be a need to have up to date copies of the blockchain on the server or will accessing the Winding Tree APIS be sufficient? Just trying to understand what sort of power consumption and requirements exist. Your roadmap also has search coming late in the piece which is a core feature as we see it so thinking through options to bring this to life earlier.**
The idea is to store everything on chain at least on a decentralized storage, we will provide the tool to rebuild a database with the same information stored on chain but indexed in a central location to provide faster queries, this can also be use as a backup database.

**It makes sense how this “living state data” in the smart contract is maintained – just need a bit more detail about how the actual purchase transactions work.**
For now all the availability of an inventory unit is stored on chain, that means that if I want to to book the unit 0x111 I will need to send a transaction with Lif or the information of my FIAT payment encrypted in that transaction to the address 0x111, once the hotelier receives the transaction he can review that the payment was done and accept/reject the booking, there is also the case where the hotelier can accept instant booking with Lif, since the verification of the payment can be done on chain using Lif or ETH as payment.

**What are the expectations of what will be stored in a smart contract? Price, image url, details etc?**
For now everything, but you can expect to have the static data stored in a decentralized storage service in a near future.

**What are the limitations (block size vs performance)**

**How long does a typical pricing query take? What are their estimates in terms of querying for a hotel, vs number of hotels in the chain, etc?**

**What’s the min spec for a blockchain node to be useful in terms of a MVP?**

**Is there a cloud offering in terms of a WT node, blockchain client, etc? Or is it assumed that all clients will connect to infura?**

**MIT license - why that choice?**

**Would WT consider themselves as standards organization, e.g. publishing the new versions of smart contract specifications, but no actual implementation - that is left to participating members?**

**What about loyalty programs, spend half in points and half in cash, etc?**

**What about baggage data, traveller metadata (i.e. PNR records, meal data, other special requests, etc)?**

**What about banned travellers?**

**What about checking/verifying visa restrictions/etc?**

**How are fare rules stored and also verified (i.e. is this at time of booking)?**

**What about no-shows, flight cancellations, refunds, schedule changes, etc?**

**Bookings could be expensive in terms of gas used, but bookings can also fail. Who pays for that (the customer is going to be annoyed at having to pay something for nothing)**

**Please explain the DAO voting process in more detail in terms of approving new versions of smart contracts. Can this end up with larger holders of Lif pushing their contracts ahead of others?**

**How do you discover new WT-based smart contracts?**

**Who will partner with WT to produce API and implementation? Is this a closed door in terms of Lufthansa and Air NZ?**

**Maximum guaranteed wait time for bookings, max guaranteed time for searching?**

**Interlining/code sharing?**

**Taxes?**

**Revenue accounting**

**NDC**

**Side payment, e.g. with credit card**

**Zero Lif no shows**

**Cancellation policies**

**Fake hotel listings**

**Bankruptcy of a party**

**How much dev work do you have planned, and do you have any timelines for this?**

**Contracts need to be optimal to not cost users too much money if the booking fails**

# Code / Github 
## Common Code
**bookWithLif calculates the cost in Lif currency and seems to use built-in Ethereum transactions (?) to transfer from the “address” (a person or another contract??) booking the room to the contract. Not quite sure how this is called – since it says it’s only called from itself (part of the AsyncCall framework??) – and also whether there needs to be prior approval of Lif from the customer/travel agent to the hotel before the transferFrom of the Lif can succeed.**

An actual implementation can be checked at http://demo.windingtree.com. It is running on the Ropsten test network.

**Are the Winding Tree APIs a specification or an actual implementation? If the latter, is that implementation actually hosted somewhere (how is this paid for?), or do companies (like travel agents) get the (open?) source and host themselves?**

## Payment
**Topics to clarify:**
- Zero Lif booking: how to agree on the mechanism to pay?
- Fraud Management
- Bankruptcy of a supplier
- Price volatility between booking and service delivery
## Hotels
**Topics to clarify:**
- Pre-paid vs Post-Paid
- Cancellation policy
- Fake hotel listing
- Duplicates
- Traveller Reviews
## Airlines
**Topics to clarify:**
- Price computation (in particular with all sort of taxes and fares), alignment with ATPCO
- Interlining
- How is the interface: WT Specific? NDC?
- How does it fit in the complex IT for airlines?
- Follow-up actions like Refund / Void / Exchange
- Flight Schedule changes

**Need to understand a bit more around pricing – e.g. return versus one-way (actually something similar applies to stay/pay for hotels) – where there are more complex pricing rules depending on overall inventory use.**

**How do upsell/changes/cancellation, and seat/bag/meal requests work?**
## Ground Transportation
## Airport Services
## Activities
## Packaging

**Would the plan be to have the latest blockchain data locally (so would have the latest inventory levels for hotels/flights) and then search through that (or some index built from that) in response to an availability request from a customer? Would this essentially involve running a (non-mining?) Ethereum node locally? Would this get all the global Ethereum data from everyone for everything? Or is there some way to just get a subset? Or would the plan be to query some other Ethereum node for all the info? That could be a lot of calls if there are a lot of potential hotels, or if we might need to knit together a number of flights for an overall trip.**
It is assumed this is Winding Tree’s thinking. Every machine that participates takes the full public Ethereum blockchain, and for read-only queries, this is for free as the smart contract in this case would only run locally, giving results based on the current state of your local blockchain. Mining on the other hand I think is a completely separate concern, i.e. all mining is done by public miners, which don't really care if WT contracts are being executed, and don't care about LIF either, i.e. they get paid in ETH for their mining reward. The trick is to make sure the smart contracts have enough ETH for gas consumption, otherwise those contracts won't be executed.
# Public vs Private Blockchains
**Will Winding Tree be making exclusive use of public blockchain technology or use private or hybrid blockchains as part of the broader ecosystem?**
Our assumption is that the most viable scenario for a travel transaction platform is one that is public. One could argue that a GDS already fits with a private blockchain insofar as it resembles a closed, centralised database. It is not until the system itself is “unowned” and decentralised that middleman fees (fees that can be charged for owning the system as opposed to mining or blockchain authentication fees which will be discussed separately) can be replaced with transaction fees (those required to run and secure the network).

Public blockchains facilitate 2 core benefits:
1) Engender trust via the fact that no one individual can exert power or force changes that are not for the "greater good" or extract rent.
2) Network effect. Their open nature enables and encourages broad-based participation.

These 2 core benefits are driven from the freedom, neutrality and openness that are inherent in a public blockchain. No one entity can own it.

Private blockchains also offer benefits including the ability to modify/change the rules of the blockchain to make fixes, low cost transactions (due to a significant reduction in required nodes for verification), and greater privacy. 

Whilst both present strong opportunities for dramatic improvements in transaction cost and time we are of the strong view that Public Blockchains provide a substantial paradigm shift where industry participants can govern and forge their own future path ensuring a fairer operating environment for all participants. 
# Data Ownership
**Who will own the data on the Winding Tree Blockchain?**
In the context of data ownership and [GDPR](https://www.eugdpr.org/) there is a big shift towards consumer rights to own their own personal data. On the flip side, a) the ability to disintermediate various business models thereby reducing cost will lead to a general acceptance of publicly accessible transactional data (financial transactions e.g. accounting, retail sales, travel). The implications of having inventory on a public ledger are that all transactions are open to anyone with the capability to use a blockchain explorer (so assume everyone). We can make the assumption that data is public anyway. Taking the OTA/Metasearch example; competitors have visibility of each others pricing and can predict availability based on pricing. Screen-scraping is common practice as is monitoring of competitor market share and performance. All of which come at a cost in terms of dollars and manpower. 

Private information specific to one’s identity e.g. passport number etc is to be kept separately off the public chain. The mechanism for this is still unconfirmed. Unlike centralised systems, which currently equates to most large online enterprises (Apple, Google, Uber, Expedia etc) where the assumption is that data stored on their systems are safe and protected however the reality is very different, ownership of personal information is expected to reside with the individual and not the corporate. 2 possible scenarios for how this plays out are 

1. Corporate consumers of personal information e.g. expedia are given the right to use a customer's passport number until such time as the transaction is complete, and then it must be expelled from their system. None of it will be stored on public blockchain.

2. Individuals will be given a unique address on the respective blockchain that manages their personal identity, in the same way that primary keys are used in database schemas. A request / approval mechanism would be required to unlock and share with right to retain information prohibited.

# Search
**What is the mechanism for discovering all the active hotel smart contracts? Will it be possible to examine the data in the blockchain to see all instances?**

All Hotels are supposed to register in the WTIndex Contract. Hotels contract are not deployed directly by the hotel manager but instead they are created by the WTIndex contract.

To be noted that it does not prevent an hotel to deploy their own contracts as the code is open source, but they would not be indexed by WT.

Below is a proposal flow ([reviewed by Jakub](https://github.com/windingtree/wt-contracts/pull/158)) for the hotel registration **according to the current MVP**

![enter image description here](https://www.websequencediagrams.com/cgi-bin/cdraw?lz=dGl0bGUgSG90ZWwgQ3JlYXRpb24gRmxvdwoKcGFydGljaXBhbnQAGwZpZXIgYXMgaHRsAA8NIldUSW5kZXggU21hcnRDb250cmFjdCIgYXMgd3QAHA4AYgYAGw0ocykAJgVoAEYOVW5pdFR5cGUAFxZ1AEkPVW5pdAATFwoKbG9vcCBGb3IgZWFjaACBZQYKbm90ZSBvdmVyIGh0bCx1dCx1LGgsd3Q6IDEuAIF-DwoKaHRsLT53dDogcmVnaXN0ZXIAgikFKG5hbWUsIGRlc2NyaXB0aW9uKQphY3RpdmF0ZSB3dAp3dC0-aDoAgk4GAIJaBwAhEiwgbWFuYWdlcgAxC2gKaC0AagYKZGUACwt3dC0tPmh0bAB-ClR4ABYMd3QAgSALY2FsbACBIQZpbmRleCwgW2VkaXRBZGRyZXNzQ2FsbERhdGFdAIEdFQAeCyhsaW5lT25lLCBsaW5lVHdvLCB6aXAsIGNvdW50cnkAgSEUAIEcGGNhbGwAgH8xTG9jAIUMBQCBDyIAIggodGltZXpvAIEuBWF0aXR1ZGUsIGxvbmcABgUpAHBCAIRECUUAhEQJIFBpY3R1cmUAgmMbYWRkSW1hZ2UAglwaABoIKGltYWdlVVJMAIIcQWVuZCBsb29wCgCBJRAAhigFAIZXBWluIHRoaXMgaACFdh4yLgApCwCGCQ91dACFWwkAhysIKABKBQCIAAgAhG0HLCB1AIdLB05hbWUAhiALdXQKdQCFUQkAh20JAIg-CCBhAIUtBgCFfwx1AIVLHmFkZACINQgAglIdAIEmCQCBEQgAgSMPAIUBXWNhbGwAgRoMAIcaBQCHDwUAhn8WACQMAIEoCU4AiH0FADIIAIg0DwCDFQVlZGl0KACIYg5pbkd1ZXN0cywgbWF4AAMIcHJpYwCDABUAh0MNdXQAhzA2AIYmEEFtZW5pdHkAgXMtYWRkADAHAIFwNgAyDgCCGBQAWAooYQCBGwYAgTlYAIcQCgCHBxIAjW8FAIgvIgCCDBUAiFkKAIFrNACJGAkAggYXAIkGGQCBRVcAjykPdW5pdACJCSgzAIkqBwCPOwkAiSUGAIkdDQCIdy8KdQCJGAwAiQIeAIh7JACIeiEAiRIFAIh1HgCKeAVpbmNyZWFzZVVuaXRzKACGa1kAiSkjAIlDBXNldEN1cnJlbmN5Q29kAIRqJACKewUAjB0JAC0TAIlIEToAFhAoYwBoCwCDJBIAiTcOAIEfYURlZmF1bHRQcmljAIFLNQAwEACBahYAWgwoAIsvEQB9fExpZgCBPEAANwwAgWIdAGQIKGwAbgcAgyhWAJJDCgCSSwsK&s=rose)

The indexing in the MVP is quite simple ([just one array with the list](https://github.com/windingtree/wt-contracts/blob/master/contracts/WTIndex.sol#L116)), but this could be improved to have it indexed in a tree-map for example by Country. Also each travel agency could build their own optimized indexing depending on their business needs.

Even if the above is possible it may be better if hotels are required to register their smart contract instances, maybe in another, Winding Tree defined, smart contract which can be queried for all active hotel contract addresses?
# Líf Token 
**The scenario for a Travel Agent is laid out in the white paper. For direct hotel bookings had you envisaged this working via a similar process?**

**Does this mean consumers will require access to lif?**
Not necessarily as the project is focused on B2B. The cool feature is that anyone with LIF can be a travel agency.

**How will travel companies / consumers get lif?**
Initially Lif will be available on a number of public exchanges. Longer term there is a need for a consumer-friendly purchase and storage mechanism. Initial adopters will mostly be travel agents. Decentralized exchanges like KyberNetwork/EtherDelta can also allow to quickly convert ETH to LIF.

**Can a booking be made with Zero lif and settled via an alternative method other than lif e.g. credit card?**
The answer to this is yes, zero lif contracts are allowed. The challenge comes in confirming that buyers are committed to using the booking given they haven’t paid anything for it. In this case the hotel may be comfortable accept post-stay payment however they will also need to capture enough information from the customer that they are satisfied the customer will use their booking.

**Is it possible to ensure lif remains cost competitive in a high appreciation scenario so that cost doesn’t become an issue as we’re currently seeing with the GDS for example.**

**Are lif fees expected to be consistent or pegged to similar mining concepts like bitcoin that fluctuate. We saw a scenario where crypto kitty users had to increase their fees from 0.01 ETH to 0.02 ETH and this sort of thing could have a large impact on travel companies if they are all competing to get their transactions processed.**

**Obviously there is a big opportunity to remove the problems currently faced with FX fluctuations and exposure and remove the need to depend on IATA rates but if lif is not consistent in terms of how it is priced e.g. pegged to a currency, this may also cause issues.**

**Can users transact in Fiat and not Lif?**
## Cost of Underlying Tokens
It is assumed that the supply and demand of the underlying tokens (ether and lif) will dictate the market rate for these tokens and that regardless of these prices the amount required to transact and pay fees will increase / decrease in relation to its current value. E.g. 

- Lif last year cost $10. Fees at this price normalised to 0.005 lif or 5c per transaction
- Lif this year costs $100. Fees maintain their fiat cost of 5c which now equates to 0.0005 lif per transaction

*The above example assumes that network conditions stay the same and is an example regarding the indifference of token cost to fees only.
## Pricing and Fees

**How do pricing and preferred rates work?**

**Does the appreciation of ETH impact the Lif price?**
ETH will be used to peg to Lif during the ICO but from there the price of Lif will move based on normal market variables.

**Do you have any solutions for token volatility? Obviously if a customer pays X fiat, that is converted to Y token e.g. lif and a refund or cancellation takes place there may be a conversion discrepancy**
For hotels, an initial option could be for Lif to be used between travel agents and hotels, to pay a form of deposit until the guest's actual credit card (of other form of payment) is known by the hotel. Once the guest's info is confirmed (or maybe after their stay), the Lif is refunded to the travel agent. There needs to be some charge (above the ethereum gas cost) for reserving a room to prevent a hotel easily being booked out with no eventual payment which would be paid by the travel agent and passed on in fiat ot the customer.

**Does pushing a smart contract with a rate into the marketplace imply general access to that smart contract or will there be limitations set on who can access which contracts?**
We imagine a supplier issues as many contracts at a specific rate as they wish and set the amount of inventory available. Those with access to those rates and inventory can then book them. Still not clear on the feature that facilitates the access between buyer and seller ie how does the buyer know they have access to certain product?

**How will transaction fees be set?**
Currently it is unclear as to how fees will be set. If it is true free marketing economics and miners are incentivised to confirm the transactions willing to pay the highest fees first, network efficiency will likely have a big impact on fees.

As at 21/12/2017 
- a transaction on ethereum costs 21,000 gas for confirmation time of less than 2 minutes (considered fast) with an ETH / gas ratio of 40 gwei / gas. At ETH / USD of $824 this equates to USD $0.69 per transaction.
- Creating a contract is 32,000 gas or $1.05
- Saving a 256-bit hash to storage is $0.66
- Off-chain storage cost e.g. [Swarm](http://swarm-gateways.net/bzz:/theswarm.eth/#) is unknown due to this still being in POC / alpha phase.

**Hotel supplier scenario**

- Holiday Inn Times Square want to make their “Standard Room” inventory from 1 Jan 2018 through to 31 March 2018 available on Winding Tree. 
- The market rate is $100 per room per night, except for specific blackout dates.
- Holiday Inn creates a contract on the Winding Tree blockchain (32,000 gas / $1.05)
- Inventory details are inserted into the contract (name, price, blackout dates, url to images, room type, amenities, room details) at a cost of ($0.66 assuming less than 256 bits).
- Large objects such as images will be stored in Swarm (pricing to be confirmed)
- Every interaction with the contract uses state channels which scales lif transactions in the same way that lightning adds multiple transactions into blockchain blocks, thereby providing scale and reducing cost.

**Travel Agent + Customer scenario:** Joe wants to book a 1 night stay in New York on Jan 31 2018 via his travel agent, Select Holidays

- Assume the current cost of Ether is $900. 
- The current cost of lif pegged to ether at 900:1 is therefore $1.
- Joe decides to stay at the Holiday Inn via his Travel Agent at Select Holidays.
- The TA confirms the booking by issuing payment through Winding Tree’s contract for holiday Inn
	- Holiday Inn receives 100 lif which can be converted to $100 fiat.
	- Joe pays the travel agent $100 + their service fee for making the booking.
	- The travel agent pays $100.66 (room cost + transaction fee) which is converted to 100.66 lif
	- The mining pool receives 1.05 lif (contract creation from supplier) + 0.66 lif (contract detail insertion fee from supplier) + 0.66 lif (transaction fee from customer) = 2.37 lif converted to 0.002633 ETH in fees

*It is assumed that in order to create an auditable record of the transaction the contract must at minimum include the public identifier of the hotel (with history related to any changes that occur against it), the inventory being confirmed, number of nights and public identifier of the customer (Joe).

The gas fee is expecting to go down with the move from Proof-Of-Work to [Proof-of-Stake](https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQ) and State Channels.
# Ethereum
**In terms of the ethereum roadmap, specifically around scalability and some of the proposals (sharding, plasma etc) and issues things like digitally birthing cats had on the network, how tied to ethereum is Winding Tree and could you technically redeploy on another blockchain if you needed to?** 
# Scalability
**Given the challenges with scaling on Ethereum (or any blockchain using Proof of Work), how do you expect to not fall victim to the impact of poor scalability (rising fees, delays to transaction confirmation etc)?** 

Scalability is maybe the most important problem to solve, we need to scale no matter want this to work, for this we have been researching about state channels and building another layer over the ETH network only for Lif holders where the transactions per second will be much higher.  

There are also a number of Ethereum Foundation sponsored initiatives that are currently underway.  These include Sharding, Plasma, and Raiden, all of which are under various stages of development.
# Functional 
**Taxes – how complex is the calculation of these? Is this a) part of the data stored in Ethereum, b) something the Winding Tree APIs will help with, or c) something to be worked out by travel agents?** 

**How do cancellations work?** 

**For hotels, not clear if there is anything there for purchasing extras (before or at end of stay)?** 

# Governance

**How to propose updates to a WT Contract code?** 
There is a DAO owner of the WTIndex. But not details has been provided how it would work. 
The code is on github so anyone can propose a change.

**How to ban a scam contract?** 
Probably with the DAO ?

**How to resolve a dispute between an agency and a supplier?** 
None?

# Legal / Accounting
**Is there any regulation to comply with from accounting perspective to use or hold LIF Token?** 

**How to comply with KYC/AML/Anti-Bribery/Anti-terrorism financing ?** 

