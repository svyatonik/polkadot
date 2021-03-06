# DMP Module

A module responsible for Downward Message Processing (DMP). See [Messaging Overview](../messaging.md) for more details.

## Storage

General storage entries

```rust
/// Paras that are to be cleaned up at the end of the session.
/// The entries are sorted ascending by the para id.
OutgoingParas: Vec<ParaId>;
```

Storage layout required for implementation of DMP.

```rust
/// The downward messages addressed for a certain para.
DownwardMessageQueues: map ParaId => Vec<InboundDownwardMessage>;
/// A mapping that stores the downward message queue MQC head for each para.
///
/// Each link in this chain has a form:
/// `(prev_head, B, H(M))`, where
/// - `prev_head`: is the previous head hash or zero if none.
/// - `B`: is the relay-chain block number in which a message was appended.
/// - `H(M)`: is the hash of the message being appended.
DownwardMessageQueueHeads: map ParaId => Hash;
```

## Initialization

No initialization routine runs for this module.

## Routines

Candidate Acceptance Function:

* `check_processed_downward_messages(P: ParaId, processed_downward_messages: u32)`:
    1. Checks that `DownwardMessageQueues` for `P` is at least `processed_downward_messages` long.
    1. Checks that `processed_downward_messages` is at least 1 if `DownwardMessageQueues` for `P` is not empty.

Candidate Enactment:

* `prune_dmq(P: ParaId, processed_downward_messages: u32)`:
    1. Remove the first `processed_downward_messages` from the `DownwardMessageQueues` of `P`.

Utility routines.

`queue_downward_message(P: ParaId, M: DownwardMessage)`:
    1. Check if the size of `M` exceeds the `config.max_downward_message_size`. If so, return an error.
    1. Wrap `M` into `InboundDownwardMessage` using the current block number for `sent_at`.
    1. Obtain a new MQC link for the resulting `InboundDownwardMessage` and replace `DownwardMessageQueueHeads` for `P` with the resulting hash.
    1. Add the resulting `InboundDownwardMessage` into `DownwardMessageQueues` for `P`.

## Session Change

1. Drain `OutgoingParas`. For each `P` happened to be in the list:
    1. Remove all `DownwardMessageQueues` of `P`.
    1. Remove `DownwardMessageQueueHeads` for `P`.
