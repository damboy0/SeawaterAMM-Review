# Smart Contract Review 

## Overview 

The SeawaterAMM contract is a modular proxy-based Automated Market Maker (AMM) designed to support an upgradeable architecture with distinct executors handling various operational roles. The contract leverages specific storage slots for executors, allowing seamless updates and customizations in the deployed contract's behavior without redeployment.

The contract serves as the backbone of a decentralized AMM protocol, enabling:

- Token swaps with optional permits for gasless approvals.
- Liquidity management through pool creation, liquidity provision, and position updates.
- Administrative functions accessible only to the designated proxy admin.

## Key Functions

# Admin Functions

- Constructor: Initializes the contract by setting the proxy admin, seawater admin, NFT manager, and various executors. Additionally, it triggers initialization in the ExecutorAdmin via a delegate call.

- updateProxyAdmin: Allows the proxy admin to change the admin address, preserving access control.

- updateExecutors: Lets the proxy admin update executor addresses, supporting the contract's modularity.

# AMM Functions

Swap Functions: Provides functions like ``swapIn``, ``swapOut``, ``swapPermit2``, and others for token swapping with optional permits. Swaps are handled by delegating calls to specific swap executors.

- Quote Functions: Retrieves token quotes through executors, allowing for flexible querying of token prices.

- Position Management: Functions to mint, burn, and transfer liquidity positions, alongside pool management, enable comprehensive liquidity control for AMM operations.


# Constants in SeawaterAMM

## Constants for Executor Storage Slots

```bytes32 constant EXECUTOR_SWAP_SLOT = keccak256("seawater.amm.executor.swap");
bytes32 constant EXECUTOR_QUOTE_SLOT = keccak256("seawater.amm.executor.quote");
bytes32 constant EXECUTOR_POSITION_SLOT = keccak256("seawater.amm.executor.position");
bytes32 constant EXECUTOR_ADMIN_SLOT = keccak256("seawater.amm.executor.admin");
```

- These constants store each executor’s address in a unique storage slot determined by the hash of a descriptive label. This is minimizes the risk of unintentional overwrites in storage.
- The keccak hash ensures a collision-resistant location for each executor address, even if the contract’s storage layout changes later.


 ## Constants for Well-Known Tokens or Fee Receivers

 ```
address constant WETH_ADDRESS = 0x...; // WETH token address (wrapped ETH)
address constant USDC_ADDRESS = 0x...; // USDC token address
```

- If the contract relies on specific tokens like WETH or USDC, having their addresses as constants allows easy integration and saves on gas costs. Also, it ensures that these tokens’ addresses don’t change by accident, which can be crucial for token-pair operations and price quotes


## Basis Points for Fee Calculations

```
uint256 constant FEE_RATE_BASIS_POINTS = 30; // Represents a 0.3% fee
```

- This constant can be used in functions where the AMM charges fees, ensuring consistent calculations across the contract. Defining it as a constant also makes it immutable, guaranteeing a set rate unless the entire contract is upgraded.


## Timeout Constants for Permit and Transaction Deadlines

```
uint256 constant DEFAULT_DEADLINE_BUFFER = 300; // Buffer of 5 minutes
```

- Used in functions requiring permit authorizations, ensuring that authorizations cannot be reused indefinitely. Timeout buffer as a constant helps protect transactions from failing due to network delays or unexpected latency.