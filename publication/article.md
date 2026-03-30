# One Wallet, Three Contracts, One Complete Seismic Flow

I wanted one Seismic testnet run that felt complete.

Not a faucet screenshot. Not one contract in an explorer tab. Not a single transaction dressed up as a milestone.

The goal was simpler and harder than that: take one funded wallet, deploy several contracts from it, move through private interactions, verify signed reads, and finish with the infrastructure checks that make Seismic interesting in the first place.

The reference wallet for this run was:

`0xBCb5ABFc5C168c27437a3c58D75D361E85dcab1e`

## What Was Deployed

Three contracts were deployed from that wallet:

- `Counter`: `0xBc82Be737749F33c32e33097C7BdB3F0804050c1`
- `Private Token`: `0x67827B110781D3936B782ffC34fbdb9e5073EB5D`
- `Walnut`: `0xAc283F36B1879b795859106D13498eCB3Eb43ff8`

This was not meant to look like a generic copy of the same template. The run had its own profile:

- Counter label: `Seismic Counter acct-001`
- Counter variantId: `1001`
- Token name: `Seismic Test Token acct-001`
- Token symbol: `S001`
- Token variantId: `2001`
- Walnut label: `Seismic Walnut acct-001`
- Walnut variantId: `3001`

That small layer of variation matters. Even a single-wallet run reads better when it looks intentional and traceable.

## The Counter Flow

The first useful test was `Counter`.

Two shielded increments were sent, and the contract was then checked with a signed read. The final value came back as `5`.

Reference transaction:

- First increment tx: `0x03de83ffeed73399fd70d34921bcc70cfeb3859b17a607801faef443198fcb54`

The point here was not the number itself. It was proof that the encrypted write path and the signed read path both held together in the same run.

## The Private Token Flow

After that came the private token contract.

The starting balance was `1017`. A private transfer of `6` was sent. The resulting balance came back as `1011`.

Reference transaction:

- Token transfer tx: `0xe8d0901d47222325471b6d58cbddcd0e7f7c8b12653e99ec3d248b244a01a0d0`

This made the run feel more grounded. It was no longer just a contract deployment story. It was a real private token interaction with a measurable before-and-after state.

## The Walnut Flow

`Walnut` was useful for a different reason. It pushed the run into private conditional logic.

The flow went through:

- transparent read before contribution
- `shake`
- `hit`
- signed read
- `reset`

The final Walnut signed read returned `16`.

That made this part more interesting than a basic storage update. It showed a private execution path with gating and state transitions, not just a simple write.

## Directory And Mercury

The run also covered the privacy infrastructure layer.

Directory viewing key registration completed successfully:

- Viewing key tx: `0xbe5fe0fc03eebc81847a78eb599911aec80ec306f2e5435ca717ccd4d35b5c53`

Mercury checks also passed:

- RNG
- ECDH / HKDF
- AES-GCM roundtrip
- secp256k1 signing

That part matters because it shows the run was not isolated to contracts alone. The lower-level privacy stack also behaved as expected in the same session.

## Why This Run Feels Publishable

What made this Seismic testnet run worth sharing was not the number of clicks or transactions by itself.

It was the shape of the flow:

- one wallet
- three deployments
- shielded writes
- signed reads
- a private token balance change
- Walnut logic
- Directory registration
- Mercury verification

That is enough to read like a real end-to-end execution path. It is specific, reproducible, and easy to inspect in public.

For one wallet, that is already a strong Seismic testnet story.
