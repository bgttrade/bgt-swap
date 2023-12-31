# BGT-swap

BGT-swap is a new automated market maker protocol that provides extensible and customizable pools.

## Contributing

If you’re interested in contributing please see our [contribution guidelines](./CONTRIBUTING.md)!

## Architecture

`BGT-swap` uses a singleton-style architecture, where all pool state is managed in the `PoolManager.sol` contract. Pool actions can be taken by acquiring a lock on the contract and implementing the `lockAcquired` callback to then proceed with any of the following actions on the pools:

- `swap`
- `modifyPosition`
- `donate`
- `take`
- `settle`
- `mint`

Only the net balances owed to the pool (negative) or to the user (positive) are tracked throughout the duration of a lock. This is the `delta` field held in the lock state. Any number of actions can be run on the pools, as long as the deltas accumulated during the lock reach 0 by the lock’s release. This lock and call style architecture gives callers maximum flexibility in integrating with the core code.

Additionally, a pool may be initialized with a hook contract, that can implement any of the following callbacks in the lifecycle of pool actions:

- {before,after}Initialize
- {before,after}ModifyPosition
- {before,after}Swap
- {before,after}Donate

Hooks may also elect to specify fees on swaps, or liquidity withdrawal. Much like the actions above, fees are implemented using callback functions.

The fee values, or callback logic, may be updated by the hooks dependent on their implementation. However _which_ callbacks are executed on a pool, including the type of fee or lack of fee, cannot change after  pool initialization.

## Repository Structure

All contracts are held within the `contracts` folder.

Note that helper contracts used by tests are held in the `contracts/test` subfolder within the contracts folder. Any new test helper contracts should be added here, but all foundry tests are in the `test/foundry-tests` folder.

```markdown
contracts/
----interfaces/
    | IPoolManager.sol
    | ...
----libraries/
    | Position.sol
    | Pool.sol
    | ...
----test
...
PoolManager.sol
test/
----foundry-tests/
```

## Local deployment and Usage

To utilize the contracts and deploy to a local testnet, you can install the code in your repo with forge:

To integrate with the contracts, the interfaces are available to use:

```solidity

import {IPoolManager} from 'contracts/interfaces/IPoolManager.sol';
import {ILockCallback} from 'contracts/interfaces/callback/ILockCallback.sol';

contract MyContract is ILockCallback {
    IPoolManager poolManager;

    function doSomethingWithPools() {
        // this function will call `lockAcquired` below
        poolManager.lock(...);
    }

    function lockAcquired(uint256 id, bytes calldata data) external returns (bytes memory) {
        // perform pool actions
        poolManager.swap(...)
    }
}

```

## License

The primary license for BGT-swap is the Business Source License 1.1 (`BUSL-1.1`)
- [Interfaces](./contracts/interfaces) have a General Public License
- Some [libraries](./contracts/libraries) and [types](./contracts/types/) have a General Public License
- [FullMath.sol](./contracts/libraries/FullMath.sol) has an MIT License

Each of these files states their license type.
