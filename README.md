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