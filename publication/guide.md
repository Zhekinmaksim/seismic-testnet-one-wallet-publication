# How To Reproduce One Full Seismic Testnet Wallet Flow

This guide shows how to turn one funded Seismic wallet into a complete, publishable testnet run.

The earlier version of this guide described the flow at a high level. This version makes the deployment path explicit, with the exact project structure and `sforge create` pattern used in the working run.

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

## Step 2: Prepare Your Environment

You need:

- Seismic Foundry with `sforge`
- a funded private key
- the Seismic testnet RPC

Reference RPC:

`https://gcp-1.seismictest.net/rpc`

Reference chain ID:

`5124`

Export the two values you will use during deployment:

```bash
export RPC_URL="https://gcp-1.seismictest.net/rpc"
export PRIVATE_KEY="0xYOUR_PRIVATE_KEY"
```

Before deploying anything, verify that the RPC answers:

```bash
curl -sS -X POST "$RPC_URL" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

You should get:

```json
{"jsonrpc":"2.0","id":1,"result":"0x1404"}
```

`0x1404` is `5124` in decimal.

## Step 3: Create A Minimal Seismic Foundry Project

Make a fresh working directory:

```bash
mkdir -p seismic-one-wallet-flow/src
cd seismic-one-wallet-flow
```

Create `foundry.toml`:

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
```

This is enough for a minimal Seismic Foundry build.

## Step 4: Add The Counter Contract

Create `src/Counter.sol`:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Counter {
    suint256 private number;
    uint256 public threshold;
    string public label;

    constructor(uint256 _threshold, string memory _label) {
        number = suint256(0);
        threshold = _threshold;
        label = _label;
    }

    function increment(suint256 amount) public {
        number += amount;
    }

    function getNumber() public view isThresholdReached returns (uint256) {
        return uint256(number);
    }

    modifier isThresholdReached() {
        require(number >= suint256(threshold), "Threshold not reached");
        _;
    }
}
```

### Important

The constructor takes **two** arguments:

1. `threshold`
2. `label`

That is why the deployment command later uses:

```bash
--constructor-args "5" "Seismic Counter acct-001"
```

## Step 5: Build Before You Deploy

Compile first:

```bash
sforge build
```

If the build succeeds, you are ready to deploy.

## Step 6: Deploy Counter

Deploy `Counter` with the same pattern used in the working Seismic run:

```bash
sforge create \
  --rpc-url "$RPC_URL" \
  --private-key "$PRIVATE_KEY" \
  --broadcast \
  src/Counter.sol:Counter \
  --constructor-args "5" "Seismic Counter acct-001"
```

What each part does:

- `--rpc-url`: points to Seismic testnet
- `--private-key`: signs the deployment transaction
- `--broadcast`: sends the transaction onchain instead of simulating it
- `src/Counter.sol:Counter`: tells `sforge` which contract to deploy
- `--constructor-args`: passes the constructor arguments in order

### What success looks like

The output should include lines like:

```text
Deployer: 0x...
Deployed to: 0x...
Transaction hash: 0x...
```

The important value is the one after `Deployed to:`. Save it. That is your deployed Counter contract address.

## Step 7: Deploy The Private Token Contract

In the working flow, the token contract was a lightweight private token used for balance checks and transfers.

Create `src/MiniSRC20.sol`:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract MiniSRC20 {
    string public name;
    string public symbol;
    uint8 public decimals;

    mapping(address => suint256) private balances;

    constructor(
        string memory _name,
        string memory _symbol,
        uint8 _decimals,
        uint256 initialSupply
    ) {
        name = _name;
        symbol = _symbol;
        decimals = _decimals;
        balances[msg.sender] = suint256(initialSupply);
    }

    function balanceOf() public view returns (uint256) {
        return uint256(balances[msg.sender]);
    }

    function transfer(address to, suint256 amount) public returns (bool) {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;
        return true;
    }
}
```

Compile again:

