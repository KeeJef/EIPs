# Service Node Checkpointing

```
Metadata 
LIP Number: 2 
Title: Block validation by Service Nodes [Foundation Vote] 
Author: Kee Jefferys, Jason Rhinelander
Status: Draft
Type: (Core)
Created: date (2018-11-30) 
```

## 1 Simple summary 

Consensus via Proof of Work in Loki faces the significant risk of 51% attack; to mitigate this risk and increase security Loki should integrate Service Nodes more closely into the Consensus mechanism. This proposal if passed would allow Service Nodes to validate the existing PoW chain to prevent reorganisations of the blockchain greater than N blocks.

## 2 Abstract

Loki currently uses only Proof of Work to come to consensus on the state of the blockchain. Although the introduction of Service Nodes has removed a portion of the block reward from miners, Service Nodes do not create, verify or submit blocks to the network. Proof of Work is a well studied system for forming decentralised consensus but it presents a number of issues especially for coins with lower market capitalisations relatively speaking. These issues include but are not limited to: Roaming hashrate attacks, Monopooling, and rented hashrate attacks. This proposal presents novel attacks on Loki’s Proof of Work consensus system and proposes Service Node block validation to prevent this.

## 3 Motivation

Proof of work’s intrinsic value in a blockchain system is derived from its ability to fairly choose who can update the blockchain. Each block requires proof that the block creator has spent some resource that is external to the blockchain itself. Although Proof of Work does well to satisfy the above claims it also has some major drawbacks.

The most pressing for Loki is that of a 51% attack which would seek to perform a double spend by producing a chain with more proof of work than a canonical chain. Typically when referring to 51% attacks they are strictly theoretical, either requiring unreasonable amounts of money or large amounts of economically disincentivised social collusion. However there are a number of factors which make lower market cap coins like Loki particularly susceptible to 51% attacks. 

First being the Roaming hashrate phenomenon which refers to the propensity for miners to quickly move hashrate across networks seeking the most profitable coins, this can create large fluctuations in hashrate which are difficult to control for.

Second being Rented hash power attacks, these are relatively new phenomena which have seen wide proliferation through services like NiceHash and MiningRigRentals these services allow users to temporarily rent hashing power for a number of popular hashing algorithms . Rented hash attacks become a significant threat when the hashrate available on these services exceeds the hashrate of the minable coin. 

Finally Monopooling, which is a complex problem where a large number of miners tend to mine only to the largest pool regardless of the fee structure of that pool. This is often a symptom of the Roaming hashrate phoneomenon. If a monopool reaches more than 51% of the network hashrate then it becomes a risk factor for general 51% attacks occurring. 

### 3.1 PoW Disadvantages

#### 3.1.1 Roaming Hashrate

To understand this phenomenon it is best to imagine a portion of the “global GPU hashrate” as liquid, this hashrate moves between coins based on their short term profitability. This means coins which have a spike in price or a large reduction in difficulty are often mined for short periods of time by these miners.

The Roaming hashrate phenomenon has increased in scale with the addition of websites which allow Miners to easily analyse the profitability of every minable coin live depending on their hardware, tools like this can make recommendations about the most profitable coin to mine   . Additionally developers have created open source utilities which allow hashrate to automatically switch between different coins depending on profitability, this can occur entirely without user interaction.
Figure 1: CryptonightProfitSwitcher CLI utility that automatically targets mining rigs to most profitable CryptoNight coin. 

Although these miners are not intending to negatively impact coins, they can further exacerbate deep cycles of hashrate fluctuation when they switch between networks. These deep cycles can be exploited by attackers, either by controlling pools that these miners automatically switch onto, or using rented hashing power attacks when profit seeking miners switch off creating a temporary drop in the network hashrate. If done programmatically, Roaming hashrate can also contribute to the Monopooling phenomenon.

#### 3.1.2 Rented Hashrate Attacks 

Although the phenomenon of Roaming hashrate would generally be considered as benign and lacking malicious intent, rented hashrate attacks often carry a much greater negative stigma. There have been numerous examples of attackers using rented hashrate to perform a wide variety of attacks, including timestamp forging, difficulty jamming and double spend attacks  .


Figure 2 : example of a hashrate attack causing difficulty jamming on the Karbowanec network 

The proliferation of services like NiceHash and MiningRigRentals make the renting of hashrate easy to even the most basic of attacker. Although the same 51% attacks have always been possible, attackers previously would either have to permanently purchase hardware totaling 51% of the networks hashrate, or convince miners to use their malicious pool. These options typically involved large setup costs in hardware, time, or social efforts (and often all three).

Miners can now spontaneously set up a private pool with modified malicious software, then rent the needed hashrate to perform a double spend with relative ease. Although this specific issue has not directly affected Loki, there have been notable attacks on a number of Cryptonote coins. This attack becomes easier as the hashing algorithm becomes more available and liquid on NiceHash relative to a coin network’s actual hashrate. 

This comes as a particular concern for Loki as we have repeatedly lowered rewards for miners which has reduced the network hashrate and led to a significant amount of hashrate moving to more profitable coins. Any plans to increase rewards for Service Nodes further may continue this effect, further reducing the Loki hashrate. This trend is illustrated in figure 3, which shows a trend of growing convergence between Nicehash available hashing power and Loki’s global hashrate.


