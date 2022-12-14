## Header
This is the course header. This will be added on top of every page. Go to [DoDAO.io](https://www.dodao.io) to know more.

---

## Composable Stable Pools


## Background


The DeFi ecosystem is full of correlated assets. We see many different tokens based on ETH, for example: wstETH, wETH, aETH, cETH, 
or BTC eg: renBTC, sBTC, WBTC, aWBTC, cWBTC.

If we try to lend or stake our assets on many of the DeFi Protocols, we get a correlated asset in return.

There are multiple ways to manage risk when investing an asset. One approach is to spread your investments across multiple 
platforms. This can be complex, and you need to understand the workflow of each platform. Another, easier way is to swap 
your token for different ones that are based on the same core asset. This prevents complexity and extra gas fees, but 
still ensures diversification and returns on your investments.

Being able to swap large quantities of correlated assets quickly and with minimal price slippage is a big advantage for both 
institutional investors and retail users in DeFi. For liquidity providers, in this case, impermanent loss is not as big of a 
concern (except in rare cases where the depeg's happen).        

    


---
## Stable Math

## Stable Math
Pools that use a constant product invariant (x*y=k) might not be the best option as there can be a lot of slippage, which 
is quite undesirable in the scenario of correlated assets. Constant sum invariant (x+y=k) seems to be more relevant but 
can end the pool having just single type of tokens.

Stable Math's Invariant solves both of these problems and can keep prices more equal as long as the liquidity pool is 
not extremely unbalanced

<div align="center">
<img width="560" height="315" src="https://github.com/balancer-labs/docs-v2/raw/fc4f11145504bf9bc2dbed3ac30b6ffbe704d0aa/.gitbook/assets/output%20(1).gif" />
</div>
<br/>
<br/>
StableSwap approaches Constant Product as A->0 and Constant Sum as A->∞

<br/>
<br/>

$$

A \cdot n^n \cdot \sum{x_i} +D = A \cdot D \cdot n^n + { \frac{D^{n+1}}{{n}^{n}\cdot \prod{x_i} } }

$$


<br/>

 Where:
 
 * $n$ is the number of tokens 
 * $x_i$  is balance of token  $$i$$ 
 * $$A$$ is the amplification parameter 

More detail about stable math can be found [here](https://docs.balancer.fi/concepts/math/stable-math)

## Amplification Parameter
The A-Factor, also known as the amplification parameter, plays a major role in how flattened the curve will be. This 
parameter controls how much slippage occurs, with higher values resulting in less slippage.

If the A-Factor is set to zero, the curve will follow a similar path to x*y=k, but with increased slippage. The 
illustration below shows the stable swap invariant with the A-Factor set to zero. Notice the green line:

The higher the A-Factor is set, the more flattened the curve becomes.

We can pass `amplificationParameter` to the constructor when creating instance of ComposableStablePool.


    


---
## Evaluation





##### When Amplification Parameter approaches infinity (A->∞) what type of invariant does the pool uses?  

- [ ]  Constant Product
- [x]  Constant Sum
- [ ]  Constant Mean
- [ ]  Constant Mode





##### When Amplification Parameter approaches zero (A-> 0) what type of invariant does the pool uses?  

- [x]  Constant Product
- [ ]  Constant Sum
- [ ]  Constant Mean
- [ ]  Constant Mode

    


---
## Composable Stable Pool

# ComposableStablePool
`ComposableStablePool` This is a 5-token stable pool that also contains its own BPT. It also is a type of Balancer Pool that extends from `BasePool`, that uses the authenticate modifier to protect its functions with advanced authorization mechanisms.

Some of the benefits that every pool gets when it inherits from BasePool are:
1. Emergency Pause - The BasePool inherits from the `TemporarilyPausable` contract, which provides an emergency pause feature within the first 30 days of factory deployment.
2. Swap Fee management - A simple but important feature is management of the swap fee percentage.
3. Vault Integration - The Vault is the central point for all pool tokens and related bookkeeping. It is the primary point of interaction for joining, exiting, or swapping pools, and delegates to the appropriate pools as needed.

## Pool Initialization
`ComposableStablePool` uses the initialization step of Balancer Pools to mint Balancer Pool Tokens(BPT) to the first account that joins them. By minting the entire BPT supply for the initial joiner and then pulling all tokens except those due to the joiner, ComposableStablePool arrives at the desired state of the Pool holding all BPT except the joiner's.

## Balancer Pool Token (BPT)
BPT is preminted and registered as one of the tokens in the pool during initialization. This allows for token swaps to be made that look like either a single-token join or exit. ComposableStablePool also support regular joins and exits, which can create or destroy BPT.

The preminted BPT is deposited into the Vault as the initial balance of the Pool. Until it is transferred out of the Pool, it does not belong to any entity. The Pool's arithmetic acts as if BPT doesn't exist. Therefore, the total supply of BPT is not a useful value. ComposableStablePool relies on the 'virtual supply' (how much BPT is actually owned outside the Vault) instead.

    


---
## Evaluation





##### What is the max number of tokens that we use in ComposableStablePool?  

- [ ]  2
- [ ]  3
- [ ]  4
- [x]  5





##### When using ComposableStablePool, the BPT that is minted is part of which pool  

- [x]  Same ComposableStablePool
- [ ]  New and separate ComposableStablePool
- [ ]  Liquidity Bootstrapping Pool
- [ ]  Boosted Pool

    


---
## Rate Provides

ComposableStablePool's most elegant feature is its ability to provide a custom Rate Provider for each token.

Constructor of ComposableStablePool takes the following struct as params

```solidity
    struct NewPoolParams {
        IVault vault;
        IProtocolFeePercentagesProvider protocolFeeProvider;
        string name;
        string symbol;
        IERC20[] tokens;
        IRateProvider[] rateProviders;
        uint256[] tokenRateCacheDurations;
        bool[] exemptFromYieldProtocolFeeFlags;
        uint256 amplificationParameter;
        uint256 swapFeePercentage;
        uint256 pauseWindowDuration;
        uint256 bufferPeriodDuration;
        address owner;
    }
```
We can pass a custom rate provider to the constructor for each token. The rate conversion logic is also handled very well when scaling the tokens. Here we can see the `ComposableStablePoolRates.sol` to see how rate provider is actually used.

```solidity
    /**
     * @dev Overrides scaling factor getter to compute the tokens' rates.
     */
    function _scalingFactors() internal view virtual override returns (uint256[] memory) {
        // There is no need to check the arrays length since both are based on `_getTotalTokens`
        uint256 totalTokens = _getTotalTokens();
        uint256[] memory scalingFactors = new uint256[](totalTokens);

        for (uint256 i = 0; i < totalTokens; ++i) {
            scalingFactors[i] = _getScalingFactor(i).mulDown(_getTokenRate(i));
        }

        return scalingFactors;
    }
```

The logic to ensure that tokens are properly scaled when calculating the swapped tokens is already part of BasePool. ComposableStablePool(`ComposableStablePoolRates.sol`) extends it and considers the rate provider also.

    


---
## Evaluation





##### What is the role of Rate Providers?  

- [ ]  The determine by how much decimals do we need to scale the tokens
- [x]  The determine and apply the token rate when swapping
- [ ]  To normalize all the token prices to same value
- [ ]  To enhance the security of the pool





##### How many rate providers can we pass to ComposableStablePool?  

- [ ]  Two rate providers are allowed per pool
- [ ]  There is no limit
- [ ]  Only one rate provider is allowed per pool
- [x]  Same as the number of tokens used in the pool

    


---
## Extend  Composable Stable Pools

You can easily extend Balancer's Composable Stable Pool by installing the package containing `ComposableStablePool.sol`.

```shell
npm i @balancer-labs/v2-pool-stable
``` 

Here are the first few lines of `ComposableStablePool.sol` and it explains how the code is structured.
```solidity
contract ComposableStablePool is
    IRateProvider,
    BaseGeneralPool,
    StablePoolAmplification,
    ComposableStablePoolRates,
    ComposableStablePoolProtocolFees
{
    using FixedPoint for uint256;
    using PriceRateCache for bytes32;
    using StablePoolUserData for bytes;
    using BasePoolUserData for bytes;

    // The maximum imposed by the Vault, which stores balances in a packed format, is 2**(112) - 1.
    // We are preminting half of that value (rounded up).
    uint256 private constant _PREMINTED_TOKEN_BALANCE = 2**(111);

    // The constructor arguments are received in a struct to work around stack-too-deep issues
    struct NewPoolParams {
        IVault vault;
        IProtocolFeePercentagesProvider protocolFeeProvider;
        string name;
        string symbol;
        IERC20[] tokens;
        IRateProvider[] rateProviders;
        uint256[] tokenRateCacheDurations;
        bool[] exemptFromYieldProtocolFeeFlags;
        uint256 amplificationParameter;
        uint256 swapFeePercentage;
        uint256 pauseWindowDuration;
        uint256 bufferPeriodDuration;
        address owner;
    }

    constructor(NewPoolParams memory params)
        BasePool(
            params.vault,
            IVault.PoolSpecialization.GENERAL,
            params.name,
            params.symbol,
            _insertSorted(params.tokens, IERC20(this)),
            new address[](params.tokens.length + 1),
            params.swapFeePercentage,
            params.pauseWindowDuration,
            params.bufferPeriodDuration,
            params.owner
        )
        StablePoolAmplification(params.amplificationParameter)
        ComposableStablePoolStorage(_extractStorageParams(params))
        ComposableStablePoolRates(_extractRatesParams(params))
        ProtocolFeeCache(
            params.protocolFeeProvider,
            ProviderFeeIDs({ swap: ProtocolFeeType.SWAP, yield: ProtocolFeeType.YIELD, aum: ProtocolFeeType.AUM })
        )
    {
        // solhint-disable-previous-line no-empty-blocks
    }

``` 

As you can see, the functionality of stable pools is divided into smaller contracts, each performing a single responsibility. This division of code helps to ensure that the code is easy to understand and extend. ComposableStablePool extends from 
1. `BaseGeneralPool` - provides some generic functionality related to security, swapping, vault integration etc. 
2. `StablePoolAmplification` - responsible for updating of Amplification Parameter with time
3. `ComposableStablePoolRates` - builds a cache of the rate provides and then using these rates to calculate the number of output or input tokens for swap.
4. `ComposableStablePoolProtocolFees` - calculates the protocol fees during joins and exits

    


---
## Create and Deploy

There are a few ways to deploy pools. Balancer provides two very easy ways to deploy new pools.
### 1. Using Factory (Typescript/Javascript)
Like other pools, ComposableStablePool has a factory from which users can deploy new pools. To deploy a pool, you must call that factory's create() function with the following arguments 
```javascript
  create(
    name: PromiseOrValue<string>,
    symbol: PromiseOrValue<string>,
    tokens: PromiseOrValue<string>[],
    amplificationParameter: PromiseOrValue<BigNumberish>,
    rateProviders: PromiseOrValue<string>[],
    tokenRateCacheDurations: PromiseOrValue<BigNumberish>[],
    exemptFromYieldProtocolFeeFlags: PromiseOrValue<boolean>[],
    swapFeePercentage: PromiseOrValue<BigNumberish>,
    owner: PromiseOrValue<string>,
    overrides?: Overrides & { from?: PromiseOrValue<string> }
  ): Promise<ContractTransaction>;

```
Using hardhat's ethers library you can easily create the pool by passing all the required parameters to the ComposableStablePoolFactory's create function   
```javascript
const ComposableStablePoolFactoryAddress = '0x.......';
const factory = await ethers.getContractAt(
          'ComposableStablePoolFactory',
          ComposableStablePoolFactoryAddress
      );

// function for your corresponding pool in that pool factory's ABI
const tx = await factory.create(
      NAME, SYMBOL, tokens, 
      amplificationParameter, rateProviders, tokenRateCacheDurations, exemptFromYieldProtocolFeeFlags,
      swapFeePercentage, owner
);
const receipt = await tx.wait();

// We need to get the new pool address out of the PoolCreated event
const events = receipt.events.filter((e) => e.event === 'PoolCreated');
const poolAddress = events[0].args.pool;

// We're going to need the PoolId later, so ask the contract for it
const pool = await ethers.getContractAt('ComposableStablePool', poolAddress);
const poolId = await pool.getPoolId();
```
## 2. Using [BalPy](https://pypi.org/project/balpy/) 
Steps 

1. Make a virtual environment
   ```shell
    python3 -m venv ./venv
    source ./venv/bin/activate
   ```
2. Install balpy
  ```shell
    python3 -m pip install balpy
  ```
3. Source your environment (balpy will warn you if you forget this step) 4. Copy a pool config for the poolType you want to create. You can find sample file for ComposableStablePool [here](https://github.com/balancer-labs/balpy/blob/main/samples/poolCreation/sampleComposableStablePool.json)
  ```shell
    cp sampleComposableStablePool.json mySampleComposableStablePool.json
  ```
5. Edit your new pool file in your favorite text editor and run the Python script
  ```shell
    python3 poolCreationSample.py mySampleComposableStablePool.json
  ```


    


---
## Evaluation





##### Select two convenient ways which Balancer provides for deploying Balancer Pools  

- [x]  Using PoolFactory(Typescript/Javascript)
- [ ]  Using balancer's clojure library
- [x]  Using BalPy
- [ ]  Using Kubernetes

    


---
## Permissions

Access Control Management in ComposableStablePool works in the exact same way as other pools. `ComposableStablePool` extends from `BasePool` which further extends from `BasePoolAuthorization`. 

`BasePool` delegates Access Control Management to Vault's Authorizer. This can be seen from the below code

```solidity
  function _getAuthorizer() internal view override returns (IAuthorizer) {
      // Access control management is delegated to the Vault's Authorizer. This lets Balancer Governance manage which
      // accounts can call permissioned functions: for example, to perform emergency pauses.
      // If the owner is delegated, then *all* permissioned functions, including `setSwapFeePercentage`, will be under
      // Governance control.
      return getVault().getAuthorizer();
  }
```

Most of the methods that modify ComposableStablePool's behaviour are protected using the same authentication mechanism as used in other pool. Below are some of the signatures of ComposableStablePool related functions that modify the behaviour and are protected using the `authenticate` modifier.

```solidity

  // ComposableStablePoolRates.sol

  function setTokenRateCacheDuration(IERC20 token, uint256 duration)
  external authenticate


  // StablePoolAmplification.sol

  function startAmplificationParameterUpdate(uint256 rawEndValue, uint256
  endTime) external authenticate


  function stopAmplificationParameterUpdate() external authenticate

```

`authenticate` modifier is declared in `Authentication.sol`

```solidity

  /**
   * @dev Reverts unless the caller is allowed to call this function. Should only be applied to external functions.
   */
  modifier authenticate() {
      _authenticateCaller();
      _;
  }

```


    


---
## Footer
This is the git footer. This will be added on top of every page. Go to [DoDAO.io](https://www.dodao.io) to know more.
    