```bash
sforge build
```

Then deploy:

```bash
sforge create \
  --rpc-url "$RPC_URL" \
  --private-key "$PRIVATE_KEY" \
  --broadcast \
  src/MiniSRC20.sol:MiniSRC20 \
  --constructor-args \
  "Seismic Test Token acct-001" \
  "S001" \
  "18" \
  "1017"
```

Constructor arguments here are:

1. token name
2. token symbol
3. decimals
4. initial supply

Again, save the address printed after `Deployed to:`.

## Step 8: Deploy Walnut

`Walnut` is the contract used to exercise more interesting private logic.

Create `src/Walnut.sol`:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Walnut {
    suint256 private number;
    uint256 public shell;
    uint256 public round;
    string public label;
    mapping(uint256 => mapping(address => uint256)) private hitsPerRound;

    constructor(uint256 startNumber, uint256 initialShell, string memory _label) {
        number = suint256(startNumber);
        shell = initialShell;
        round = 1;
        label = _label;
    }

    function shake(suint256 numShakes) public {
        number += numShakes;
    }

    function hit() public {
        hitsPerRound[round][msg.sender] += 1;
        if (shell > 0) {
            shell -= 1;
        }
    }

    function look() public view onlyContributor returns (uint256) {
        return uint256(number);
    }

    function reset(uint256 newNumber, uint256 newShell) public {
        round += 1;
        number = suint256(newNumber);
        shell = newShell;
    }

    function contributions(address user) public view returns (uint256) {
        return hitsPerRound[round][user];
    }

    modifier onlyContributor() {
        require(hitsPerRound[round][msg.sender] > 0, "Not a contributor");
        _;
    }
}
```

Build again:

```bash
sforge build
```

Then deploy:

```bash
sforge create \
  --rpc-url "$RPC_URL" \
  --private-key "$PRIVATE_KEY" \
  --broadcast \
  src/Walnut.sol:Walnut \
  --constructor-args "12" "1" "Seismic Walnut acct-001"
```

Constructor arguments here are:

1. start number
2. initial shell
3. label

Save the `Deployed to:` address.

## Step 9: What You Should Have After Deployment

At this point you should have three deployed addresses:

- Counter
- Private Token
- Walnut

In the reference run, those addresses were:

- Counter: `0xBc82Be737749F33c32e33097C7BdB3F0804050c1`
- Private Token: `0x67827B110781D3936B782ffC34fbdb9e5073EB5D`
- Walnut: `0xAc283F36B1879b795859106D13498eCB3Eb43ff8`

If you do not have all three addresses yet, stop here and fix deployment first. The rest of the guide assumes deployment is already complete.

## Step 10: Run The Counter Flow

Use the deployed `Counter` contract to verify the basic private interaction path.

Do this:

1. Send the first shielded increment.
2. Send the second shielded increment.
3. Run a signed read to confirm the final value.

Reference result:

- First increment tx: `0x03de83ffeed73399fd70d34921bcc70cfeb3859b17a607801faef443198fcb54`
- Final value: `5`

## Step 11: Run The Private Token Flow

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

## Step 12: Run The Walnut Flow

Use `Walnut` to show that private conditional logic also works.

Do this:

1. Attempt a transparent read before contribution.
2. Call `shake`.
3. Call `hit`.
4. Run the signed read.
5. Call `reset`.

Reference result:

- Final Walnut signed read: `16`

## Step 13: Register A Directory Viewing Key

This validates account-level privacy infrastructure, not just contract logic.

Reference transaction:

- Viewing key registration: `0xbe5fe0fc03eebc81847a78eb599911aec80ec306f2e5435ca717ccd4d35b5c53`

## Step 14: Check Mercury Precompiles

Verify the lower-level cryptographic path in the same run.

Check:

- RNG
- ECDH / HKDF
- AES-GCM roundtrip
- secp256k1 signing

## Step 15: Record Final Outputs

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
