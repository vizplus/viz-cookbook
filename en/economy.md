# Economy

The conditions for the so-called *predictable* economy were created in the VIZ blockchain. If other systems have principles with decaying inflation, which puts participants who joined the system at different times in unequal conditions, then VIZ has a fixed inflation with rounds of 1 year ([10512000 blocks](https://github.com/VIZ-Blockchain/viz-cpp-node/blob/master/libraries/protocol/include/graphene/protocol/config.hpp#L28)).


***

## Inflation model

At the start of the chain, 50 million viz were distributed. Each round is based on a fixed inflation rate of 10%. A year after the start, there were 55 million viz in the network, which means that the inflation for the next round was already considered from 55 million viz, which led to the issuance of 5.5 million viz for the second year.

Due to fixed inflation, it becomes possible to predict the issue of viz for each block. In the second year, for example, each block is issued `5500000 / 10512000 = 0.523 viz`. And this number does not change until the next round begins.

## Token issue distribution

The existing VIZ model provides for the management of the economy by network delegates. The delegates elected by the network participants vote for the network parameters, among which there are parameters for the distribution of the token issue:

 - **inflation_witness_percent** — - an economic parameter that sets the percentage of the issue directed to the remuneration of delegates for maintaining the infrastructure of the blockchain system;
 - **inflation_ratio_committee_vs_reward_fund** — an economic parameter that determines the ratio of the issue sent to the DAO Fund and the Awards Fund.

> You can find out the current values of the parameters by executing an [API request `get_chain_properties`to the plugin `database_api`](plugins-api.md#database_api) or [via the network browser on the control.viz.world website](https://control.viz.world/tools/blocks/)

## DAO Fund

The DAO Foundation collects viz tokens to finance initiatives for the development of the ecosystem. If a network participant has decided to hold a competition, develop an application, benefit the entire network — he can apply for funding from the Fund. Any member of the network can vote both against and for the full or partial execution of the application. When the time comes for considering the application, the shares of all network participants who took part in the voting will be calculated and a decision will be made.

## Awards Fund

Each participant of the network has a resource that is renewable over time — energy. The spent energy is replenished at a rate of 100 percentage points in 5 days (20 p. p. per day, 0.83 p. p.per hour, etc.). The energy of the account cannot exceed 100%.

When a VIZ participant decides to award someone, he indicates the percentage of the account's energy that he wants to spend on the award. The blockchain considers the size of the participant's social capital and the energy spent on the award and correlates them with the sum of the awards of all participants for the last 5 days. As a result, the target account receives a reward in social capital from the issue in proportion to the social capital and the energy spent by the recipient.This ensures equal competitive access to the Awards Fund. The network participant decides for himself who to reward and for what, opening up the possibility of free choice and stimulating any actions and initiatives.

For example:
 - The author of the story and his readers can reward the illustrator who published the drawings of the characters;
 - On the website of questions and answers, participants can reward the one who helped solve and understand the question;
 - Viewers can reward a video blogger for an interesting topic or a training lesson.

## Selfgovernment

The entire economy of the VIZ is managed by the owners of social capital. They are the ones who vote for the delegates if they agree with their vision and model of managing the system (if there is no one to vote for, everyone can become a delegate himself and get the votes of other participants). The Awards Fund and the DAO Fund work according to a fair, equally shared management model.