Figure 3: Available NiceHash CryptoNight heavy hashing power versus the Global hashing power of at Loki.


#### 3.1.3 Monopooling

We will use the term  Monopooling here which refers to single pools which dominate more than 51% of the hashrate of a coin’s network, this is a phenomenon which mostly affects small, GPU-mineable coins. Monopools are not necessarily malicious, although provided the economic incentives from an attacker they can exert their hashrate over a coin network. Monopooling seems to result as the culmination of many of the above phenomenon, being particularly endemic in coins that have an issues with Roaming hashrate.

If we consider a miner who switches between profitable coins using the CryptoNight hashing algorithm, switching which network the mine numerous times a day. The most profitable strategy for this miner is not always to seek the pool with the lowest fees, but to join a pool which will will produce more blocks and yeild a higher payout in the short time that they are mining. This reduces the variance during the time that they do mine ensuring that hashes are seldom wasted. If these pools reach over 50% of the hashrate they can also offer advantages to miners. For example in the event of a hard fork, the community and developers of a coin may seek to legitimise the longest chain as the the legitimate chain meaning profits are unlikely to be lost in a split.

On a number of coins this leads to large Monopools, by measuring the percentage hashrate of the largest pools of 18 small GPU-mineable coins the results indicated that on average these coins had a single pool that represented 53% of their hashrate.



Although these pools are primarily benign, they still pose a threat due to the high control they have over the network. It would be feasible for an attacker to bribe or hack a pool operator to perform a double spend.

## 4 Specification 

### 4.1 Block Validation 

As a result of the above problems, I propose that Loki utilise Service Nodes to validate the existing Proof of Work chain after N blocks.

### 4.2 Reorganisation limits

Checkpoints are one way to prevent reorganisations greater than N blocks. Adding a checkpoint to a blockchain dictates that once the checkpoint is reached the normal rules of Proof of Work are ignored. This means that when a checkpoint is set block reorganizations past the checkpoint are invalid, even if the proposed candidate for reorganisation contains a longer chain with higher difficulty targets.

The other, more naive approach is to simply cap reorganisations greater than P blocks. However there are serious problems with this approach, if an attacker can produce a chain of length P+1 and submit it to different edge of the network as the non attacking chain is being checkpointed, then the network will diverge  and will be unable to merge back into a single chain, effectively permanently forking the chain. This attack could, for example, deliberately target an exchange to put the exchange on a fork controlled by the attacker but distinct from the main network until manual maintenance by the exchange to re-sync to the proper network chain. This kind of attack is prevented by Service Node checkpointing as the attacker would need their two chains to be signed by the quorum to create a conflict.
 

### 4.3 Checkpointing rules

Already implemented in Loki are Service Node quorums, which dictate that each block 10 random Service Nodes are selected by the hash of the 10th previous block to assess the uptime proofs of other Service Nodes. To validate the blockchain every N blocks a new class of Service Node quorums should be selected. should be given an additional task, which is to add a checkpoint to the blockchain.

A Service Node checkpoint should consist of a blockhash, a block height and the signature of a super majority (66%) of the Service nodes in the relevant quorum. To avoid miners colluding to ignore checkpoints and reorg past the hard limits, the checkpoint should not be included as a transaction in the blockchain. Rather, the checkpoint data should be appended to the ‘top’ of the block after the fact, all data inside of blocks will maintain immutability but clients will append outside of the hashed data the Service Node data.



 Figure 5: example of a Checkpointed block 


#### Choosing chains

A Service node quorum is prevented from checkpointing more than one chain at anytime, so behaviour must be written for what should occur when either the quorum disagrees on which chain should be checkpointed. First, before any decision is made each node should share their candidate checkpoint chain with all other nodes in the quorum.

Some conditions should be established beyond normal block validation:

The proposed chain must be built from one of the last two checkpoints
Nodes shall not accept more than  (N*3) - 1 blocks since the last checkpoint (Where N is the reorg limit)
If two chains of proof of work are produced with the same cumulative difficulty Service Nodes should choose the chain based on the lowest hamming distance of the chain tip’s blockhash to 0
The latest checkpoint C3 can invalidate two checkpoints back to C1 

Once Service nodes internally agree on the correct chain they should all produce a signed copy of the block which should then be submitted to the network, if valid (containing a super majority of Service Node signatures) then the extra Service Node data should be appended to the relevant checkpointed block in all client databases.

Why have more than one checkpoint?
Adding additional checkpoints to the chain before finalisation adds a high degree of redundancy, in the rare case that a quorum is unable to come to consensus and misses a checkpoint the chain can continue uninterrupted, the network will then fall back onto either of the next two quorums which can sign and continue the chain. 

#### Service Node Data 

As discussed each checkpoint requires that data be appended to the top of blocks, practically this data should include a couple of things. 

The blockhash of one of the last two checkpointed blocks, the Service Node data hash of one of the last two checkpointed blocks, And a super majority of the selected quorum signatures singing the blockhash of the new block to be checkpointed

