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
        address token ,
        uint256 sqrtPriceX96 ,
        uint32 fee ,
        uint8  tickSpacing,
        uint128 maxLiquidityPerTick
    ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``createPoolD650E2D0`` function delegate the creation of a new liquidity pool to the Admin Executor contract. it uses the directDelegate to passes the function call to a specific implementation contract (Seawater Executor) to handle the logic, rather than handling it within SeawaterAMM itself.


## collectProtocol7540FA9F

```
function collectProtocol7540FA9F(
        address pool,
        uint128 amount0 ,
        uint128 amount1 ,
        address recipient 
    ) external returns (uint128, uint128) {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``collectProtocol7540FA9F`` function collects accumulated protocol fees from the Seawater AMM's liquidity pool to a designated treasury, administrator, or governance contract using the ``directDelegate``.  it takes an ``address`` which is the pool address, an ``amount0`` and ``amount1`` which is the amount to be sent and ``address`` of the recipient which the money is sent to as parameters.


## enablePool579DA658

```
function enablePool579DA658(
        address pool ,
        bool enabled
    ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``enablePool579DA658`` function is inherited from ``IseawaterExecutor`` and delegates the enabling of a particular pool that has been created using the ``directDelegate``. it uses ``directDelegate`` where the actual logic for enabling the pool is handled by an external contract. it takes an ``address`` which is the address of the pool and a ``bool`` "enabled" as parameters.


## authoriseEnabler5B17C274

```
function authoriseEnabler5B17C274(
        address enabler ,
        bool enabled
    ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``authoriseEnabler5B17C274`` function is inherited from ``IseawaterExecutor`` and it delegates the authorzation of a specific address as an "Enabler" which is  permitted to activate or manage pools, through functions like ``enablePool579DA658`` using the ``directDelegate``. it takes in an ``address`` which is the address of the person to be the enabler and also a ``bool`` "enabled" as paramerers. 


## setSqrtPriceFF4DB98C

```
function setSqrtPriceFF4DB98C(address pool , uint256 price ) external {
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


## updateEmergencyCouncil7D0C1C58

```
 function updateEmergencyCouncil7D0C1C58(address council ) external {
        directDelegate(_getExecutorAdmin());
    }
```

- The ``updateEmergencyCouncil7D0C1C58`` function is responsible for managing the assignment or update of an "Emergency Council" using the ``directDelegate``. Emergency Council typically refers to a group or address with elevated permissions, tasked with handling urgent protocol actions. This council is generally in place to respond to critical situations like hacks, major bugs, or economic attacks against the protocol. it takes the ``address`` to be given the Emergenc council role as a parameter.

//Swap functions

## swap904369BE

```
function swap904369BE(address pool , bool zeroForOne , int256 amount , uint256 priceLimit ) external returns (int256, int256) {
        directDelegate(_getExecutorSwap());
    }
```

- The ``swap904369BE`` function facilitates the exchange of one asset (Token A) for another (Token B) within a liquidity pool managed by the Seawater AMM through a low-level call using the ``directDelegate``. This allows users to trade between tokens in the liquidity pool with protections against excessive slippage and fee calculations that benefit liquidity providers. It takes an ``address pool`` which is the address of the pool, a ``bool zeroForOne`` to specify the direction of the swap between two tokens ,an ``int256 amount`` which is the amount to be swapped, and ``uint256 priceLimit`` which take the price limit as parameters


## quote72E2ADE7

```
function quote72E2ADE7(address pool , bool zeroForOne , int256 amount , uint256 priceLimit ) external {
        directDelegate(_getExecutorQuote());
    }
```

- The ``quote72E2ADE7`` function is responsible for calculating or returning a quote or estimated amount for a potential swap between two tokens without actually executing the swap. It takes an ``address pool`` which is the address of the pool, a ``bool zeroForOne`` to specify the direction of the swap between two tokens ,an ``int256 amount`` which is the amount to be swapped, and ``uint256 priceLimit`` which take the price limit as parameters. 


## quote2CD06B86E

```
 function quote2CD06B86E(address  to , address from , uint256 amount , uint256  minOut) external {
        directDelegate(_getExecutorQuote());
    }
```

- The ``quote2CD06B86E`` function is responsible for retrieving a quote for swapping an amount of tokens from the ``from`` address to the ``to`` address, ensuring that the minimum output (``minOut``) requirement is met. This quote does not execute the swap but provides the estimated outcome to the caller, which is useful for planning trades and assessing price impacts. it takes the ``to`` which is the address of the token to receive in the swap, ``from`` Address of the token being swapped, ``amount`` Amount of from tokens to be swapped and ``minOut`` Minimum acceptable amount of to tokens to be received as parameters.



## swapPermit2EE84AD91

```
function swapPermit2EE84AD91(
        address pool,
        bool zeroForOne ,
        int256 amount ,
        uint256 priceLimit ,
        uint256 nonce ,
        uint256  deadline ,
        uint256  maxAmount ,
        bytes memory sig 
    ) external returns (int256, int256) {
        directDelegate(_getExecutorSwapPermit2());
    }
```


- The ``swapPermit2EE84AD91`` function enables swap of tokens in a way that also authorizes SeawaterAMM to spend the user’s tokens, using a permit instead of a separate approve transaction. This function is efficient for gas and user experience since it bundles approval and swap actions into a single step. 

Parameters:

- ``pool``: Address of the liquidity pool for the swap.
- ``zeroForOne``: Boolean indicating swap direction.
- ``amount``: Amount of tokens to be swapped.
- ``priceLimit``: Price limit for the swap to prevent slippage.
- ``nonce``: Unique identifier for permit signatures.
- ``deadline``: Expiration time for the permit authorization.
- ``maxAmount``: Maximum amount of tokens that can be swapped.
- ``sig``: Signature data (from the permit) for authorizing token transfers.



## swap2ExactIn41203F1D

```
 function swap2ExactIn41203F1D(address /* tokenA */, address /* tokenB */, uint256 /* amountIn */, uint256 /* minAmountOut */) external returns (uint256, uint256) {
        directDelegate(_getExecutorSwap());
    }
```

- The ``swap2ExactIn41203F1D`` function Executes a token swap , where the input amount of one token is provided exactly, and the output amount of another token is determined by the pool's current exchange rate. 

Parameters:

` ``tokenIn``: The address of the token being exchanged (input token).
- ``tokenOut``: The address of the token being received (output token).
- ``amountIn``: The exact amount of tokenIn that will be swapped.


## swap2ExactInPermit236B2FDD8

```
function swap2ExactInPermit236B2FDD8(
        address from ,
        address  to ,
        uint256  amount ,
        uint256  minOut ,
        uint256  nonce ,
        uint256  deadline ,
        bytes memory  sig 
    ) external returns (uint256, uint256) {
        directDelegate(_getExecutorSwapPermit2());
    }
```

-  The ``swap2ExactInPermit236B2FDD8`` function facilitates an exact-input swap, where the user specifies the exact amount of the input token they want to swap using the ``directDelegate``.It also make use of a "Permit2" mechanism, which may be an enhanced version of the EIP-2612 "Permit" feature, allowing token approvals to be signed off-chain and enabling gasless approvals. 

Parameters:

- ``inputAmount``: The exact amount of tokens the user wishes to swap.
- ``outputToken``: The token the user wants to receive.
- ``inputToken``: The token the user is swapping from.
- ``permitData``: Off-chain signed data for Permit2 authorization, allowing the contract to access the user's tokens without requiring an on-chain approval transaction.
- ``minOutputAmount``: The minimum acceptable output amount to protect the user against unfavorable price slippage.



## swapIn32502CA71

```
function swapIn32502CA71(address token, uint256 amountIn, uint256 minOut) external returns (int256, int256) {
        (bool success, bytes memory data) = _getExecutorSwap().delegatecall(abi.encodeCall(
            ISeawaterExecutorSwap.swap904369BE,
            (
                token,
                true,
                int256(amountIn),
                type(uint256).max
            )
        ));
        require(success, string(data));

        (int256 swapAmountIn, int256 swapAmountOut) = abi.decode(data, (int256, int256));
        require(-swapAmountOut >= int256(minOut), "min out not reached!");
        return (swapAmountIn, swapAmountOut);
    }
```

- The ``swapIn32502CA71`` function is used for performing an "exact-input" token swap. This means the user specifies the exact amount of tokens they want to swap (input token), and the contract will calculate the resulting output token amount based on real-time exchange rates on a decentralized exchange (DEX) or on the automated market maker (AMM). 

Parameters:

- ``inputAmount``: The exact amount of tokens the user wants to swap.
- ``inputToken``: The token type the user is swapping from.
- ``outputToken``: The token the user wants to receive.
- ``minOutputAmount``: The minimum amount of outputToken the user is willing to accept to protect against slippage.


## swapInPermit2CEAAB576

```
 function swapInPermit2CEAAB576(address token, uint256 amountIn, uint256 minOut, uint256 nonce, uint256 deadline, uint256 maxAmount, bytes memory sig) external returns (int256, int256) {
        (bool success, bytes memory data) = _getExecutorSwapPermit2().delegatecall(abi.encodeCall(
            ISeawaterExecutorSwapPermit2.swapPermit2EE84AD91,
            (
                token,
                true,
                int256(amountIn),
                type(uint256).max,
                nonce,
                deadline,
                maxAmount,
                sig
            )
        ));
        require(success, string(data));

        (int256 swapAmountIn, int256 swapAmountOut) = abi.decode(data, (int256, int256));
        // this contract uses checked arithmetic, this negate can revert
        require(-swapAmountOut >= int256(minOut), "min out not reached!");
        return (swapAmountIn, swapAmountOut);
    }
```


- The ``swapInPermit2CEAAB576`` function IS a token swap function that uses Permit2 functionality for off-chain approvals. This enables users to swap an exact amount of tokens without needing an additional on-chain approval transaction saving on gas costs. 

Parameters: 

- inputAmount: The exact amount of tokens the user wants to swap.
- inputToken: The address of the token the user is swapping from.
- outputToken: The address of the token the user wishes to receive.
- minOutputAmount: The minimum acceptable amount of outputToken that the user is willing to receive, providing slippage protection.
- permitData: The Permit2 authorization data (including a signed message) allowing the contract to transfer inputAmount of inputToken from the user’s wallet.





## swapOut5E08A399

```
function swapOut5E08A399(address token, uint256 amountIn, uint256 minOut) external returns (int256, int256) {
        (bool success, bytes memory data) = _getExecutorSwap().delegatecall(abi.encodeCall(
            ISeawaterExecutorSwap.swap904369BE,
            (
                token,
                false,
                int256(amountIn),
                type(uint256).max
            )
        ));
        require(success, string(data));

        (int256 swapAmountIn, int256 swapAmountOut) = abi.decode(data, (int256, int256));
        require(swapAmountOut >= int256(minOut), "min out not reached!");
        return (swapAmountIn, swapAmountOut);
    }
```

-  The ``swapOut5E08A399`` function performs a "swap-out" operation, where the user specifies the exact amount of tokens they want to receive (output amount), and the function calculates the required input amount of tokens to execute this swap.  


Parameters:

- ``outputAmount``: The exact amount of tokens the user wants to receive.
- ``inputToken``: The token being swapped from.
- ``outputToken``: The token the user wishes to receive.
- ``maxInputAmount``: The maximum acceptable amount of inputToken that the user is willing to pay, ensuring protection against high slippage.


## swapOutPermit23273373B

```
function swapOutPermit23273373B(address token, uint256 amountIn, uint256 minOut, uint256 nonce, uint256 deadline, uint256 maxAmount, bytes memory sig) external returns (int256, int256) {
        (bool success, bytes memory data) = _getExecutorSwapPermit2().delegatecall(abi.encodeCall(
            ISeawaterExecutorSwapPermit2.swapPermit2EE84AD91,
            (
                token,
                false,
                int256(amountIn),
                type(uint256).max,
                nonce,
                deadline,
                maxAmount,
                sig
            )
        ));
        require(success, string(data));

        (int256 swapAmountIn, int256 swapAmountOut) = abi.decode(data, (int256, int256));
        require(swapAmountOut >= int256(minOut), "min out not reached!");
        return (swapAmountIn, swapAmountOut);
    }
```


-  The ``swapOutPermit23273373B`` funtion performs an exact-output token swap with ``permit`` functionality, which enables token approval and transfer in a single transaction allowing the user to approve the swap without needing a separate approval transaction. 

Parameters: 

- ``outputAmount``: The exact amount of the token the user wants to receive.
- ``inputToken``: The token being swapped from.
- ``outputToken``: The token the user wishes to receive.
- ``maxInputAmount``: The maximum amount of inputToken the user is willing to pay.
- ``permitData``: Data necessary for the permit signature, including the signature itself (v, r, s) and possibly the deadline for the permit to be valid.



 // position functions





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



# Getter Functions

``_getExecutorSwap()``: Retrieves the address of the ``EXECUTOR_SWAP_SLOT``. This executor is responsible for handling swap operations.

``_getExecutorSwapPermit2()``: Retrieves the address of the ``EXECUTOR_SWAP_PERMIT2_SLOT``, involved in handling swap transactions with Permit2 (a type of permit or token allowance mechanism).

``_getExecutorQuote()``: Retrieves the address from ``EXECUTOR_QUOTE_SLOT``, used for fetching or calculating price quotes for token exchanges.

``_getExecutorPosition()``: Retrieves the address from ``EXECUTOR_POSITION_SLOT``, managing or querying the user's position (e.g., open trades or staked positions).

``_getExecutorUpdatePosition()``: Retrieves the address from ``EXECUTOR_UPDATE_POSITION_SLOT``, used to update or modify the user’s position.

``_getExecutorAdmin()``: Retrieves the address from ``EXECUTOR_ADMIN_SLOT``, representing the admin module or the contract's primary administrator.

``_getExecutorFallback()``: Retrieves the address from ``EXECUTOR_FALLBACK_SLOT``, a fallback handler for unspecified or emergency actions.




# Setter Functions


## _setProxyAdmin

```
 function _setProxyAdmin(address newAdmin) internal {
        StorageSlot.getAddressSlot(PROXY_ADMIN_SLOT).value = newAdmin;
    }
```


``_setProxyAdmin(address newAdmin)`` Sets the address stored in ``PROXY_ADMIN_SLOT`` to newAdmin. This function is used to update the main administrative address for the proxy, which typically has permissions to upgrade or configure other components.


## _setProxies

```
function _setProxies(
        ISeawaterExecutorSwap executorSwap,
        ISeawaterExecutorSwapPermit2 executorSwapPermit2,
        ISeawaterExecutorQuote executorQuote,
        ISeawaterExecutorPosition executorPosition,
        ISeawaterExecutorUpdatePosition executorUpdatePosition,
        ISeawaterExecutorAdmin executorAdmin,
        ISeawaterExecutorFallback executorFallback
    ) internal {
        StorageSlot.getAddressSlot(EXECUTOR_SWAP_SLOT).value = address(executorSwap);
        StorageSlot.getAddressSlot(EXECUTOR_SWAP_PERMIT2_SLOT).value = address(executorSwapPermit2);
        StorageSlot.getAddressSlot(EXECUTOR_QUOTE_SLOT).value = address(executorQuote);
        StorageSlot.getAddressSlot(EXECUTOR_POSITION_SLOT).value = address(executorPosition);
        StorageSlot.getAddressSlot(EXECUTOR_UPDATE_POSITION_SLOT).value = address(executorUpdatePosition);
        StorageSlot.getAddressSlot(EXECUTOR_ADMIN_SLOT).value = address(executorAdmin);
        StorageSlot.getAddressSlot(EXECUTOR_FALLBACK_SLOT).value = address(executorFallback);
    }
```

- The ``_setProxies(...)`` function Accepts several executor addresses as parameters and assigns them to their respective storage slots:

- ``executorSwap`` → ``EXECUTOR_SWAP_SLOT``
- ``executorSwapPermit2`` → ``EXECUTOR_SWAP_PERMIT2_SLOT``
- ``executorQuote`` → ``EXECUTOR_QUOTE_SLOT``
- ``executorPosition`` → ``EXECUTOR_POSITION_SLOT``
- ``executorUpdatePosition`` → ``EXECUTOR_UPDATE_POSITION_SLOT``
- ``executorAdmin`` → ``EXECUTOR_ADMIN_SLOT``
- ``executorFallback`` → ``EXECUTOR_FALLBACK_SLOT``



