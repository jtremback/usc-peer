# Lifecycle of a channel

## Opening

A channel starts when a user uses a usc caller to create some state to put in the opening tx and decides on a hold period. They create an opening tx signed by only them. They send it to the counterparty and create a new channel in pending open phase. They also start up a daemon to check for the channel with the judge at least every hold period.

When the counterparty receives the opening tx, they verify the signature and check that there is not already a channel with that ID. They then save the channel.

When a user decides to check for proposed channels, they look up all channels with a phase of PENDING_OPEN and an OpeningTx with one signature.

When a user decides to confirm a proposed channel, they send a message to usc, which signs the OpeningTx, saves the channel with the new opening tx envelope, and sends it to the judge.

When the judge receives the opening tx, they verify the signatures and check that there is not already a channel with that ID. They then save the channel.

When a judge decides to check for proposed channels, they look up all channels with a phase of PENDING_OPEN.

When a judge decides to confirm a proposed channel, they send a message to usc, which signs the OpeningTx, and starts serving the channel.

When the original channel starters daemon (account 0) finds the fully signed opening tx being served by the judge, they check all three signatures and change the channel state to Open. They then save the channel.


## Updating

A user updates a channel by having the caller generate a new state and send it to usc. Usc signs it, sends it to the counterparty, and saves it as LastUpdateTx in the Channel.

When the counterparty receives an update tx, it checks if the sequence number is higher than the sequence number of LastFullUpdateTx. It then saves the update tx as LastUpdateTx.

When the user wants to check if there are update txs to be approved, she looks for channels with a LastUpdateTx not signed by her.

When a user wants to approve an update tx, she sends the channelId to usc. Usc checks if the channel has an update tx to be approved and if so signs it and saves it in LastFullUpdateTx. And clears LastUpdateTx.


## Closing

A user closes a channel by sending the LastFullUpdateTx to the judge.


## Daemon

The usc daemon checks with the judge of every channel every once in a while. If it finds that an update tx has been posted, it places the channel into PENDING_CLOSED if it isnt already, and checks to make sure that its LastFullUpdateTx is not higher than the update tx that the judge has. If the LastFullUpdateTx is higher, it sends that to the judge.