#### Practical issues with checkpointing

When checkpointing or setting reorganisation limits the question must always be asked: can the blockchain self repair, and in what situation could the blockchain become stuck or permanently forked?

If we imagine a protocol bug that causes the blockchain to stop after a reorganisation point or an invalid block is checkpointed then is no way to continue the chain since the longer chain can never reorganise past the first checkpoint on chain 1 in figure 3.



Figure 3: Blockchain with unrepairable checkpoints, N = 4 

The only way to overcome this issue is to allow Service Nodes to simultaneously sign multiple chains:, clients must measure the dominant chain as the one with the most Service Node signatures. However this reintroduces the risk of a double spend as Service Nodes can now authorise duplicate chains outside of N.


Figure 4: Blockchain as measured by signature weight

Practically this leads us to the conclusion that chain repairability and double spend protection are mutually exclusive. Since any transactions that were accepted in chain 1 (figure 4) could be reversed in chain 2. 

### 4.4 Proposed Checkpointing in Loki

After assessing the above issues I believe Loki should value double spend protection over absolute chain repariablity for a couple of reasons:

Chain repairability is already a problem present in Loki’s current PoW, if a chain of blocks is mined and then there is a bug encountered that bug is likely going to cause an issue for all miners, in this case the Loki team or a third party would release new software which allows miners and clients to sync past the faulty block. This would be the same scenario if an invalid block was to be checkpointed.

An active double spend on the Loki network is likely to cause more damage to the ecosystem than a temporary chain pause. Double spends can shake confidence and erode the trust of users and merchants which accept and exchange Loki, a chain pause is unlikely to cause the same effect.

Chain pauses with Loki’s checkpointing system should be extremely rare and only occur when, there is an intractable bug that produces a that is checkpointed more twice Or if three consecutive quorums are unable to reach consensus and checkpoint a block


### 4.5 Rewards and consequences

This request does not seek to increase the Service node block reward, however it would add additional conditions to a Service Node receiving its normal portion of the block rewards. The primary things we want to prevent Service Nodes from doing is

Checkpointing more than one chain
Checkpointing a chain that defies the consensus of the Service Node quorum
Failing to sign any chain despite being in a selected Service Node quorum

The first and second rules can be easily detected by allowing Service Nodes to submit any Service Nodes signature as evidence for a deregistration transaction if the Signature conflicts with an existing onchain checkpoint.

Rule 3 can be solved by investigating each Checkpoint for all Service Node signatures, because Service Node quorum selection is deterministic if any checkpoint is missing a Service Node signature more than once the quorum the next quorum should have the ability to vote that non-voting Service Node off the network through a deregistration transaction.

### 4.6 Setting N

N is the depth of the chain before checkpointing occurs. The appropriate N should consider the propensity of natural reorganisations in the Loki blockchain which occur at a rate of approximately 20 per 10000 blocks and at an average reorg height of 1.3 blocks. We should also consider the time it will take for a Service Node to actually checkpoint the chain, currently processing votes for deregistration takes approximately 1-2 blocks. Taking this into account I propose we set N = 4. In the future we should have the ability to lower this time if we have good reason to do so.

### 4.7 DDoS Protection

Because Service Node quorums are deterministically selected before checkpointing blocks and Service Node IP addresses are publicly stored on a DHT in Lokinet it is important to consider an attacker who would seek to disrupt consensus communications. Without protections an attacker could carry out a DDoS attack as Service Nodes were checkpointing the chain disrupt communications and prevent. Because a super majority of quorum signatures is required to sign a block and the quorum size is 20 the attack would only need to prevent communication of 18 nodes over the course of two consecutive checkpoints to freeze the blockchain. 

the novel protection for this attack is Service nodes whitelisting other nodes inside of their own active quorum when the network detects and attack. Practically Service Nodes would turn on this protection automatically when a the previous quorum has failed to sign a block. This method allows nodes to communicate to come to consensus on the correct chain to sign, without accepting connections from unknown IP addresses potentially attempting DDoS attacks, once finished this block should be removed and these nodes should regain normal functionality.  


### 4.8 Considerations for Merchants and Exchanges


An RPC call would be added to the Loki daemon to indicate the state of the current chains checkpoints, this RPC call will indicate the height of the last four blocks which were checkpointed. Merchants can with an extremely high degree of certainty accept any chain with more than two checkpoints and with a high degree of certainty accept a chain that has received a single checkpoint from a Service nodes quorum. Chains with a length of 12 that have three checkpoints are considered final and cannot be reorganised. 

If two checkpoints have been missed in a row this RPC call should indicate that there may be a serious error on the network and Merchants should disable withdrawals deposits temporarily while investigating the state of the chain.

## Backwards Capability 

#### The switch

This proposal is not backwards compatible and will require a hard fork to enact. As stated in 4.4 this change introduces a significant alteration to the way we will measure consensus on the Loki chain. Before making this change we should thoroughly discuss the potential consequences and recognise opposing opinions.

## Implementation 

Should this LIP pass a foundation vote code will be developed under a pull request which will be open to community review which I will link here.

