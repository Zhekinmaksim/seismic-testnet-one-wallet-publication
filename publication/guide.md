# How To Reproduce One Full Seismic Testnet Wallet Flow

This guide shows how to turn one funded Seismic wallet into a complete, publishable testnet run.

## Goal

Use one wallet to do more than deploy a contract. The target is a full Seismic flow that includes:

- three contract deployments
- shielded writes
- signed reads
- a private token transfer
- Walnut interactions
- Directory viewing key registration
- Mercury precompile checks

## Step 1: Start With One Funded Wallet

Reference wallet:

`0xBCb5ABFc5C168c27437a3c58D75D361E85dcab1e`

The wallet should have enough testnet funds for:

- three deployments
- several shielded transactions
- viewing key registration
- follow-up verification reads

## Step 2: Give The Run A Distinct Profile

Avoid making the run look like a generic copy of the same demo contracts.

Reference profile:

- Counter label: `Seismic Counter acct-001`
- Counter variantId: `1001`
- Token name: `Seismic Test Token acct-001`
- Token symbol: `S001`
- Token variantId: `2001`
- Walnut label: `Seismic Walnut acct-001`
- Walnut variantId: `3001`

## Step 3: Deploy Three Contracts

Deploy:

1. `Counter`
2. `Private Token`
3. `Walnut`

Reference deployment addresses:

- Counter: `0xBc82Be737749F33c32e33097C7BdB3F0804050c1`
- Private Token: `0x67827B110781D3936B782ffC34fbdb9e5073EB5D`
- Walnut: `0xAc283F36B1879b795859106D13498eCB3Eb43ff8`

## Step 4: Run The Counter Flow

Use the deployed `Counter` contract to verify the basic private interaction path.

Do this:

1. Send the first shielded increment.
2. Send the second shielded increment.
3. Run a signed read to confirm the final value.

Reference result:

- First increment tx: `0x03de83ffeed73399fd70d34921bcc70cfeb3859b17a607801faef443198fcb54`
- Final value: `5`

## Step 5: Run The Private Token Flow

Use the private token contract for a real balance change, not just deployment.

Do this:

1. Read the initial token balance.
2. Send a private transfer.
3. Read the balance again.

Reference result:

- Initial balance: `1017`
- Transfer amount: `6`
- Final balance: `1011`
- Transfer tx: `0xe8d0901d47222325471b6d58cbddcd0e7f7c8b12653e99ec3d248b244a01a0d0`

## Step 6: Run The Walnut Flow

Use `Walnut` to show that private conditional logic also works.

Do this:

1. Attempt a transparent read before contribution.
2. Call `shake`.
3. Call `hit`.
4. Run the signed read.
5. Call `reset`.

Reference result:

- Final Walnut signed read: `16`

## Step 7: Register A Directory Viewing Key

This validates account-level privacy infrastructure, not just contract logic.

Reference transaction:

- Viewing key registration: `0xbe5fe0fc03eebc81847a78eb599911aec80ec306f2e5435ca717ccd4d35b5c53`

## Step 8: Check Mercury Precompiles

Verify the lower-level cryptographic path in the same run.

Check:

- RNG
- ECDH / HKDF
- AES-GCM roundtrip
- secp256k1 signing

## Step 9: Record Final Outputs

At the end, keep one clean result set:

- wallet address
- deployed contract addresses
- important transaction hashes
- final Counter value
- token balance before and after transfer
- final Walnut value
- viewing key tx
- Mercury success status

## What Counts As A Successful Run

Treat the flow as successful if:

- all three contracts deploy
- Counter shielded writes succeed
- Counter signed read returns the expected value
- the private token transfer succeeds
- Walnut completes
- viewing key registration succeeds
- Mercury precompile checks pass

If all of that is true, you have a strong one-wallet Seismic testnet case study that is detailed enough to publish.
