# Smart Contract Review 

## Overview 

The SeawaterAMM contract is a modular proxy-based Automated Market Maker (AMM) designed to support an upgradeable architecture with distinct executors handling various operational roles. The contract leverages specific storage slots for executors, allowing seamless updates and customizations in the deployed contract's behavior without redeployment.

The contract serves as the backbone of a decentralized AMM protocol, enabling:

- Token swaps with optional permits for gasless approvals.
- Liquidity management through pool creation, liquidity provision, and position updates.
- Administrative functions accessible only to the designated proxy admin.

# Key Functions

## Admin Functions

- Constructor: Initializes the contract by setting the proxy admin, seawater admin, NFT manager, and various executors. Additionally, it triggers initialization in the ExecutorAdmin via a delegate call.

- updateProxyAdmin: Allows the proxy admin to change the admin address, preserving access control.

- updateExecutors: Lets the proxy admin update executor addresses, supporting the contract's modularity.

## AMM Functions

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


# Functions

## updateProxyAdmin
```
 function updateProxyAdmin(address newAdmin) public onlyProxyAdmin {
        _setProxyAdmin(newAdmin);
    }
```

- It Allows the current proxy admin to set a new admin address. Updates the ``proxyAdmin`` variable with ``newAdmin`` and ensures control over admin permissions can transfer smoothly and securely.


## updateExecutors

```
function updateExecutors(
        ISeawaterExecutorSwap executorSwap,
        ISeawaterExecutorSwapPermit2 executorSwapPermit2,
        ISeawaterExecutorQuote executorQuote,
        ISeawaterExecutorPosition executorPosition,
        ISeawaterExecutorUpdatePosition executorUpdatePosition,
        ISeawaterExecutorAdmin executorAdmin,
        ISeawaterExecutorFallback executorFallback
    ) public onlyProxyAdmin {
        _setProxies(executorSwap, executorSwapPermit2, executorQuote, executorPosition, executorUpdatePosition, executorAdmin, executorFallback);
    }
```

- The ``updateExecutors`` function Enables the proxy admin to update executor addresses for different operational roles (e.g swap, quote, position, admin). it takes in parameters ``newExecutorSwap``, ``newExecutorQuote``, ``newExecutorPosition``, ``newExecutorAdmin``. The function Sets each executor address to the provided parameter if it is non-zero and it allows each executor to be updated individually.



## directDelegate

```
 function directDelegate(address to) internal {
        assembly {
            calldatacopy(0, 0, calldatasize())

            
            let result := delegatecall(gas(), to, 0, calldatasize(), 0, 0)

            
            returndatacopy(0, 0, returndatasize())

            switch result
            
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }

```

- The directDelegate function here helps faciitate low-level delegate call to an external contract. This would possibly be allowing the AMM to integrate new features or updates without changing its own code. it takes a ``to`` address parameter when it needs to be called.


## createPoolD650E2D0

```
function createPoolD650E2D0(
        address ,
        uint256 ,
        uint32 ,
        uint8 ,
        uint128 
    ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``createPoolD650E2D0`` function delegate the creation of a new liquidity pool to the Admin Executor contract. it uses the directDelegate to passes the function call to a specific implementation contract (Seawater Executor) to handle the logic, rather than handling it within SeawaterAMM itself.


## enablePool579DA658

```
function enablePool579DA658(
        address ,
        bool
    ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``enablePool579DA658`` function is inherited from ``IseawaterExecutor`` and delegates the enabling of a particular pool that has been created using the ``directDelegate``. it uses ``directDelegate`` where the actual logic for enabling the pool is handled by an external contract. it takes an ``address`` which is the address of the pool and a ``bool`` as parameters.


## authoriseEnabler5B17C274

```
function authoriseEnabler5B17C274(
        address ,
        bool
    ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``authoriseEnabler5B17C274`` function is inherited from ``IseawaterExecutor`` and it delegates the authorzation of a specific address as an "Enabler" which is  permitted to activate or manage pools, through functions like ``enablePool579DA658`` using the ``directDelegate``. it takes in an ``address`` which is the address of the person to be the enabler and also a ``bool`` as paramerers. 


## setSqrtPriceFF4DB98C

```
function setSqrtPriceFF4DB98C(address , uint256 ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``setSqrtPriceFF4DB98C`` function is inherited from ``IseawaterExecutor`` and it is used in setting or adjusting the square root price for a pool using the ``directDelegate``. it takes an ``address `` which is the address of the pool, and a ``price`` as parameters.


## updateNftManager9BDF41F6

```
function updateNftManager9BDF41F6(address ) external {
        directDelegate(_getExecutorAdmin());
    }
```
- The ``updateNftManager9BDF41F6`` function is inherited from  ``IseawaterExecutor``  and it is used for assigning or updating the contract or entity responsible for managing NFTs within the AMM ecosystem using the ``directDelegate``. it takes and ``address`` which is the address of the person that is being assigned is the Nft Manager as parameter.  



## fallback()

```
fallback() external {
        require(msg.data.length > 3);
        require(msg.data[0] == 0);
        // swaps
        if (uint8(msg.data[2]) == 0) directDelegate(_getExecutorSwap());
        // update positions
        else if (uint8(msg.data[2]) == 1) directDelegate(_getExecutorUpdatePosition());
        // positions
        else if (uint8(msg.data[2]) == 2) directDelegate(_getExecutorPosition());
        // admin
        else if (uint8(msg.data[2]) == 3) directDelegate(_getExecutorAdmin());
        // swap permit 2
        else if (uint8(msg.data[2]) == 4) directDelegate(_getExecutorSwapPermit2());
        // quotes
        else if (uint8(msg.data[2]) == 5) directDelegate(_getExecutorQuote());
        else directDelegate(_getExecutorFallback());
    }
```

- Handles any function calls not explicitly defined in this contract. it delegates the call to ``executorFallback``, if one exists.