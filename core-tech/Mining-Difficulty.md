Mining difficulty is a measure of how many attempts on average it is required to find the proof-of-work solution required to mine a block and receive the mining reward.
One can define the Difficulty as the inverse probability of a random solution being the correct one. Thus, a difficulty of 100 means that one in 100 tries should produce a valid block in average.

# Why change the difficulty?

In Proof of Work blockchains, the difficulty is a dynamic parameter, periodically retargeted to reflect the fluctuations in the total computational power of all the miners. Accounting for changes in mining competition, as well as improvements in mining technology.

The goal of updating the difficulty is to keep the average block time at a certain value. In Beam, the target block time is 60 seconds, which underpins a constant currency issuance and the transaction settlement time of the network.

Here’s an example. Let’s assume we have 100 miners in our network. They are trying different random solutions and find one roughly every minute. Now 100 more miners join, the hashrate of the network doubles, and it will take just 30 seconds to find the solution in average. And if 200 and more join, the time will halve into 15 seconds, and so on. 

To mitigate that, Beam nodes adjust [shall we mention that they reach consensus here?]the difficulty, increasing the difficulty of the proof of work algorithm as more miners join, and lowering it if the number of miners decreases.

# How Difficulty is calculated in Beam

In Beam, the difficulty is reassessed with every block, by every client independently
The algorithm looks at the average time and the average difficulty the last 1440 blocks. f the time required to mine the last 1440 blocks is higher or lower than 24 hours, the difficulty is retargeted accordingly. 

The detailed algorithm is outlined below:

1. Look at the last 7 blocks. Identify the block that has the median block time. This is the Window End block.
1. Look at blocks from 127 to 120 before the current block (7 blocks altogether). Identify the block that has the median block time out of those. This will be the Window Start block.
1. Sum all the difficulties of the blocks from Window Start to Window End. This is Delta Work.
1. Calculate the time difference between Window Start and Window End blocks. This is Delta Time.
1. Calculate the difficulty for next block as: NewDifficulty = (Delta Work / Delta Time) * 60seconds.
1. The Delta Time is bounded by 1 hour and 4 hours. I.e. if the difference is less than 1 hour, 1 hours is still used as Delta Time, same for differences larger than 4 hours. This is done to prevent extreme changes in difficulty.

The algorithm above ensures that as the total solution power of the network fluctuates, the difficulty is gradually adjusted and thus the target block time is ensured.


